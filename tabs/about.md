---
title: About

# The About page
# v2.0
# https://github.com/cotes2020/jekyll-theme-chirpy
# © 2017-2019 Cotes Chung
# MIT License
---

> **Note**: Add Markdown syntax content to file `tabs/about.md` and it will show up on this page.


<div id="socialMedia">
{% if site.social-media.email %}
    <a href="mailto:{{ site.social-media.email }}" title="Email"><i class="fa fa-envelope-square"></i></a>
{% endif %}
{% if site.social-media.facebook %}
    <a href="https://www.facebook.com/{{ site.social-media.facebook }}" title="Facebook"><i class="fa fa-facebook-square"></i></a>
{% endif %}
{% if site.social-media.twitter %}
    <a href="https://www.twitter.com/{{ site.social-media.twitter }}" title="Twitter"><i class="fa fa-twitter-square"></i></a>
{% endif %}
.
.
.
</div>
