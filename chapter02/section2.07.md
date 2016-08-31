# REST Web 服务

## 什么是 REST

REST 全称是 Representational State Transfer，翻译成中文是『表现层状态转移』，估计读者看到这个词也是云里雾里的，我当初也是！这里，我们先不纠结这个词到底是什么意思。事实上，REST 是一种 Web 架构风格，它有六条准则，满足下面六条准则的 Web 架构可以说是 Restuful 的。

1. 客户端-服务器（Client-Server）

    服务器和客户端之间有明确的界限。一方面，服务器端不再关注用户界面和用户状态。另一方面，客户端不再关注数据的存储问题。这样，服务器端跟客户端可以独立开发，只要它们共同遵守约定。

2. 无状态（Stateless）

    来自客户端的每个请求必须包含服务器所需要的所有信息，也就是说，服务器端不存储来自客户端的某个请求的信息，这些信息应由客户端负责维护。

3. 可缓存（Cachable）

    服务器的返回内容可以在通信链的某处被缓存，以减少交互次数，提高网络效率。

4. 分层系统（Layered System）

    允许在服务器和客户端之间通过引入中间层（比如代理，网关等）代替服务器对客户端的请求进行回应，而且这些对客户端来说不需要特别支持。

5. 统一接口（Uniform Interface）

    客户端和服务器之间通过统一的接口（比如 GET, POST, PUT, DELETE 等）相互通信。

6. 支持按需代码（Code-On-Demand，可选）

    服务器可以提供一些代码（比如 Javascript）并在客户端中执行，以扩展客户端的某些功能。


## 使用 Flask 提供 REST Web 服务

REST Web 服务的核心概念是资源（resources）。资源被 URI（Uniform Resource Identifier, 统一资源标识符）定位，客户端使用 HTTP 协议操作这些资源，我们用一句不是很全面的话来概括就是：URI 定位资源，用 HTTP 动词（GET, POST, PUT, DELETE 等）描述操作。下面列出了 REST 架构 API 中常用的请求方法及其含义：

| HTTP Method | Action | Example |
| :-: | :-: | :-: |
| GET | 从某种资源获取信息 | http://example.com/api/articles (获取所有文章) |
| GET | 从某个资源获取信息 | http://example.com/api/articles/1 (获取某篇文章) |
|  POST | 创建新资源 | http://example.com/api/articles (创建新文章) |
| PUT | 更新资源 |  http://example.com/api/articles/1 (更新文章) |
|  DELETE | 删除资源 | http://example.com/api/articels/1 (删除文章) |

### 设计一个简单的 Web Service

现在假设我们要为一个 blog 应用设计一个 Web Service。

首先，我们先明确访问该 Service 的根地址是什么。这里，我们可以这样定义：

```
http://[hostname]/blog/api/
```

然后，我们明确有哪些资源是要公开的。可以知道，我们这个 blog 应用的资源就是 articles。

下一步，我们要明确怎么去操作这些资源，如下所示：

| HTTP Method | URI | Action |
| :-: | :-: | :-: |
| GET | http://[hostname]/blog/api/articles |  获取所有文章列表 |
| GET |  http://[hostname]/blog/api/articles/[article_id] | 获取某篇文章内容 |
| POST |  http://[hostname]/blog/api/articles |  创建一篇新的文章 |
| PUT |  http://[hostname]/blog/api/articles/[article_id] | 更新某篇文章 |
| DELETE | http://[hostname]/blog/api/articles/[article_id] | 删除某篇文章 |

为了简便，我们定义一篇 article 的属性如下：

- id：文章的 id，Numeric 类型
- title: 文章的标题，String 类型
- content: 文章的内容，TEXT 类型

至此，我们基本完成了这个 Web Service 的设计，下面我们就来实现它。

### 使用 Flask 提供 RESTful api

在实现这个 Web 服务之前，我们还有一个问题没有考虑到：我们应该怎么存储我们的数据。毫无疑问，我们应该使用数据库，比如 MySql、MongoDB 等。但是，数据库的存储不是我们这里要讨论的重点，所以我们采用一种偷懒的做法：使用一个内存中的数据结构来代替数据库。

