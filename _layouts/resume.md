---
layout: default
---

<div class="wrapper" itemscope itemtype="http://schema.org/Person">
  <header class="page-header">

    {% if site.resume_avatar == 'true' %}
    <img src="images/avatar.jpg" alt="my photo" class="avatar" itemprop="image"> {% endif %}

    <h1 class="header-name" itemprop="name">{{ site.resume_name }}</h1>

    <div class="title-bar">

      <h2 class="header-title" itemprop="jobTitle">{{ site.resume_title }}</h2>

      {% include icon-links.html %}
    </div>

    <div class="executive-summary" itemprop="description">
      <p>
        I'm a web software engineer with vast experience in eCommerce and web development. I've led teams of 3 to 4 developers and
        have experience setting up team workflow according to industry's best practices.
      </p>
      <p>
        I'm passionate about complete process of product development - from requirements collection to deployment and impact analysis.
        I enjoy working with smart people, crafting software which brings value to customers, and continuously improving
        my skills.
      </p>
      <p>
        In my free time I contribute to open source projects and learn new technologies.
      </p>
    </div>

    {% if site.resume_looking_for_work == 'yes' %}
    <a href="mailto:{{ site.resume_contact_email }}" class="contact-button" itemprop="email">Contact me</a> {% elsif site.resume_looking_for_work
    == 'no' %}
    <a class="contact-button not-looking">I'm not looking for work right now.</a> {% else %} {% endif %}

  </header>


  {% if site.resume_section_skills %}
  <!-- begin Skills -->
  <section class="content-section">

    <header class="section-header">
      <h2>Skills</h2>
    </header>

    <div class="flex-row">
      <div class="one-half-desktop">
        <div class="resume-item">
          <h4 class="resume-item-details">Languages</h4>
          <p class="resume-item-copy">
            JavaScript, Ruby, PHP
          </p>
        </div>

        <div class="resume-item">
          <h4 class="resume-item-details">Frameworks</h4>
          <p class="resume-item-copy">
            React, AngularJS, Laravel, Sinatra
          </p>
        </div>

        <div class="resume-item">
          <h4 class="resume-item-details">WebServices</h4>
          <p class="resume-item-copy">
            SOAP, REST, JSON API, oAuth
          </p>
        </div>
      </div>

      <div class="one-half-desktop">
        <div class="resume-item">
          <h4 class="resume-item-details">Databases & Search</h4>
          <p class="resume-item-copy">
            MySQL, CouchDB, Redis, ElasticSearch
          </p>
        </div>

        <div class="resume-item">
          <h4 class="resume-item-details">Testing</h4>
          <p class="resume-item-copy">
            Jest, Jasmine, PHPUnit, Selenium
          </p>
        </div>

        <div class="resume-item">
          <h4 class="resume-item-details">Continuous Integration</h4>
          <p class="resume-item-copy">
            Bamboo, Capistrano, TravisCI
          </p>
        </div>
      </div>
    </div>

  </section>
  <!-- end Skills -->
  {% endif %} {% if site.resume_section_experience %}
  <!-- begin Experience -->
  <section class="content-section">

    <header class="section-header">
      <h2>Experience</h2>
    </header>

    <div class="resume-item" itemscope itemprop="worksFor" itemtype="http://schema.org/Organization">
      <h3 class="resume-item-title" itemprop="name">Luxury Fashion Trade GmbH (<a href="http://fashionette.de" target="_blank">fashionette.de</a>)
      </h3>
      <h4 class="resume-item-details" itemprop="description">Senior Software Developer &bull; 2013 &mdash; 2017</h4>
      <p class="resume-item-copy">
        <strong>Technologies:</strong> Magento, PHP, Laravel, Solr, Varnish, Microservices
      </p>
      <p class="resume-item-copy">
        Led development team of 3-4 developers in various projects, including new features implementation, payment methods integration
        and online shop localization in 16 new countries for Germany's biggest online designer handbags retailer.
      </p>

    </div>

    <div class="resume-item" itemscope itemprop="worksFor" itemtype="http://schema.org/Organization">
      <h3 class="resume-item-title" itemprop="name">Magento, X.commerce, Inc. (an eBay company)</h3>
      <h4 class="resume-item-details" itemprop="description">Senior Software Engineer &bull; 2011 &mdash; 2013</h4>
      <p class="resume-item-copy">
        <strong>Technologies:</strong> PHP, SOAP WSI, REST, Redis, MongoDB, Gearman
      </p>
      <p class="resume-item-copy">
        Worked on Magento Core API, performed R&D on Magento 2 API platform.<br/> Participated in Magento Connect and Magento
        GO (SaaS) projects development.
      </p>

    </div>

    <div class="resume-item" itemscope itemprop="worksFor" itemtype="http://schema.org/Organization">
      <h3 class="resume-item-title" itemprop="name">Independent Contractor</h3>
      <h4 class="resume-item-details" itemprop="description">Web Developer &bull; 2007 &mdash; 2010</h4>
      <p class="resume-item-copy">
        <strong>Technologies:</strong> AWS, PHP, HTML, CSS, JavaScript
      </p>
      <p class="resume-item-copy">
        Implemented various web projects for international customers.
      </p>

    </div>

    <div class="resume-item" itemscope itemprop="worksFor" itemtype="http://schema.org/Organization">
      <h3 class="resume-item-title" itemprop="name">eTek Systems</h3>
      <h4 class="resume-item-details" itemprop="description">Junior PHP Developer &bull; 2006 &mdash; 2007</h4>
      <p class="resume-item-copy">
        <strong>Technologies:</strong> PHP, XML, Atom
      </p>
      <p class="resume-item-copy">
        Worked on international customer's web projects and in-house tools.
      </p>

    </div>

  </section>
  <!-- end Experience -->
  {% endif %} {% if site.resume_section_education %}
  <!-- begin Education -->
  <section class="content-section">

    <header class="section-header">
      <h2>Education</h2>
    </header>

    <div class="resume-item" itemscope itemprop="alumniOf" itemtype="http://schema.org/CollegeOrUniversity">
      <h3 class="resume-item-title" itemprop="name">Coursera</h3>
      <h4 class="resume-item-details" itemprop="description">Data Structures and Algorithms, 2016</h4>
    </div>

    <div class="resume-item" itemscope itemprop="alumniOf" itemtype="http://schema.org/CollegeOrUniversity">
      <h3 class="resume-item-title" itemprop="name">Udacity</h3>
      <h4 class="resume-item-details" itemprop="description">Tech Entrepreneur Nanodegree &bull; 2015 &mdash; 2016</h4>
    </div>

    <div class="resume-item" itemscope itemprop="alumniOf" itemtype="http://schema.org/CollegeOrUniversity">
      <h3 class="resume-item-title" itemprop="name">National Mining University of Ukraine</h3>
      <h4 class="resume-item-details" itemprop="description">Bachelor, Management, Economics &bull; 2004 &mdash; 2008
      </h4>
    </div>

  </section>
  <!-- end Education -->
  {% endif %}

</div>
