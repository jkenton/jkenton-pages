---
layout: default

---

<div class="home">

<h3 class="page-heading">Posts</h3>

  <ul class="post-list">
    {% for post in site.posts limit:10 %}
      <li>
        <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>

        <h2>
          <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
        </h2>
      </li>
    {% endfor %}
  </ul>

</div>


