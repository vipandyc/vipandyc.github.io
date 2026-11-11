---
permalink: /publications/
title: "Selected Publications"
excerpt: "Selected first-author and shared-lead publications"
author_profile: true
---

# Selected Publications

Selected first-author and shared-lead work. <sup>*</sup>Equal contribution. For the full list, see my [Google Scholar profile](https://scholar.google.com/citations?user=RQP17VsAAAAJ&hl=en).

{% for paper in site.data.selected_publications %}
<div class="paper-box">
  <div class="paper-box-image">
    <div class="paper-visual">
      {% if paper.image %}
      <img src="{{ paper.image | relative_url }}" alt="{{ paper.image_alt | escape }}" loading="lazy">
      {% else %}
      <div class="paper-placeholder" aria-hidden="true"><span>{{ paper.topic }}</span></div>
      {% endif %}
      <div class="badge">{{ paper.badge }}</div>
    </div>
  </div>
  <div class="paper-box-text">
    <h2 class="paper-title"><a href="{{ paper.url }}">{{ paper.title }}</a></h2>
    <p class="paper-authors">{{ paper.authors }}</p>
    <p class="paper-venue">{{ paper.venue }}</p>
    <p class="paper-summary">{{ paper.summary }}</p>
  </div>
</div>
{% endfor %}
