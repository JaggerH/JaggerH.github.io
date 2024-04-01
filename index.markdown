---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

<div class="posts-list">
    {% for post in site.posts %}
        <div class="post">
            <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
            <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
            <p>{{ post.excerpt }}</p>
        </div>
    {% endfor %}
</div>