# 项目结构规范

我们在前面所举的例子基本都是写在一个单一的脚本文件中，比如 `app.py`，这在做一些简单的测试中是可行的，但是在较大的项目中则不应该这么做。好的项目结构可以让人更易于查找代码，也易于维护。当然了，每个团队都有自己的项目规范，在这里，我分享自己在平时的开发中经常用到的项目结构，仅供参考。

我们以该 [TODO](https://github.com/ethan-funny/flask-todo-app) 项目为例，介绍项目结构。

为了方便，这里使用 shell 脚本生成项目基础骨架：

```shell
# !/bin/bash

dirname=$1

if [ ! -d "$dirname" ]
then
    mkdir ./$dirname && cd $dirname
    mkdir ./application
    mkdir -p ./application/{controllers,models,static,static/css,static/js,templates}
    touch {manage.py,requirements.txt}
    touch ./application/{__init__.py,app.py,configs.py,extensions.py}
    touch ./application/{controllers/__init__.py,models/__init__.py}
    touch ./application/{static/css/style.css,templates/404.html,templates/base.html}
    echo "File created"
else
    echo "File exists"
fi
```

将上面的脚本保存为文件 `generate_flask_boilerplate.sh`，使用如下命令生成项目骨架：

```
$ sh generate_flask_boilerplate.sh flask-todo-app
```

生成的项目骨架如下所示：

```
flask-todo-app
├── application
│   ├── __init__.py
│   ├── app.py
│   ├── configs.py
│   ├── controllers
│   │   ├── __init__.py
│   ├── extensions.py
│   ├── models
│   │   ├── __init__.py
│   ├── static
│   │   ├── css
│   │   │   └── style.css
│   │   └── js
│   └── templates
│       ├── 404.html
│       ├── base.html
├── manage.py
├── requirements.txt
```

该项目骨架包含三个顶级文件(夹)：application 目录、manage.py 文件和 requirements.txt 文件，在一般情况下，我们可能还需要一个 tests 目录，存放单元测试的代码，在这里，我们没有把它包含进来。下面，我解释一下该项目骨架：

- application 目录存放 Flask 程序，包含业务逻辑代码、数据模型和静态文件等
    - configs.py 存放项目配置
    - models 目录存放数据模型文件
    - templates 目录存放模板文件
    - static 目录用于存放静态文件，如 js、css 等文件
- manage.py 用于启动我们的 Web 程序以及其他的程序任务
- requirements.txt 文件列出了项目的安装依赖包，便于在其他机器部署





