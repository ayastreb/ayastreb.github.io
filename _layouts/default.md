<!DOCTYPE html>
<html>
<head>
  <title>{% if page.title %}{{ page.title }} – {% endif %}{{ site.name }}</title>

  {% include meta.html %}

  <!--[if lt IE 9]>
  <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
  <![endif]-->

  <link
      href="https://fonts.googleapis.com/css?family=PT+Sans|PT+Serif|PT+Mono|PT Serif Caption|Roboto"
      rel="stylesheet">
  <link rel="stylesheet" type="text/css" href="{{ site.baseurl }}/style.css"/>
  <link rel="alternate" type="application/rss+xml"
        title="{{ site.name }} - {{ site.description }}"
        href="{{ site.baseurl }}/feed.xml"/>
  <link rel="icon" type="image/x-icon" href="/favicon.png" />

  <!-- Created with Jekyll Now - http://github.com/barryclark/jekyll-now -->
</head>

<body>
<div class="wrapper-masthead">
  <div class="container">
    <header class="masthead clearfix">
      <div class="site-info">
        <h1 class="site-name">
          <a href="{{ site.baseurl }}/">{{ site.name }}</a>
        </h1>
      </div>

      <nav>
        <ul>
          {% assign url = page.url|remove:'index.html' %}
          {% for link in site.navigation %}
          {% if page.path contains '_posts' and link.url == '/' %}
            {% assign item_class = 'active' %}
          {% else %}
            {% if link.url == url %}
              {% assign item_class = 'active' %}
            {% else %}
              {% assign item_class = 'inactive' %}
            {% endif %}
          {% endif %}
          <li class="{{ item_class }}">
            <a href="{{ link.url }}" {% if link.target %}target="{{ link.target }}" {% endif %}>
              {{ link.text }}
            </a>
          </li>
          {% endfor %}
          <span></span>
        </ul>
      </nav>
    </header>
  </div>
</div>

<div id="main" role="main" class="container content">
  {{ content }}
</div>

<div class="wrapper-footer">
  <div class="container">
    <footer class="footer">
      Made with ❤️ by <a href="https://twitter.com/mrHabicht">Anatoliy Yastreb</a>
    </footer>
  </div>
</div>

{% include analytics.html %}
</body>
</html>
