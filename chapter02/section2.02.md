# 路由和视图

我们在[前面的一小节](./section2.01.md)介绍了一个简单的 Flask 程序是怎么运行的。其中，有三行代码，我们并没有深入讲解。在这里，我们就对它们进行深入解析。回顾这三行代码：

```python
@app.route("/")
def hello():
	return "Hello World!"
```

这三行代码的意思就是：如果浏览器要访问服务器程序的根地址（"/"），那么 Flask 程序实例就会执行函数 `hello()` ，返回『Hello World!』。

也就是说，**上面三行代码定义了一个 URL 到 Python 函数的映射关系**，我们将处理这种映射关系的程序称为『路由』，而 `hello()` 就是视图函数。


## 动态路由

假设服务器域名为 `https://hello.com`, 我们来看下面一个路由：

```python
@app.route("/ethan")
def hello():
    return '<h1>Hello, ethan!</h1>'
```

再来看一个路由：

```python
@app.route("/peter")
def hello():
    return '<h1>Hello, peter!</h1>'
```

可以看到，上面两个路由的功能是当用户访问 `https://hello.com/<user_name>` 时，网页显示对该用户的问候。按上面的写法，如果对每个用户都需要写一个路由，那么 100 个用户岂不是要写 100 个路由！这当然是不能忍受的，实际上一个路由就够了！且看下面：

```python
@app.route("/user_name")
def hello(user_name):
    return '<h1>Hello, %s!</h1>' % user_name
```

现在，任何类似 `https://hello.com/<user_name>` 的 URL 都会映射到这个路由上，比如 `https://hello.com/ethan-funny`，`https://hello.com/torvalds`，访问这些 URL 都会执行上面的路由程序。

也就是说，Flask 支持这种动态形式的路由，路由中的动态部分默认是字符串，像上面这种情况。当然，除了字符串，Flask 也支持在路由中使用 int、float，比如路由 /articles/<int:id> 只会匹配动态片段 id 为整数的 URL，例如匹配 https://hello.com/articles/100，https://hello.com/articles/101，但不匹配 https://hello.com/articles/the-first-article 这种 URL。


