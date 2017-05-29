---
layout: post
title: Deploy Jekyll to GitHub Pages with Travis CI
---

If you want to use [GitHub Pages](https://pages.github.com/) with custom Jekyll plugins 
(e.g. anything other than [officially supported plugins](https://help.github.com/articles/adding-jekyll-plugins-to-a-github-pages-site/))
&mdash; you need to generate the site content and push it to the repository manually. 
If you want to automate this step and deploy your site every time you push changes to Jekyll, [Travis CI](https://travis-ci.org/) might help you.
<!--more-->

Travis CI is a continuous integration tool which allows you to setup build process for your GitHub repo. It is free for public repos.
Every time you push changes to the repository, Travis CI will setup a virtual machine and run the build script you provide it.

### Travis Configuration
Since Jekyll runs on Ruby, we need to tell Travis to setup a VM with Ruby environment.
We need to create `.travis.yml` file in the root of the repository for that.

{% highlight yaml %}
language: ruby
# only run CI-builds on master branch
branches:
  only:
  - master
rvm:
- 2.2.3
# set execution permission on our build script
before_script:
 - chmod +x ./script/cibuild
# path to our build script. 
# Travis will run `bundle install` by default before running our script
script: ./script/cibuild
exclude: [vendor]
sudo: false
{% endhighlight %}

### Build Script  
To deploy our Jekyll site we need to push generated static content to `gh-pages` branch of our repository.
To achieve this we need to do following:
* clone our repo at `gh-pages` branch with write permissions to `_site` directory
* build Jekyll site to `_site` directory
* commit and push all changes in `_site` directory to `gh-pages` branch

We need to create [personal access token](https://github.com/settings/tokens/new) 
and set it in Travis build "Environment Variables" settings to be able to clone the repo with write permissions.  

Then we need to add following Bash script to `script/cibuild` in our repository:
{% highlight bash %}
#!/bin/bash

# skip if build is triggered by pull request
if [ $TRAVIS_PULL_REQUEST == "true" ]; then
  echo "this is PR, exiting"
  exit 0
fi

# enable error reporting to the console
set -e

# cleanup "_site"
rm -rf _site
mkdir _site

# clone remote repo to "_site"
git clone https://${GH_TOKEN}@github.com/ayastreb/11route.git --branch gh-pages _site

# build with Jekyll into "_site"
bundle exec jekyll build

# push
cd _site
git config user.email "anatoliy.yastreb@gmail.com"
git config user.name "Anatoliy Yastreb"
git add --all
git commit -a -m "Travis #$TRAVIS_BUILD_NUMBER"
git push --force origin gh-pages

{% endhighlight %}

After this is done, Travis will run the build script after every push to `master` branch and deploy 
the generated static content to `gh-pages` branch automatically! 