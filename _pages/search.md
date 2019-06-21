---
layout: search
title: "Search Posts"
permalink: /search/
author_profile: false
comments: false
exclude_from_search: true
---


<div class="search-content__inner-wrap">
{%- assign search_provider = site.search_provider | default: "lunr" -%}
{%- case search_provider -%}
  {%- when "lunr" -%}
          <input type="text" id="search" class="search-input" tabindex="-1" placeholder="{{ site.data.ui-text[site.locale].search_placeholder_text | default: 'Enter your search term...' }}" />
    <div id="results" class="results"></div>
  {%- when "google" -%}
    <form onsubmit="return executeQuery();" id="cse-search-box-form-id">
    <input type="text" id="cse-search-input-box-id" class="search-input" tabindex="-1" placeholder="{{ site.data.ui-text[site.locale].search_placeholder_text | default: 'Enter your search term...' }}" />
    </form>
    <div id="results" class="results">
        <gcse:searchresults-only></gcse:searchresults-only>
    </div>
  {%- when "algolia" -%}
    <div class="search-searchbar"></div>
    <div class="search-hits"></div>
{%- endcase -%}
</div>