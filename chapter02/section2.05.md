# 请求、重定向及会话

Web 开发中经常需要处理 HTTP 请求、重定向和会话等诸多事务，相应地，Flask 也内建了一些常见的对象如 request, session, redirect 等对它们进行处理。

## 请求对象 request

HTTP 请求方法有 GET、POST、PUT 等，request 对象也相应地提供了支持。举个例子，假设现在我们开发一个功能：用户注册。如果 HTTP 请求方法是 POST，我们就注册该用户，如果是 GET 请求，我们就显示注册的字样。代码示例如下（注意，下面代码并不能直接运行，文末提供了完整的代码）：

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/register', methods=['POST', 'GET']):
def register():
    if request.method == 'GET':
        return 'please register!'
    elif request.method == 'POST':
        user = request.form['user']
        return 'hello', user
```

## 重定向对象 redirect

当用户访问某些网页时，如果他还没登录，我们往往会把网页**重定向**到登录页面，Flask 提供了 redirect 对象对其进行处理，我们对上面的代码做一点简单的改造，如果用户注册了，我们将网页重定向到首页。代码示例如下：

```python
from flask import Flask, request, redirect

app = Flask(__name__)

@app.route('/home', methods=['GET']):
def index():
    return 'hello world!'

@app.route('/register', methods=['POST', 'GET']):
def register():
    if request.method == 'GET':
        return 'please register!'
    elif request.method == 'POST':
        user = request.form['user']
        return redirect('/home')
```

## 会话对象 session

程序可以把数据存储在**用户会话**中，用户会话是一种私有存储，默认情况下，它会保存在客户端 cookie 中。Flask 提供了 session 对象来操作用户会话，下面看一个示例：

```python
from flask import Flask, request, session, redirect, url_for, render_template

app = Flask(__name__)

@app.route('/home', methods=['GET'])
def index():
    return 'hello world!'
    
@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        user_name = request.form['user']
        session['user'] = user_name
        return 'hello, ' + session['user']
    elif request.method == 'GET':
        if 'user' in session:
            return redirect(url_for('index'))
        else:
            return render_template('login.html')

app.secret_key = '123456'

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5632, debug=True)
```

操作 `session` 就像操作 python 中的字典一样，我们可以使用 `session['user']` 获取值，也可以使用 `session.get('user')` 获取值。注意到，我们使用了 `url_for` 生成 URL，比如 `/home` 写成了 `url_for('index')`。`url_for()` 函数的第一个且唯一必须指定的参数是端点名，即路由的内部名字。默认情况下，路由的端点是相应视图函数的名字，因此 `/home` 应该写成 `url_for('index')`。还有一点，使用`session` 时要设置一个密钥 `app.secret_key`。

## 附录

本节完整的代码如下：

```python
$ tree .
.
├── flask-session.py
└── templates
    ├── layout.html
    └── login.html
    
$ cat flask-session.py
from flask import Flask, request, session, redirect, url_for, render_template

app = Flask(__name__)

@app.route('/')
def head():
    return redirect(url_for('register'))

@app.route('/home', methods=['GET'])
def index():
    return 'hello world!'
    
@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        user_name = request.form['user']
        session['user'] = user_name
        return 'hello, ' + session['user']
    elif request.method == 'GET':
        if 'user' in session:
            return redirect(url_for('index'))
        else:
            return render_template('login.html')

app.secret_key = '123456'

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5632, debug=True)
    
$ cat layout.html
<!doctype html>
<title>Hello Sample</title>
<link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='style.css') }}">
<div class="page">
    {% block body %}
    {% endblock %}
</div>

$ cat login.html
{% extends "layout.html" %}
{% block body %}
<form name="register" action="{{ url_for('register') }}" method="post">
    Hello {{ title }}, please login by:
    <input type="text" name="user" />
</form>
{% endblock %}
```

