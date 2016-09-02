# 部署

我们这里以项目 [flask-todo-app](https://github.com/ethan-funny/flask-todo-app) 为例，介绍如何将其部署到生产环境，主要有以下几个步骤：

- 创建项目的运行环境
- 使用 Gunicorn 启动 flask 程序
- 使用 supervisor 管理服务器进程
- 使用 Nginx 做反向代理

## 创建项目的运行环境

- 创建 Python 虚拟环境，以便隔离不同的项目
- 安装项目依赖包

```
$ pip install virtualenvwrapper
$ source /usr/local/bin/virtualenvwrapper.sh
$ mkvirtualenv flask-todo-env   # 创建完后，会自动进入到该虚拟环境，以后可以使用 workon 命令
$ 
(flask-todo-env)$ git clone https://github.com/ethan-funny/flask-todo-app
(flask-todo-env)$ cd flask-todo-app
(flask-todo-env)$ pip install -r requirements.txt
```

## 使用 Gunicorn 启动 flask 程序

我们在本地调试的时候经常使用命令 `python manage.py runserver` 或者 `python app.py` 等启动 Flask 自带的服务器，但是，Flask 自带的服务器性能无法满足生产环境的要求，因此这里我们采用 [Gunicorn](http://gunicorn.org/) 做 wsgi (Web Server Gateway Interface，Web 服务器网关接口) 容器，假设我们以 root 用户身份进行部署：

```
(flask-todo-env)$ pip install gunicorn
(flask-todo-env)$ /home/root/.virtualenvs/flask-todo-env/bin/gunicorn -w 4 -b 127.0.0.1:7345 application.app:create_app()
```

上面的命令中，-w 参数指定了 worker 的数量，-b 参数绑定了地址（包含访问端口）。

需要注意的是，由于我们这里将 Gunicorn 绑定在本机 127.0.0.1，因此它仅仅监听来自服务器自身的连接，也就是我们从外网访问该服务。在这种情况下，我们通常使用一个反向代理来作为外网和 Gunicorn 服务器的中介，而这也是推荐的做法，接下来也会介绍如何使用 nginx 做反向代理。不过，有时为了调试方便，我们可能需要从外网发送请求给 Gunicorn，这时我们可以让 Gunicorn 绑定 0.0.0.0，这样它就会监听来自外网的所有请求。

## 使用 supervisor 管理服务器进程

在上面，我们手动使用命令启动了 flask 程序，当程序挂掉的时候，我们又要再启动一次。另外，当我们想关闭程序的时候，我们需要找到 pid 进程号并 kill 掉。这里，我们采用一种更好的方式来管理服务器进程，我们将 supervisor 安装全局环境下，而不是在当前的虚拟环境：

```
$ pip install supervisor
$ echo_supervisord_conf > supervisor.conf   # 生成 supervisor 默认配置文件
$ vi supervisor.conf    # 修改 supervisor 配置文件，添加 gunicorn 进程管理
```

在 `supervisor.conf` 添加以下内容：

```
[program:flask-todo-env]
directory=/home/root/flask-todo-app
command=/home/root/.virtualenvs/%(program_name)s/bin/gunicorn
  -w 4
  -b 127.0.0.1:7345
  --max-requests 2000
  --log-level debug
  --error-logfile=-
  --name %(program_name)s
  "application.app:create_app()"

environment=PATH="/home/root/.virtualenvs/%(program_name)s/bin"
numprocs=1
user=deploy
autostart=true
autorestart=true
redirect_stderr=true
redirect_stdout=true
stdout_logfile=/home/root/%(program_name)s-out.log
stdout_logfile_maxbytes=100MB
stdout_logfile_backups=10
stderr_logfile=/home/root/%(program_name)s-err.log
stderr_logfile_maxbytes=100MB
stderr_logfile_backups=10
```

supervisor 的常用命令如下：

```
supervisord -c supervisor.conf                             通过配置文件启动 supervisor
supervisorctl -c supervisor.conf status                    查看 supervisor 的状态
supervisorctl -c supervisor.conf reload                    重新载入 配置文件
supervisorctl -c supervisor.conf start [all]|[appname]     启动指定/所有 supervisor 管理的程序进程
supervisorctl -c supervisor.conf stop [all]|[appname]      关闭指定/所有 supervisor 管理的程序进程
supervisorctl -c supervisor.conf restart [all]|[appname]   重启指定/所有 supervisor 管理的程序进程
```

## 使用 Nginx 做反向代理

将 Nginx 作为反向代理可以处理公共的 HTTP 请求，发送给 Gunicorn 并将响应带回给发送请求的客户端。在 ubuntu 上可以使用 `sudo apt-get install nginx` 安装 nginx，其他系统也类似。

要想配置 Nginx 作为运行在 127.0.0.1:7345 的 Gunicorn 的反向代理，我们可以在 /etc/nginx/sites-enabled 下给应用创建一个文件，不妨称之为 flask-todo-app.com，nginx 的类似配置如下：

```
# Handle requests to exploreflask.com on port 80
server {
    listen 80;
    server_name flask-todo-app.com;

    # Handle all locations
    location / {
        # Pass the request to Gunicorn
        proxy_pass http://127.0.0.1:7345;

        # Set some HTTP headers so that our app knows where the request really came from
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

常用的 nginx 使用命令如下：

```
$ sudo service nginx start
$ sudo service nginx stop
$ sudo service nginx restart
```

可以看到，我们上面的部署方式，都是手动部署的，如果有多台服务器要部署上面的程序，那就会是一个恶梦，有一个自动化部署的神器 [Fabric](http://www.fabfile.org/) 可以帮助我们解决这个问题，感兴趣的读者可以了解一下。

## 参考资料

- [部署 | Flask之旅](https://spacewander.github.io/explore-flask-zh/14-deployment.html)
- [python web 部署：nginx + gunicorn + supervisor + flask 部署笔记 - 简书](http://www.jianshu.com/p/be9dd421fb8d)
- [新手教程：建立网站的全套流程与详细解释 | 谢益辉](http://yihui.name/cn/2009/06/how-to-build-a-website-as-a-dummy/)


