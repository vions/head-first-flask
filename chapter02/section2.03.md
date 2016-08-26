# 静态文件

静态文件，顾名思义，就是那些不会被改变的文件，比如图片，CSS 文件和 JavaScript 源码文件。默认情况下，Flask 在程序根目录中名为 static 的子目录中寻找静态文件。因此，我们一般在应用的包中创建一个叫 static 的文件夹，并在里面放置我们的静态文件。比如，我们可以按下面的结构组织我们的 app：

```python
app/
    __init__.py
    static/
        css/
            style.css
            home.css
            admin.css
        js/
            home.js
            admin.js
        img/
            favicon.co
            logo.svg
    templates/
        index.html
        home.html
        admin.html
    views/
    models/
run.py

```

但是，我们有时还会应用到第三方库，比如 jQuery, Bootstrap 等，这时我们为了不跟自己的 Javascript 和 CSS 文件混起来，我们可以将这些第三方库放到 lib 文件夹或者 vendor 文件夹，比如下面这种：

```python
static/
    css/
        lib/
            bootstrap.css
        style.css
        home.css
        admin.css
    js/
        lib/
            jquery.js
            chart.js
        home.js
        admin.js
    img/
        logo.svg
        favicon.ico
```

## 提供一个 favicon 图标

favicon 是 favorites icon 的缩写，也被称为 website icon（网页图标）、page icon（页面图标）等。通常而言，定义一个 favicon 的方法是将一个名为『favicon.ico』的文件置于 Web 服务器的根目录下。但是，正如我们在上面指出，我们一般将图片等静态资源放在一个单独的 static 文件夹中。为了解决这种不一致，我们可以在站点模板的 <head> 部分添加两个 link 组件，比如我们可以在 template/base.html 中定义 favicon 图标：

```html
{% block head %}
{{ super() }}
<link rel="shortcut icon" href="{{ url_for('static', filename = 'favicon.ico') }}" type="image/x-icon">
<link rel="icon" href="{{ url_for('static', filename = 'favicon.ico') }}" type="image/x-icon">
{% endblock %}
```

在上面的代码中，我们使用了 `super()` 来保留基模板中定义的块的原始内容，并添加了两个 link 组件声明图标位置，这两个 link 组件声明会插入到 head 块的末尾。

