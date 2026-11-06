---
layout: page
permalink: /repositories/
title: Repositories
description: Things I build, contribute to, and tinker with.
nav: true
nav_order: 4
---

{%- comment -%}
Tiles are driven by the repositories data file; its header comments are the
field reference. The markup lives in this page rather than in an include
because the includes directory is gem-owned (see AGENTS.md) and the style
contract fails the build if this repo owns it.
{%- endcomment -%}

{%- assign stats_url = site.external_services.github_readme_stats_url | default: "https://github-stats-extended.vercel.app" -%}
{%- assign max_lines = site.data.repositories.repo_description_lines_max | default: 2 -%}
{%- assign lang = site.lang | split: "-" | first -%}

{% if site.data.repositories.github_users %}

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {%- for user in site.data.repositories.github_users -%}
    {% include repository/repo_user.liquid username=user %}
  {%- endfor -%}
</div>
{% endif %}

{%- for section in site.data.repositories.repo_sections -%}

<h2 id="{{ section.title | slugify }}">{{ section.title }}</h2>
{%- if section.description %}<p>{{ section.description }}</p>{% endif -%}
<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {%- for entry in section.repos -%}
    {%- assign parts = entry.repo | split: "/" -%}
    {%- if site.data.repositories.github_users contains parts.first -%}
      {%- assign show_owner = false -%}
    {%- else -%}
      {%- assign show_owner = true -%}
    {%- endif -%}
    {%- assign pin_query = "username=" | append: parts[0] | append: "&repo=" | append: parts[1] | append: "&locale=" | append: lang | append: "&show_owner=" | append: show_owner | append: "&description_lines_count=" | append: max_lines -%}
    <div class="repo p-2 text-center">
      <div class="repo-pin">
        <a href="https://github.com/{{ entry.repo }}">
          <img class="only-light w-100" alt="{{ entry.repo }} on GitHub" src="{{ stats_url }}/api/pin/?{{ pin_query }}&theme={{ site.repo_theme_light }}" onerror="this.closest('.repo-pin').style.display='none'" />
          <img class="only-dark w-100" alt="{{ entry.repo }} on GitHub" src="{{ stats_url }}/api/pin/?{{ pin_query }}&theme={{ site.repo_theme_dark }}" onerror="this.closest('.repo-pin').style.display='none'" />
        </a>
      </div>
      <p class="mt-2 mb-2"><a href="https://github.com/{{ entry.repo }}"><i class="fa-brands fa-github"></i> {{ entry.repo }}</a></p>
      {%- if entry.note %}<p class="card-text">{{ entry.note }}</p>{% endif -%}
      {%- if entry.projects -%}
        <p class="mb-2">
          {%- for slug in entry.projects -%}
            {%- assign project_path = "_projects/" | append: slug | append: ".md" -%}
            {%- assign project = site.projects | where: "path", project_path | first -%}
            {%- if project -%}
              <a class="btn btn-sm btn-outline-primary" href="{{ project.url | relative_url }}" title="{{ project.title }}">{{ project.title | truncate: 44 }}</a>
            {%- else -%}
              <strong class="text-pink-700">No project page matches <code>{{ slug }}</code></strong>
            {%- endif -%}
          {%- endfor -%}
        </p>
      {%- endif -%}
    </div>
  {%- endfor -%}
</div>
{%- endfor -%}