#### GET 方法

下面我们使用 GET 方法获取资源。

```python
# -*- coding: utf-8 -*-

from flask import Flask, jsonify, abort, make_response

app = Flask(__name__)

articles = [
    {
        'id': 1,
        'title': 'the way to python',
        'content': 'tuple, list, dict'
    },
    {
        'id': 2,
        'title': 'the way to REST',
        'content': 'GET, POST, PUT'
    }
]

@app.route('/blog/api/articles', methods=['GET'])
def get_articles():
    """ 获取所有文章列表 """
    return jsonify({'articles': articles})

@app.route('/blog/api/articles/<int:article_id>', methods=['GET'])
def get_article(article_id):
    """ 获取某篇文章 """
    article = filter(lambda a: a['id'] == article_id, articles)
    if len(article) == 0:
        abort(404)

    return jsonify({'article': article[0]})

@app.errorhandler(404)
def not_found(error):
    return make_response(jsonify({'error': 'Not found'}), 404)


if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5632, debug=True)
```

将上面的代码保存为文件 `app.py`，通过 `python app.py` 启动这个 Web Service。

接下来，我们进行测试。这里，我们采用命令行语句 [curl](https://curl.haxx.se/) 进行测试。

开启终端，敲入如下命令进行测试：

```
$ curl -i http://localhost:5632/blog/api/articles
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 224
Server: Werkzeug/0.11.4 Python/2.7.11
Date: Tue, 16 Aug 2016 15:21:45 GMT

{
  "articles": [
    {
      "content": "tuple, list, dict",
      "id": 1,
      "title": "the way to python"
    },
    {
      "content": "GET, POST, PUT",
      "id": 2,
      "title": "the way to REST"
    }
  ]
}


$ curl -i http://localhost:5632/blog/api/articles/2
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 101
Server: Werkzeug/0.11.4 Python/2.7.11
Date: Wed, 17 Aug 2016 02:37:48 GMT

{
  "article": {
    "content": "GET, POST, PUT",
    "id": 2,
    "title": "the way to REST"
  }
}


$ curl -i http://localhost:5632/blog/api/articles/3
HTTP/1.0 404 NOT FOUND
Content-Type: application/json
Content-Length: 26
Server: Werkzeug/0.11.4 Python/2.7.11
Date: Wed, 17 Aug 2016 02:32:10 GMT

{
  "error": "Not found"
}
```

上面，我们分别测试了『获取所有文章列表』、『获取某篇文章』和『获取不存在的文章』这三个功能，结果也正是我们所预料的。

#### POST 方法

下面我们使用 POST 方法创建一个新的资源。在上面的代码中添加以下代码：

```python
from flask import request

@app.route('/blog/api/articles', methods=['POST'])
def create_article():
    if not request.json or not 'title' in request.json:
        abort(400)
    article = {
        'id': articles[-1]['id'] + 1,
        'title': request.json['title'],
        'content': request.json.get('content', '')
    }
    articles.append(article)
    return jsonify({'article': article}), 201
```

测试如下：

```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"title":"the way to java"}' http://localhost:5632/blog/api/articles
HTTP/1.0 201 CREATED
Content-Type: application/json
Content-Length: 87
Server: Werkzeug/0.11.4 Python/2.7.11
Date: Wed, 17 Aug 2016 03:07:14 GMT

{
  "article": {
    "content": "",
    "id": 3,
    "title": "the way to java"
  }
}
```

可以看到，创建一篇新的文章也是很简单的。request.json 保存了请求中的 JSON 格式的数据。如果请求中没有数据，或者数据中没有 title 的内容，我们将会返回一个 "Bad Request" 的 400 错误。如果数据合法（必须要有 title 的字段），我们就会创建一篇新的文章。


#### PUT 方法

下面我们使用 PUT 方法更新文章，继续添加代码：

```python
@app.route('/blog/api/articles/<int:article_id>', methods=['PUT'])
def update_article(article_id):
    article = filter(lambda a: a['id'] == article_id, articles)
    if len(article) == 0:
        abort(404)
    if not request.json:
        abort(400)
    
    article[0]['title'] = request.json.get('title', article[0]['title'])
    article[0]['content'] = request.json.get('content', article[0]['content'])

    return jsonify({'article': article[0]})
```

测试如下：

```
$ curl -i -H "Content-Type: application/json" -X PUT -d '{"content": "hello, rest"}' http://localhost:5632/blog/api/articles/2
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 98
Server: Werkzeug/0.11.4 Python/2.7.11
Date: Wed, 17 Aug 2016 03:44:09 GMT

{
  "article": {
    "content": "hello, rest",
    "id": 2,
    "title": "the way to REST"
  }
}
```

可以看到，更新文章也是很简单的，上面我们更新了第 2 篇文章的内容。


#### DELETE 方法

下面我们使用 DELETE 方法删除文章，继续添加代码：

```python
@app.route('/blog/api/articles/<int:article_id>', methods=['DELETE'])
def delete_article(article_id):
    article = filter(lambda t: t['id'] == article_id, articles)
    if len(article) == 0:
        abort(404)
    articles.remove(article[0])
    return jsonify({'result': True})
```

测试如下：

```
$ curl -i -H "Content-Type: application/json" -X DELETE http://localhost:5632/blog/api/articles/2
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 20
Server: Werkzeug/0.11.4 Python/2.7.11
Date: Wed, 17 Aug 2016 03:46:04 GMT

{
  "result": true
}

$ curl -i http://localhost:5632/blog/api/articles
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 125
Server: Werkzeug/0.11.4 Python/2.7.11
Date: Wed, 17 Aug 2016 03:46:09 GMT

{
  "articles": [
    {
      "content": "tuple, list, dict",
      "id": 1,
      "title": "the way to python"
    }
  ]
}
```

## 附录

### 常见 HTTP 状态码

HTTP 状态码主要有以下几类：

* 1xx —— 元数据

* 2xx —— 正确的响应

* 3xx —— 重定向

* 4xx —— 客户端错误

* 5xx —— 服务端错误

常见的 HTTP 状态码可见以下表格：

| 代码 | 说明 |
| --- | --- |
| 100 | Continue。客户端应当继续发送请求。 |
| 200 | OK。请求已成功，请求所希望的响应头或数据体将随此响应返回。 |
| 201 | Created。请求成功，并且服务器创建了新的资源。  |
| 301 | Moved Permanently。请求的网页已永久移动到新位置。 服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置。 |
| 302 | Found。服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。 |
| 400 | Bad Request。服务器不理解请求的语法。  |
| 401 | Unauthorized。请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。  |
| 403 | Forbidden。服务器拒绝请求。 |
| 404 | Not Found。服务器找不到请求的网页。 |
| 500 | Internal Server Error。服务器遇到错误，无法完成请求。 |


### curl 命令参考

| 选项 | 作用 |
| :-- | :-- |
| -X | 指定 HTTP 请求方法，如 POST，GET, PUT |
| -H | 指定请求头，例如 Content-type:application/json |
| -d | 指定请求数据 |
| --data-binary | 指定发送的文件 |
| -i | 显示响应头部信息 |
| -u | 指定认证用户名与密码 |
| -v | 输出请求头部信息 |

### 完整代码

本文的完整代码可在 [Gist](https://gist.github.com/ethan-funny/cf19aa89175055de6517d851759ad743) 查看。

## 参考链接

- [理解本真的REST架构风格](http://www.infoq.com/cn/articles/understanding-restful-style)
- [基于 Flask 实现 RESTful API | Ross's Page](http://xhrwang.me/2014/12/13/restful-api-by-flask.html)
- [(译)使用Flask实现RESTful API - nummy的专栏 - SegmentFault](https://segmentfault.com/a/1190000005642670)
- [怎样用通俗的语言解释什么叫 REST，以及什么是 RESTful？ - 知乎](https://www.zhihu.com/question/28557115)
- [What Does RESTful Really Mean? - DZone Integration](https://dzone.com/articles/what-does-restful-really-mean?utm_medium=feed&utm_source=feedpress.me&utm_campaign=Feed:%20dzone)
- [HTTP状态码 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)
- [理解RESTful架构 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/09/restful.html)




