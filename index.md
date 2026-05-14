---
layout: default
---

# Project Overview

A small collection of hobby projects I worked on, written as short blog articles.

## About

I am interested in practical software projects that solve real problems. Most of my projects are around Python, home automation, local AI, small server setups, and experiments with learning systems.

I like building things that I can actually use, then writing down what worked, what did not work and what I learned from it.

## Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
