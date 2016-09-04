# 数据模型

该项目是一个 TODO 应用，可以对任务清单进行增加、修改、删除等，相应地，我们需要设计一个数据模型来存储相应的数据和状态。不难想到，表的字段主要有以下几个：

- id: 标识每条记录的字段，是表的主键，Integer 类型；
- title: 即任务清单的标题，String 类型；
- posted_on: 任务创建时间，DATE 类型；
- status: 任务的状态，Boolean 类型；

因此，我们的数据模型定义如下 (完整代码参考[这里](https://github.com/ethan-funny/flask-todo-app))：

```python
# -*- coding: utf-8 -*-

from application.extensions import db
from datetime import datetime

__all__ = ['Todo']

class Todo(db.Model):
    """数据模型"""
    __tablename__ = 'todo'

    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(255), nullable=False)
    posted_on = db.Column(db.Date, default=datetime.utcnow)
    status = db.Column(db.Boolean(), default=False)

    def __init__(self, *args, **kwargs):
        super(Todo, self).__init__(*args, **kwargs)

    def __repr__(self):
        return "<Todo '%s'>" % self.title
```

## 创建数据库

这里，我们使用 [Flask-Migrate](https://github.com/miguelgrinberg/Flask-Migrate) 插件管理数据库的迁移和升级，详细用法可以查看 [Flask-Migrate](https://github.com/miguelgrinberg/Flask-Migrate)。

在项目的根目录下按顺序执行如下命令，生成 migratios 文件夹、数据迁移脚本和更新数据库。

```shell$ python manage.py db init   # 初始化，生成 migratios 文件夹
  Creating directory /Users/admin/flask-todo/migrations ... done
  Creating directory /Users/admin/flask-todo/migrations/versions ... done
  Generating /Users/admin/flask-todo/migrations/alembic.ini ... done
  Generating /Users/admin/flask-todo/migrations/env.py ... done
  Generating /Users/admin/flask-todo/migrations/env.pyc ... done
  Generating /Users/admin/flask-todo/migrations/README ... done
  Generating /Users/admin/flask-todo/migrations/script.py.mako ... done
  Please edit configuration/connection/logging settings in '/Users/admin/flask-
  todo/migrations/alembic.ini' before proceeding.$ python manage.py db migrate  # 自动创建迁移脚本
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'todo'
  Generating /Users/admin/flask-todo/migrations/versions/70215a3905e0_.py ... done

$ python manage.py db upgrade   # 更新数据库
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 70215a3905e0, empty message
```

