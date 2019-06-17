4.切换到项目目录，修改该目录下的urls.py文件，对应用中设定的URL进行合并。

```
(venv) $ vim oa/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('hrs/', include('hrs.urls')),
]
```

5.重新运行项目，并打开浏览器中访问[http://localhost:8000/hrs](http://localhost:8000/hrs)。

```
(venv)$ python manage.py runserver
```

6.修改views.py生成动态内容

```
(venv)$ vim hrs/views.py
from io import StringIO

from django.http import HttpResponse

depts_list = [
    {'no': 10, 'name': '财务部', 'location': '北京'},
    {'no': 20, 'name': '研发部', 'location': '成都'},
    {'no': 30, 'name': '销售部', 'location': '上海'},
]


def index(request):
    output = StringIO()
    output.write('<html>\n')
    output.write('<head>\n')
    output.write('\t<meta charset="utf-8">\n')
    output.write('\t<title>首页</title>')
    output.write('</head>\n')
    output.write('<body>\n')
    output.write('\t<h1>部门信息</h1>\n')
    output.write('\t<hr>\n')
    output.write('\t<table>\n')
    output.write('\t\t<tr>\n')
    output.write('\t\t\t<th width=120>部门编号</th>\n')
    output.write('\t\t\t<th width=180>部门名称</th>\n')
    output.write('\t\t\t<th width=180>所在地</th>\n')
    output.write('\t\t</tr>\n')
    for dept in depts_list:
        output.write('\t\t<tr>\n')
        output.write(f'\t\t\t<td align=center>{dept["no"]}</td>\n')
        output.write(f'\t\t\t<td align=center>{dept["name"]}</td>\n')
        output.write(f'\t\t\t<td align=center>{dept["location"]}</td>\n')
        output.write('\t\t</tr>\n')
    output.write('\t</table>\n')
    output.write('</body>\n')
    output.write('</html>\n')
    return HttpResponse(output.getvalue())
```

7.刷新页面查看程序的运行结果。

#### 使用视图模板



1.先回到manage.py文件所在的目录创建名为templates文件夹。

```
(venv)$ mkdir templates
```

2.创建模板页index.html。`{{ greeting }}`这样的模板占位符语法，也使用了`{% for %}`这样的模板指令，这些都是Django模板语言（DTL）的一部分

```
(venv)$ touch templates/index.html
(venv)$ vim templates/index.html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>首页</title>
</head>
<body>
	<h1>部门信息</h1>
	<hr>
	<table>
	    <tr>
            <th>部门编号</th>
            <th>部门名称</th>
            <th>所在地</th>
        </tr>
        {% for dept in depts_list %}
        <tr>
            <td>{{ dept.no }}</td>
            <td>{{ dept.name }}</td>
            <td>{{ dept.location }}</td>
        <tr>
        {% endfor %}
    </table>
</body>
</html>
```

3.回到应用目录，修改views.py文件。

```
(venv)$ vim hrs/views.py
from django.shortcuts import render

depts_list = [
    {'no': 10, 'name': '财务部', 'location': '北京'},
    {'no': 20, 'name': '研发部', 'location': '成都'},
    {'no': 30, 'name': '销售部', 'location': '上海'},
]


def index(request):
    return render(request, 'index.html', {'depts_list': depts_list})
```

4.切换到项目目录修改settings.py文件。让views.py中的`render`函数找到模板文件index.html

```
(venv)$ vim oa/settings.py
# 此处省略上面的内容

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

# 此处省略下面的内容
```

5.重新运行项目或直接刷新页面查看结果。

```
(venv)$ python manage.py runserver
```



### 配置关系型数据库MySQL

1.修改项目的settings.py文件，应用hrs添加已安装的项目中

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'hrs',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'oa',
        'HOST': 'localhost',
        'PORT': 3306,
        'USER': 'root',
        'PASSWORD': '123456',
    }
}
```

2.安装MySQL客户端工具

```
(venv)$ pip install pymysql
```

使用Python 3需要修改**项目**的`__init__.py`文件

```
import pymysql

pymysql.install_as_MySQLdb()
```

3.运行manage.py并指定migrate参数实现数据库迁移，先启动MySQL数据库服务器并创建名为oa的数据库

```
drop database if exists oa;
create database oa default charset utf8;
(venv)$ cd ..
(venv)$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying sessions.0001_initial... OK
```



4.创建自己的数据模型。在models.py中

```
(venv)$ vim hrs/models.py
from django.db import models


class Dept(models.Model):
    """部门类"""
    
    no = models.IntegerField(primary_key=True, db_column='dno', verbose_name='部门编号')
    name = models.CharField(max_length=20, db_column='dname', verbose_name='部门名称')
    location = models.CharField(max_length=10, db_column='dloc', verbose_name='部门所在地')

    class Meta:
        db_table = 'tb_dept'


class Emp(models.Model):
    """员工类"""
    
    no = models.IntegerField(primary_key=True, db_column='eno', verbose_name='员工编号')
    name = models.CharField(max_length=20, db_column='ename', verbose_name='员工姓名')
    job = models.CharField(max_length=10, verbose_name='职位')
    # 自参照完整性多对一外键关联
    mgr = models.ForeignKey('self', on_delete=models.SET_NULL, null=True, blank=True, verbose_name='主管编号')
    sal = models.DecimalField(max_digits=7, decimal_places=2, verbose_name='月薪')
    comm = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True, verbose_name='补贴')
    # 多对一外键关联
    dept = models.ForeignKey(Dept, db_column='dno', on_delete=models.PROTECT, verbose_name='所在部门')

    class Meta:
        db_table = 'tb_emp'
```

5.通过模型创建数据表。

```
(venv)$ python manage.py makemigrations hrs
Migrations for 'hrs':
  hrs/migrations/0001_initial.py
    - Create model Dept
    - Create model Emp
(venv)$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, hrs, sessions
Running migrations:
  Applying hrs.0001_initial... OK
```

### 在后台管理模型

1.创建超级管理员账号。

```
(venv)$ python manage.py createsuperuser
```

2.启动Web服务器，登录后台管理系统。

```
(venv)$ python manage.py runserver
```

3.注册模型类。

```
(venv)$ vim hrs/admin.py
from django.contrib import admin

from hrs.models import Emp, Dept

admin.site.register(Dept)
admin.site.register(Emp)
```

4.对模型进行CRUD操作。

5.注册模型管理类。修改admin.py文件，显示方式

```
from django.contrib import admin

from hrs.models import Emp, Dept


class DeptAdmin(admin.ModelAdmin):

    list_display = ('no', 'name', 'location')
    ordering = ('no', )


class EmpAdmin(admin.ModelAdmin):

    list_display = ('no', 'name', 'job', 'mgr', 'sal', 'comm', 'dept')
    search_fields = ('name', 'job')


admin.site.register(Dept, DeptAdmin)
admin.site.register(Emp, EmpAdmin)
```

### 使用ORM完成模型的CRUD操作