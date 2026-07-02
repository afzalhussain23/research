---
layout: page
permalink: /repositories/
title: repositories
description: Selected GitHub repositories by Afzal Hussain.
nav: true
nav_order: 4
---

{% if site.data.repositories.github_users %}

{% assign github_user = site.data.repositories.github_users | first %}
{% endif %}

{% if site.data.repositories.github_repos %}

## GitHub Repositories

<style>
  .repositories .repository-profile {
    margin-bottom: 1rem;
  }

  .repositories .repository-card,
  .repositories .repository-card:hover {
    color: inherit;
    text-decoration: none;
  }

  .repositories .repository-card .card-title {
    font-size: 1.05rem;
    line-height: 1.35;
    overflow-wrap: anywhere;
    text-transform: none;
  }

  .repositories .repository-path {
    color: var(--global-text-color-light);
    overflow-wrap: anywhere;
  }

  .repositories .repository-action {
    color: var(--global-theme-color);
    font-size: 0.9rem;
    font-weight: 500;
  }
</style>

<div class="repositories">
  <p class="repository-profile">
    <a href="https://github.com/{{ github_user }}" rel="external nofollow noopener" target="_blank">
      <i class="fa-brands fa-github"></i>
      github.com/{{ github_user }}
    </a>
  </p>

  <div class="row row-cols-1 row-cols-md-2">
    {% for repo in site.data.repositories.github_repos %}
      {% assign repo_parts = repo | split: "/" %}
      {% assign repo_name = repo_parts[1] %}
      {% assign repo_title = repo_name | replace: "-", " " %}
      <div class="col mb-4">
        <a class="repository-card card hoverable h-100" href="https://github.com/{{ repo }}" rel="external nofollow noopener" target="_blank">
          <div class="card-body d-flex flex-column">
            <h3 class="card-title">{{ repo_title }}</h3>
            <p class="repository-path card-text mb-3">{{ repo }}</p>
            <p class="repository-action card-text mt-auto mb-0">
              <i class="fa-brands fa-github"></i>
              View repository
            </p>
          </div>
        </a>
      </div>
    {% endfor %}
  </div>
</div>
{% endif %}
