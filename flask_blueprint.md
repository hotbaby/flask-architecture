# Introduction

Flask蓝图的用途：

* 将应用程序映射到蓝图中
* 在URL前缀或子域上注册蓝图
* 在不同的URL规则上多次注册蓝图
* 提供模板过滤器，静态文件，模板，以及其他实用工程

Flask使用Blueprint（蓝图）将视图关联起来。

# Examples

```
import flask
from flask import abort
from flask import Flask
from flask import Blueprint
from flask import render_template
from jinja2 import TemplateNotFound

app = Flask(__name__)

bp = Blueprint('home', __name__)

@bp.route('/')
def index():
    try:
        return render_template('index.html')
    except TemplateNotFound:
        abort(404)

app.register_blueprint(bp)
```

# References

* [flask blueprint](http://flask.pocoo.org/docs/0.12/blueprints/)
* [jinja](http://jinja.pocoo.org/)
