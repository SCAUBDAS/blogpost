# Django学习篇二
### 首先解答上篇博客的相对路径问题
>>在进行相对导入时，package所对应的文件夹必须正确的被python解释器视作package,而不是普通文件夹，否则由于不被视作package，无法利用package之间的嵌套关系实现python中包的相对导入。

>>文件夹被python解释器视作package需要满足两个条件：
>>>1.文件夹中必须含有_init_.py文件，可以为空，但必须存在。

>>>2.该文件的同一根目录下的py文件不能作为主函数的入口，即不能有含有主函数的py文件。

>>只有这样，相对路径才能够导入成功，否则需要使用绝对路径或者调整项目结构。

### Django的前端编写
>>首先明确我们需要一些什么功能，什么页面，由于是做一个简单的登录系统，需要的功能和页面也很简单，即以下四个URL：
>>>`index,login,register,logut`

>>分别是主页，登录页面，注册，注销。

>>在URL中编写路由条目：

![Snipaste_2019-03-31_09-02-26](https://user-images.githubusercontent.com/38837394/55283371-0d01d580-5394-11e9-965c-214c195ca993.png)

![Snipaste_2019-03-31_09-03-04](https://user-images.githubusercontent.com/38837394/55283374-18550100-5394-11e9-804d-bfcd1e0bec9a.png)

>>路由写好之后，在view文件中构建视图框架：

![Snipaste_2019-03-31_09-07-23](https://user-images.githubusercontent.com/38837394/55283384-72ee5d00-5394-11e9-803e-b45e0257a68e.png)

>>添加如下代码：

```python
from django.shortcuts import render,redirect

def index(request):
    pass
    return render(request,'login/index.html')

def login(request):
    pass
    return render(request,'login/login.html')

def register(request):
    pass
    return render(request,'login/register.html')

def logout(request):
    pass
    return redirect('/index/')
```
### 编写静态页面
>>此时，相关配置已经全部完成，剩下的就是编写前端代码，在编写前端代码时，我们先来了解一些语法：

>>1.`{% static '相对路径' %}`，这是Django提供的链接静态页面的方法。

>>2.页面顶端需要加上`{% load staticfiles %}`，才可以使用static方法。

>>3.`{% block title %}base{% endblock %}`，设置了一个动态的页面title块

>>4.`{% block css %}{% endblock %}`，设置了一个动态的css加载块

>>5.`{% block content %}{% endblock %}`，为具体页面的主体内容留下接口

>>除了以上内容，语法与静态网页编写一致。

>>建立如下文件结构：

![Snipaste_2019-03-31_09-21-13](https://user-images.githubusercontent.com/38837394/55283456-60752300-5396-11e9-8ae4-ccc33da77015.png)

>>index的内容：

```python
{% extends 'base.html' %}
{% block title %}主页{% endblock %}
{% block content %}
    {% if request.session.is_login %}
    <h1>你好,{{ request.session.user_name }}！欢迎回来！</h1>
    {% else %}
    <h1>你尚未登录，只能访问公开内容！</h1>
    {% endif %}
{% endblock %}
```

>>login的内容：

```pythn
{% extends 'login/base.html' %}
{% load staticfiles %}
{% block title %}登录{% endblock %}
{% block css %}
    <link rel="stylesheet" href="{% static 'static/css/login.css' %}">
{% endblock %}


{% block content %}
    <div class="container">
        <div class="col-md-4 col-md-offset-4">
          <form class='form-login' action="/login/" method="post">

              {% if message %}
                  <div class="alert alert-warning">{{ message }}</div>
              {% endif %}
              {% csrf_token %}
              <h2 class="text-center">欢迎登录</h2>
              <div class="form-group">
                  {{ login_form.username.label_tag }}
                  {{ login_form.username}}
              </div>
              <div class="form-group">
                  {{ login_form.password.label_tag }}
                  {{ login_form.password }}
              </div>

              <div class="form-group">
                  {{ login_form.captcha.errors }}
                  {{ login_form.captcha.label_tag }}
                  {{ login_form.captcha }}
              </div>

              <button type="reset" class="btn btn-default pull-left">重置</button>
              <button type="submit" class="btn btn-primary pull-right">提交</button>

          </form>
        </div>
    </div> <!-- /container -->

{% endblock %}
```

>>register的内容：

```python
{% extends 'login/base.html' %}

{% block title %}注册{% endblock %}
{% block content %}
    <div class="container">
        <div class="col-md-4 col-md-offset-4">
          <form class='form-register' action="/register/" method="post">

              {% if message %}
                  <div class="alert alert-warning">{{ message }}</div>
              {% endif %}

              {% csrf_token %}

              <h2 class="text-center">欢迎注册</h2>
              <div class="form-group">
                  {{ register_form.username.label_tag }}
                  {{ register_form.username}}
              </div>
              <div class="form-group">
                  {{ register_form.password1.label_tag }}
                  {{ register_form.password1 }}
              </div>
              <div class="form-group">
                  {{ register_form.password2.label_tag }}
                  {{ register_form.password2 }}
              </div>
              <div class="form-group">
                  {{ register_form.email.label_tag }}
                  {{ register_form.email }}
              </div>
              <div class="form-group">
                  {{ register_form.sex.label_tag }}
                  {{ register_form.sex }}
              </div>
              <div class="form-group">
                  {{ register_form.captcha.errors }}
                  {{ register_form.captcha.label_tag }}
                  {{ register_form.captcha }}
              </div>

              <button type="reset" class="btn btn-default pull-left">重置</button>
              <button type="submit" class="btn btn-primary pull-right">提交</button>

          </form>
        </div>
    </div> <!-- /container -->
{% endblock %}
```

>>我们这里使用的是bootstrap框架，有内置的CSS样式，只需要下载下来引用即可。

![Snipaste_2019-03-31_09-28-04](https://user-images.githubusercontent.com/38837394/55283503-5869b300-5397-11e9-8983-351373f94007.png)

>>在根目录下创建static文件，将bootstrap解压到static文件下，在static文件下创建css文件夹和js文件夹，bootstrap框架依赖Jquery，下载Jquery，将Jquery复制到js文件夹下。自己写的css样式储存在login.css文件中。下载链接在文末。

>>每个页面中都用static方法加载了base.html页面，这是所有页面的共同部分，base.html页面的代码如下：

```python
{% load staticfiles %}
<html >
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <title>{% block title %}base{% endblock %}</title>

    <!-- Bootstrap -->
    <link href="{% static 'static/bootstrap-3.3.7-dist/css/bootstrap.css' %}" rel="stylesheet">

    <!-- HTML5 shim and Respond.js for IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
      <script src="https://cdn.bootcss.com/html5shiv/3.7.3/html5shiv.min.js"></script>
      <script src="https://cdn.bootcss.com/respond.js/1.4.2/respond.min.js"></script>
    <![endif]-->
    {% block css %}{% endblock %}
  </head>
  <body>
    <nav class="navbar navbar-default">
      <div class="container-fluid">
        <!-- Brand and toggle get grouped for better mobile display -->
        <div class="navbar-header">
          <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#my-nav" aria-expanded="false">
            <span class="sr-only">切换导航条</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="#">Mysite</a>
        </div>

        <!-- Collect the nav links, forms, and other content for toggling -->
        <div class="collapse navbar-collapse" id="my-nav">
          <ul class="nav navbar-nav">
            <li class="active"><a href="/index/">主页</a></li>
          </ul>
          <ul class="nav navbar-nav navbar-right">
            <li><a href="/login">登录</a></li>
            <li><a href="/register/">注册</a></li>
          </ul>
        </div><!-- /.navbar-collapse -->
      </div><!-- /.container-fluid -->
    </nav>

    {% block content %}{% endblock %}


    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="{% static 'static/js/jquery-3.2.1.js' %}"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="{% static 'static/bootstrap-3.3.7-dist/js/bootstrap.js' %}"></script>
  </body>
</html>
```

>>login.css样式的结果如下：

```python
body {
  background-color: #eee;
}
.form-login {
  max-width: 330px;
  padding: 15px;
  margin: 0 auto;
}
.form-login .form-control {
  position: relative;
  height: auto;
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
  padding: 10px;
  font-size: 16px;
}
.form-login .form-control:focus {
  z-index: 2;
}
.form-login input[type="text"] {
  margin-bottom: -1px;
  border-bottom-right-radius: 0;
  border-bottom-left-radius: 0;
}
.form-login input[type="password"] {
  margin-bottom: 10px;
  border-top-left-radius: 0;
  border-top-right-radius: 0;
}
```

>>最后运行，我的运行结果如下：

![Snipaste_2019-03-31_09-39-33](https://user-images.githubusercontent.com/38837394/55283573-f14cfe00-5398-11e9-95b0-34d00ce46ad5.png)

>>bootstrap:https://v3.bootcss.com/getting-started/#download

>>jquery:http://www.jq22.com/jquery-info122
