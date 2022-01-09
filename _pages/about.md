---
layout: page
title: About
permalink: /about/
---

<style type="text/css">
    .image {
        border-radius: 50%;
        overflow: hidden;
        background: transparent url('/assets/about/5b94db56.jpg') no-repeat 50% 50%;
        width: 400px;
        height: 400px;
        background-size: cover;
        margin-bottom: 32px
    }
    .profile {
        line-height: 36px;
        display: flex;
        align-items: center;
        flex-direction: column
    }
    .info {
        margin-top: 32px
    }
    .profile + ul {
        display: flex;
        list-style-type: none;
        justify-content: center;
        margin: 16px 0 0 0 !important;
        padding: 0 !important;
    }
    .profile + ul a {
        background-image: none !important
    }
    .profile + ul .icon {
        width: 70px;
        height: 70px;
        margin: 0 8px;
        fill: #999 !important
    }
    .profile + ul .icon:hover {
        fill: #777 !important
    }
</style>

<div class="profile">
    <div class="image"></div>
    <div>My name is Marek Hudyma.</div>
    <div>I am Backend Developer who love coding and software architecture.</div>
    <div class="info">You can see my professional profile: </div>
</div>

{% if site.twitter_username %}
+ <a href="http://www.twitter.com/{{ site.twitter_username }}" target="_blank"><span data-icon="ei-sc-twitter" data-size="s"></span></a>
{% endif %}

{% if site.github_username %}
+ <a href="http://www.github.com/{{ site.github_username }}" target="_blank"><span data-icon="ei-sc-github" data-size="s"></span></a>
{% endif %}

{% if site.linkedin_username %}
+ <a href="https://www.linkedin.com/in/{{ site.linkedin_username }}" target="_blank"><span data-icon="ei-sc-linkedin" data-size="s"></span></a>
{% endif %}

{% if site.email %}
+ <a href="mailto:{{ site.email }}" target="_blank"><span data-icon="ei-envelope" data-size="s"></span></a>
{% endif %}
