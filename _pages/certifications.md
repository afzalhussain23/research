---
layout: page
permalink: /certifications/
title: certifications
description: courses and specializations completed online
nav: true
nav_order: 3.5
---

{% assign sorted = site.data.certifications | sort: "date" | reverse %}

<style>
  .certifications .cert-item {
    padding: 1.1rem 0;
    border-bottom: 1px solid var(--global-divider-color);
  }
  .certifications .cert-item:last-child { border-bottom: none; }
  .certifications .cert-title {
    font-weight: 600; font-size: 1.05rem; line-height: 1.3; margin-bottom: .25rem;
  }
  .certifications .cert-meta,
  .certifications .cert-tags {
    font-size: 0.875rem;
    color: var(--global-text-color-light);
  }
  .certifications .cert-tags { margin-top: .25rem; }
</style>

<div class="certifications">
  {% for c in sorted %}
    <div class="cert-item">
      <div class="cert-title">
        {% if c.url %}<a href="{{ c.url }}" target="_blank" rel="noopener">{{ c.name }}</a>{% else %}{{ c.name }}{% endif %}
      </div>
      <div class="cert-meta">
        {{ c.issuer }} &middot; {{ c.date | date: "%b %Y" }}{% if c.credential_id %} &middot; ID {{ c.credential_id }}{% endif %}
      </div>
      {% if c.tags %}
        <div class="cert-tags">{{ c.tags | join: " &middot; " }}</div>
      {% endif %}
    </div>
  {% endfor %}
</div>
