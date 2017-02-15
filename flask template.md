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

![template-flow.png](https://bitbucket.org/repo/zp95RA/images/2771160943-template-flow.png)

# References