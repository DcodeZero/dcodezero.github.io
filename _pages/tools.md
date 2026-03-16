---
title: "Tools & Scripts"
layout: archive
permalink: /tools/
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.7"
  overlay_image: /assets/images/tools-scripts.svg
---

Custom scripts, automation tools, exploit code, and red team utilities. From reconnaissance to post-exploitation - tools I've built and use in my engagements.

{% assign posts = site.posts | where_exp: "post", "post.categories contains 'Tools'" %}

{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
