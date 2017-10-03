---
layout: default
---

<div class="wrapper">
  {% if site.resume_section_projects %}
  <!-- begin Projects -->
  <section class="content-section">
    <div class="resume-item">
      <h3 class="resume-item-title"><a href="https://github.com/ayastreb/money-tracker" target="_blank">Money Tracker</a></h3>
      <h4 class="resume-item-details">Maintainer &bull; 2017 &mdash; Present</h4>
      <p class="resume-item-copy">
        Offline-capable progressive web app for personal finance management.
        Implemented with React, Redux and PouchDB/CouchDB.<br/>
        Does not have a backend &mdash; communicates directly with CouchDB server.
      </p>
    </div>

    <div class="resume-item">
      <h3 class="resume-item-title"><a href="https://github.com/ayastreb/bandwidth-hero" target="_blank">Bandwidth Hero</a></h3>
      <h4 class="resume-item-details">Maintainer &bull; 2016 &mdash; Present</h4>
      <p class="resume-item-copy">
        Chrome extension which compresses images on the page via proxy server to save data.<br/>
        Proxy server is implemented with Node.js.
      </p>
    </div>

    <div class="resume-item">
      <h3 class="resume-item-title"><a href="https://ayastreb.me/jekyll-maps/" target="_blank">Jekyll Maps</a></h3>
      <h4 class="resume-item-details">Maintainer &bull; 2016 &mdash; Present</h4>
      <p class="resume-item-copy">
        A plugin that allows users to easily create different Google Maps on a <a href="http://jekyllrb.com" target="_blank">Jekyll</a> site.
      </p>
    </div>

    <div class="resume-item">
      <h3 class="resume-item-title"><a href="https://meal-planner.org" target="_blank">Meal Planner</a></h3>
      <h4 class="resume-item-details">Founder & Primary Developer &bull; 2015 &mdash; Present</h4>
      <p class="resume-item-copy">
        Recipe recommendation and nutrition balancing service.<br/>
        A health & fitness progressive web app built with AngularJS, Sinatra & ElasticSearch.
      </p>
    </div>

    <div class="resume-item">
      <h3 class="resume-item-title"><a href="https://jekyllrb.com/" target="_blank">Jekyll</a></h3>
      <h4 class="resume-item-details">Team Captain &bull; 2016 &mdash; Present</h4>
      <p class="resume-item-copy">
        As <a href="https://teams.jekyllrb.com/" target="_blank">@jekyll/build</a> team captain I help triaging incoming issues,
        review pull requests and occasionally contribute to bug fixes, refactoring and feature development.
      </p>
    </div>

  </section>
  <!-- end Projects -->
  {% endif %}
</div>
