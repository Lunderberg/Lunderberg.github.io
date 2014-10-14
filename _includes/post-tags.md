{% if post.tags.size > 0 %}
  {% capture tags_content %} Tags: {% endcapture %}
  {% for post_tag in post.tags %}
    {% for data_tag in site.data.tags %}
      {% if data_tag.slug == post_tag %}
        {% assign tag = data_tag %}
      {% endif %}
    {% endfor %}
    {% if tag %}
      {% capture tags_content_temp %}{{ tags_content }}<a href="/tags/{{ tag.slug }}/">{{ tag.name }} </a> {% if forloop.last == false %}, {% endif %}{% endcapture %}
      {% assign tags_content = tags_content_temp %}
    {% endif %}
  {% endfor %}
{% else %}
  {% assign tags_content = '' %}
{% endif %}