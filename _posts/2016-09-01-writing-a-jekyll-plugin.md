---
layout: post
title: Writing a Jekyll plugin
---

In this blog post I share my experience of building a Jekyll plugin.
<!--more-->
I have a [map page](https://11route.com/map) in my travel blog which shows all the places I've been to. This page goes through all blog posts and collects locations from YAML data. After that it builds a Google Map with a marker for each location.

I thought it might be a good idea to extract this functionality into a separate plugin and [open source](https://ayastreb.me/jekyll-maps/) it, so that other people can also use it on their pages. 

## Introduction

Jekyll plugins allow you to change behaviour without modifying Jekyll source.
There are several ways to use plugins: 

* you can just dump plugin code to `_plugins` directory 
* or you can pack your code as a Ruby gem and reference it in `_config.yml`
 
Jekyll has [good documentation](https://jekyllrb.com/docs/plugins/) for plugins system.
Plugins as a gem are more convenient to users and generally preferred. 

## Best Practices

Before diving into coding my new plugin I decided to research existing plugins implementations to find some common patterns and best practices. I've inspected plugins such as [jekyll-assets](https://github.com/jekyll/jekyll-assets), [jekyll-feed](https://github.com/jekyll/jekyll-feed), [jekyll-sitemap](https://github.com/jekyll/jekyll-sitemap), [jekyll-mentions](https://github.com/jekyll/jekyll-mentions) and others and found some good ideas.

### Directory Structure

Most of the plugins have following directory structure:
{% highlight bash %}
/lib/
/lib/${plugin_name}.rb         # Plugin module defined here
/lib/${plugin_name}/*.rb       # Actual plugin implementation
/lib/${plugin_name}/version.rb # Keep version number in constant here
/script/                       # Bootstrap, CI scripts and other bash scripts
/spec/                         # Tests for plugin code
HISTORY.md                     # Describe release changes here
README.md                      # User documentation for the plugin
${plugin_name}.gemspec         # Ruby Gem specification
{% endhighlight %}

### Versioning
Plugin version is kept in a separate file and re-used everywhere when you need to display a version. It makes releasing new versions easier.

### Gemspec
Gemspec describes your gem and its dependencies, I've came up with following for my plugin, based on several other plugins:

{% highlight ruby %}
# coding: utf-8
lib = File.expand_path("../lib", __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require "jekyll-maps/version"
Gem::Specification.new do |spec|
  spec.name          = "jekyll-maps"
  spec.summary       = "Jekyll Google Maps integration"
  spec.description   = "Google Maps support in Jekyll blog to easily embed maps with posts' locations"
  spec.version       = Jekyll::Maps::VERSION
  spec.authors       = ["Anatoliy Yastreb"]
  spec.email         = ["anatoliy.yastreb@gmail.com"]
  spec.homepage      = "https://github.com/ayastreb/jekyll-maps"
  spec.licenses      = ["MIT"]
  spec.files         = `git ls-files -z`.split("\x0").reject { |f| f.match(%r!^(test|spec|features)/!) }
  spec.require_paths = ["lib"]
  spec.add_dependency "jekyll", "~> 3.0"
  spec.add_development_dependency "rake", "~> 11.0"
  spec.add_development_dependency "rspec", "~> 3.5"
  spec.add_development_dependency "rubocop", "~> 0.41"
end
{% endhighlight %}

### Testing
To test integration of your plugin in Jekyll you can use a test helper.

{% highlight ruby %}
# spec_helper.rb
$LOAD_PATH.unshift File.expand_path("../../lib", __FILE__)
require "jekyll"
require "jekyll-maps"

Jekyll.logger.log_level = :error

RSpec.configure do |config|
  config.run_all_when_everything_filtered = true
  config.filter_run :focus
  config.order = "random"

  SOURCE_DIR = File.expand_path("../fixtures", __FILE__)
  DEST_DIR   = File.expand_path("../dest", __FILE__)

  def source_dir(*files)
    File.join(SOURCE_DIR, *files)
  end

  def dest_dir(*files)
    File.join(DEST_DIR, *files)
  end

  CONFIG_DEFAULTS = {
    "source"      => source_dir,
    "destination" => dest_dir,
    "gems"        => ["jekyll-maps"]
  }.freeze

  def make_page(options = {})
    page      = Jekyll::Page.new(site, CONFIG_DEFAULTS["source"], "", "page.md")
    page.data = options
    page
  end

  def make_site(options = {})
    site_config = Jekyll.configuration(CONFIG_DEFAULTS.merge(options))
    Jekyll::Site.new(site_config)
  end

  def make_context(registers = {}, environments = {})
    Liquid::Context.new(environments, {}, 
      { :site => site, :page => page }.merge(registers))
  end
end
{% endhighlight %}

And use it in the tests:

{% highlight ruby %}
# google_map_tag_spec.rb
require "spec_helper"

describe Jekyll::Maps::GoogleMapTag do
  let(:site) { make_site }
  before { site.process }

  context "full page rendering" do
    let(:content) { File.read(dest_dir("page.html")) }

    it "builds javascript" do
      expect(content).to match(%r!#{Jekyll::Maps::GoogleMapTag::JS_LIB_NAME}!)
    end

    it "includes external js only once" do
      expect(content.scan(%r!maps\.googleapis\.com!).length).to eq(1)
    end

    it "renders API key" do
      expect(content).to match(%r!maps/api/js\?key=GOOGLE_MAPS_API_KEY!)
    end
  end
end
{% endhighlight %}

## Plugin Categories

There are several different ways your plugin can alter Jekyll's behaviour:

* [Generators](https://jekyllrb.com/docs/plugins/#generators): create additional content or fill in template variables
* [Converters](https://jekyllrb.com/docs/plugins/#converters): convert custom markup language
* [Commands](https://jekyllrb.com/docs/plugins/#commands): implement custom subcommands for `jekyll` executable
* [Tags](https://jekyllrb.com/docs/plugins/#tags): create custom Liquid templates
* [Hooks](https://jekyllrb.com/docs/plugins/#hooks): subscribe to different events and modify content

In jekyll-maps I used tag to create `{% raw %}{% google_map %}{% endraw%}` tag and hooks to inject JavaScript code into the page.
Jekyll documentation provides good examples for each plugin category and you can also check my [implementation](https://github.com/ayastreb/jekyll-maps/tree/master/lib/jekyll-maps) on GitHub.

## Releasing

To release a new version of a plugin we need to do following:

* increase version number in `lib/${plugin_name}/version.rb`
* add release notes to `HISTORY.md`
* create a release and a tag in GitHub (optional)
* build new gem 
  `gem build ${plugin_name}.gemspec`
* push new version to rubygems.org 
  `gem push ${plugin_name}-${version}.gem`

When your plugin is ready to be used &mdash; don't forget to [add it](https://jekyllrb.com/docs/contributing/#adding-plugins) to the list of plugins in Jekyll docs, so that people can find it! 
