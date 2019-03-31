# diango学习篇一
### 使用pycharm创建Diango项目
>>这里只需要在pycharm自带的terminal中输入`pip install Diango`后创建即可，初始的项目文件夹目录如下：

![Snipaste_2019-03-25_18-46-46](https://user-images.githubusercontent.com/38837394/55272984-09247380-5300-11e9-887a-a2acdf7897a0.png)


>>随后在terminal中输入`python manage.py startapp 你创建的文件名`就可以得到一个app，文件的目录如下：

![Snipaste_2019-03-25_18-56-19](https://user-images.githubusercontent.com/38837394/55273008-5accfe00-5300-11e9-80a3-bda94cfbfc9b.png)



### 项目语言配置
>>在settings文件中语言默认是英文，可已将它改为中文。
>>>未改前：
```python
LANGUAGE_CODE = 'en-us'
TIME_ZOME = 'UTC'
USE_I18N = True
USE_L10N = True
USE-TZ = True
```
>>>改正后：

![Snipaste_2019-03-25_19-06-32](https://user-images.githubusercontent.com/38837394/55273011-6caea100-5300-11e9-9367-2b97756596c8.png)

>>现在可以来编写你的代码了。

>>连接数据库，要求已安装mysql，使用`pip install pymysql`安装所需的第三方库，在settings文件中添加数据库配置，如下：
```python
DATABASES = {
    'default':{
        'ENGINE':'dango.db.backends.mysql',
        'NAME':'数据库名称'
        'USER'：'用户名'
        'PASSWORD':'密码'
        'HOST':'IP地址'
        'PORT':'端口'
    }
}
```
>>PS:数据库名称，用户名，密码。IP地址，端口都是在安装时配置的，其中用户名默认是root，IP地址是localhost，端口是'3306'
## 数据库模型设计
>>我们这里做一个简单的登录系统，那么我们需要一张表User，他需要有如下属性：用户名，密码，邮箱，性别，创建时间。数据库模型的创建在你创建的APP中的model中创建。

![Snipaste_2019-03-25_21-15-10](https://user-images.githubusercontent.com/38837394/55273017-8819ac00-5300-11e9-848a-75c261c69ed5.png)

>>创建代码如下：
```python

class User(models.Model):
    '''用户表'''

    gender = (
        ('male','男'),
        ('female','女'),
    )

    name = models.CharField(max_length=128,unique=True)
    password = models.CharField(max_length=256)
    email = models.EmailField(unique=True)
    sex = models.CharField(max_length=32,choices=gender,default='男')
    c_time = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name

    class Meta:
        ordering = ['c_time']
        verbose_name = '用户'
        verbose_name_plural = '用户'

```

>>数据库模型创建好之后，需要将数据迁移。首先在seetings文件中注册创建的APP。

![Snipaste_2019-03-25_21-26-02](https://user-images.githubusercontent.com/38837394/55273035-a97a9800-5300-11e9-9cf8-1c8370de2d5a.png)

>>如图所示，在相应位置加上所创建的数据库名称。然后就可以利用terminal迁移数据库。依次执行下列语句：

```python
python manage.py makemigrations
python manage.py migtate
```

>>在admin文件中注册该数据库模型

![Snipaste_2019-03-25_21-32-00](https://user-images.githubusercontent.com/38837394/55273056-e3e43500-5300-11e9-8ed6-8fd771c53c1d.png)


![Snipaste_2019-03-25_21-31-11](https://user-images.githubusercontent.com/38837394/55273045-c44d0c80-5300-11e9-9b2e-a66f5653ead8.png)

>>然后在terminal中创建管理员：`python manage.py createsuperuser`

![Snipaste_2019-03-25_21-35-21](https://user-images.githubusercontent.com/38837394/55273061-f6f70500-5300-11e9-8987-51d398835980.png)

>>完成这些之后可以开始准备前端的编写了。

>>ps:在进行项目开发的过程中有一个值得注意的地方，那就是相对路径的使用，推荐阅读下面两篇官方解释，具体我们下期再说。

[官方解释1](https://python3-cookbook.readthedocs.io/zh_CN/latest/c10/p03_import_submodules_by_relative_names.html)

[官方解释2](http://www.pythondoc.com/pythontutorial3/modules.html)

[参考文档](https://blog.csdn.net/bysjlwdx/article/details/80684320)