---
layout: default
title: Home
nav_order: 1
---

# RemotiveLabs Engineering Blog

Welcome to the RemotiveLabs engineering blog! Here we share technical insights, tutorials, and best practices for automotive software development using RemotiveLabs tools.

## Latest Posts

{% for post in site.posts %}
<article>
  <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
  {% if post.excerpt %}
    {{ post.excerpt }}
  {% endif %}
  <p><a href="{{ post.url | relative_url }}">Read more →</a></p>
</article>
<hr>
{% endfor %}

---

## About RemotiveLabs

[RemotiveLabs](https://www.remotivelabs.com/) provides tools for automotive software development and testing, enabling teams to test earlier, faster, and more flexibly through virtualization and infrastructure-as-code.

**Key Products:**
- **[RemotiveTopology](https://www.remotivelabs.com/products/remotivetopology)** - Infrastructure as code for vehicle platforms
- **RemotiveBroker** - Communication backbone for automotive protocols
- **RemotiveCloud** - Collaboration platform for distributed teams

[Get Started →](https://docs.remotivelabs.com/) | [GitHub →](https://github.com/remotivelabs)
