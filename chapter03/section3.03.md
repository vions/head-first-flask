# 编写业务逻辑

我们的业务逻辑代码主要在 controllers 目录中，新建一个 `todo.py` 文件, 核心代码如下 (完整代码参考[这里](https://github.com/ethan-funny/flask-todo-app))，代码说明可以参考注释：

```python
# -*- coding: utf-8 -*-

import flask
from flask import request, redirect, flash, render_template, url_for
from application.extensions import db
from application.models import Todo

todo_bp = flask.Blueprint(
    'todo',
    __name__,
    template_folder='../templates'
)

# 主页
@todo_bp.route('/', methods=['GET', 'POST'])
def index():
    todo = Todo.query.order_by('-id')
    _form = request.form

    if request.method == 'POST':
        # 添加任务
        title = _form["title"]
        td = Todo(title=title)
        try:
            td.store_to_db()  # 将数据保存到数据库
            flash("add task successfully!")
            return redirect(url_for('todo.index'))
        except Exception as e:
            print e
            flash("fail to add task!")

    return render_template('index.html', todo=todo)

# 删除任务
@todo_bp.route('/<int:todo_id>/del')
def del_todo(todo_id):
    todo = Todo.query.filter_by(id=todo_id).first()
    if todo:
        todo.delete_todo()
    flash("delete task successfully")
    return redirect(url_for('todo.index'))

# 编辑 (更新) 任务
@todo_bp.route('/<int:todo_id>/edit', methods=['GET', 'POST'])
def edit(todo_id):
    todo = Todo.query.filter_by(id=todo_id).first()

    if request.method == 'POST':
        Todo.query.filter_by(
            id=todo_id
        ).update({
            Todo.title: request.form['title']
        })
        db.session.commit()
        flash("update successfully!")
        return redirect(url_for('todo.index'))

    return render_template('edit.html', todo=todo)

# 标记任务完成
@todo_bp.route('/<int:todo_id>/done')
def done(todo_id):
    todo = Todo.query.filter_by(id=todo_id).first()
    if todo:
        Todo.query.filter_by(id=todo_id).update({Todo.status: True})
        db.session.commit()
        flash("task is completed!")

    return redirect(url_for('todo.index'))

# 重置任务
@todo_bp.route('/<int:todo_id>/redo')
def redo(todo_id):
    todo = Todo.query.filter_by(id=todo_id).first()
    if todo:
        Todo.query.filter_by(id=todo_id).update({Todo.status: False})
        flash("redo successfully!")
        db.session.commit()

    return redirect(url_for('todo.index'))

# 404 处理
@todo_bp.errorhandler(404)
def page_not_found():
    return render_template('404.html'), 404
```

上面只是核心的业务逻辑，关于模板渲染，数据模型的定义等可以参考[完整代码](https://github.com/ethan-funny/flask-todo-app)。

由于该项目的业务逻辑比较简单，因此也就不做过多介绍，相信读者可以轻松看懂代码，如果有问题，也可以在 github 上面提 issue，欢迎指正。


