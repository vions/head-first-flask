# 一个最简单的应用

我们在第 1 章已经看到了一个简单的 Hello World 的例子，相信你已经成功地把它跑起来了，下面我们对这个程序进行讲解。回顾一下这个程序：

```python
$ cat hello.py

from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
	return "Hello World!"

if __name__ == "__main__":
	app.run()
```

- 先看程序的第 1 句：

```python
from flask import Flask
```

该句从 flask 包导入了一个 Flask 类，这也是后面构建 Flask Web 程序的基础。

- 接着看程序的第 2 句：

```python
app = Flask(__name__)
```

上面这一句通过将 `__name__` 参数传给 Flask 类的构造函数，创建了一个程序实例 `app`，也就创建了一个 Flask 集成的开发 Web 服务器。**Flask 用 `__name__` 这个参数决定程序的根目录，以便程序能够找到相对于程序根目录的资源文件位置，比如静态文件等。**

- 接着看程序的第 3，4，5 句：

```python
@app.route("/")
def hello():
	return "Hello World!"
```

可能读者会对这三句感到很困惑：它们的作用是什么呢？我们知道，Web 浏览器把请求发送给 Web 服务器，Web 服务器再把请求发送给 Flask 程序实例，那么程序实例就需要知道对每个 URL 请求应该运行哪些代码。

上面这三句代码的意思就是：如果浏览器要访问服务器程序的根地址（"/"），那么 Flask 程序实例就会执行函数 `hello()` ，返回『Hello World!』。

比如，假设我们部署程序的服务器域名为 `www.hello.com`，当我们在浏览器访问 http://
www.hello.com（也就是根地址）时，会触发 Flask 程序执行 `hello()` 这个函数，返回『Hello World!』，**这个函数的返回值称为响应，是客户端接收到的内容。**

但是，如果我们在浏览器访问 http://www.hello.com/peter 时，程序会返回 `404` 错误，因为我们的 Flask 程序并没有对这个 URL 指定处理函数，所以会返回错误代码。

- 接着看程序的最后两句：

```python
if __name__ == "__main__":
	app.run()
```

上面两句的意思，当我们运行该脚本的时候（第 1 句），启动 Flask 集成的开发 Web 服务器（第 2 句）。默认情况下，改服务器会监听本地的 5000 端口，如果你想改变端口的话，可以传入 "port=端口号"，另外，如果你想支持远程，需要传入 "host=0.0.0.0"，你还可以设置调试模式，如下：

```
app.run(host='0.0.0.0', port=8234, debug=True)
```

服务器启动后，程序会进入轮询，等待并处理请求。轮询会一直运行，直到程序被终止。需要注意的是，Flask 提供的 Web 服务器不适合在生产环境中使用，后面我们会介绍生产环境中的 Web 服务器。

OK，到此为止，我们基本明白一个简单的 Flask 程序是怎么运作的了，后面我们就一起慢慢揭开 Flask 的神秘面纱吧~~

