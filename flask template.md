# Introduction

Flask使用`Jinja2`作为模板引擎。

# Syntax

* `{% ... %}` for statements
* `{{ ... }}` for Expressions to print the template output
* `{# ... #}` for Comments not included in the template output
* `# ... ##` for Line  Statements

## Example

```
{% extends "layout.html" %}
{% block body %}
  <ul>
  {% for user in users %}
    <li><a href="{{ user.url }}">{{ user.username }}</a></li>
  {% endfor %}
  </ul>
{% endblock %}
```

# Architecture

**data flow**

![template-flow.png](https://bytebucket.org/hotbaby/resource/raw/da7f226e5e9842ecdfaec267bd3076dbd3b448ab/flask/flask-template-flow.png?token=4937b72f9e36d35a1d9639023730d4497a4ac475)

# References
