---
layout: page
title: Open Source
---

<p>Throughout the years I've contributed to Open Source. Sometimes to fix a an itch with existing projects, sometimes the itch called for a whole new project. The latter of which are listed here:</p>

## Featured Repositories

{% assign repos = site.github.public_repositories | sort: 'stargazers_count' | where:"fork","false" %}
{% for repository in repos reversed %}
{% if site.featured contains repository.name %}
  * [{{ repository.name }}]({{ repository.html_url }}) - 
  <small>{{ repository.language }}
  <i class="fa fa-star-o" aria-hidden="true"></i> {{ repository.stargazers_count }} 
  <i class="fa fa-code-fork" aria-hidden="true"></i> {{ repository.forks_count }}   
   {{ repository.description }}</small>
{% endif %}
{% endfor %}

## Other Source Repositories

{% assign repos = site.github.public_repositories | sort: 'stargazers_count' | where:"fork","false" %}
{% for repository in repos reversed %}
  {% unless site.featured contains repository.name %}
  * [{{ repository.name }}]({{ repository.html_url }}) - 
  <small>{{ repository.language }}
  <i class="fa fa-star-o" aria-hidden="true"></i> {{ repository.stargazers_count }} 
  <i class="fa fa-code-fork" aria-hidden="true"></i> {{ repository.forks_count }}   
   {{ repository.description }}</small>
  {% endunless %}
{% endfor %}
