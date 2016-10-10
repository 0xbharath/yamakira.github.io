---
layout:    page
permalink: "/training/"
author:    bharath
weight:    5
menutitle: Training
title:     
excerpt:   Content of all the workshops I have conducted
---

Below is the list of workshops I have conducted in the past.

All material is licensed with an open license like CreativeCommons, allowing anyone to use the material however they see fit, so long as they share modified works back to the community.

We highly encourage people to use this material to learn, modify and also to teach it elsewhere.

For more info on the philosophy of open course material: [http://www.opensecuritytraining.info/Why.html](http://www.opensecuritytraining.info/Why.html)

<div id="content" class="content">
    <center><h3>Trainings</h3></center>
    <ul class="category recent-posts">       
        {% for post in site.categories.trainings %}
        {% unless post.draft %}

        {% if post.menutitle %}
        {% assign title = post.menutitle %}
        {% else %}
        {% assign title = post.title %}
        {% endif %}

        <li>
            <div class="article">
                <article class="article" itemscope itemtype="http://schema.org/BlogPosting">
                    <header class="post-header">
                        <span class="title"><a itemprop="name" href="{{ post.url | prepend: site.github.url | prepend: site.baseurl}}" title="{{ title }}">{{ title }}</a></span>
                        <time class="date" itemprop="datePublished" datetime="{{post.date | date: "%Y-%m-%d"}}">{{post.date | date: "%B %e, %Y"}}</time>
                    </header>
                </article>
            </div>
        </li>
        {% endunless %}
        {% endfor %}
    </ul>

