---
title: 🎶 First post! (OMG, det sag' du bare ik!) 🎵
---

Velkommen til min blog. Her prøver jeg at skrive om ting, der interesserer mig, og sjove opdagelser.

<h1>Latest Posts</h1>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="https://0708andreas.github.io/blog{{ post.url }}">{{ post.title }}</a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
