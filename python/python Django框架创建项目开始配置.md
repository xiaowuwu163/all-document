# python Django框架创建项目开始配置

## 一、django-admin命令使用

#### 1、创建django工程

​	django-admin startproject  project-name



#### 2、创建app 

​	cd 工程名

​	python manage.py startapp app-name



### 二、配置文件

##### 1、配置模板路径，templates默认已配置好，若为其他模板名，则打开setting.py文件，设置

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
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
```

##### 2、配置静态目录

##### 	在setting.py文件最后加上

```
STATICFILES_DIRS = [os.path.join(BASE_DIR,'static')]
```

三 、 MTV框架模型

M -- model  数据模型

T -- templates  视图模板

V -- view 逻辑控制层，定义视图函数



##### 3、上传文件注意事项

1、需要在form表单中添加属性enctype="multipart/form-data" 否则后台不能获取上传文件的数据



2、上传文件用request.FILES 获取文件数据

##### 4、日期时间区设置

```
LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'Asia/Shanghai'

DATETIME_FORMAT = 'Y-m-d H:i:s'

USE_I18N = True

USE_L10N = False

USE_TZ = False
```

##### 5、邮箱配置	

```
# 邮箱配置
EMAIL_USE_SSL = True
EMAIL_HOST = 'smtp.exmail.qq.com'  # 如果是 163 改成 smtp.163.com
# EMAIL_HOST = 'smtp.qq.com'  # 如果是 163 改成 smtp.163.com
EMAIL_PORT = 465
EMAIL_HOST_USER = '2128@kingon.cn' # 帐号
EMAIL_HOST_PASSWORD = ''  # 密码
DEFAULT_FROM_EMAIL = '监控主机<2128@kingon.cn>' #默认主题显示
```

6、mysql 数据库连接设置

```
DATABASES = {
    # 'default': {
    #     'ENGINE': 'django.db.backends.sqlite3',
    #     'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    # }

    'default': {
                'ENGINE': 'django.db.backends.mysql',
                'NAME': 'KingonPaas',
                'HOST':"172.10.0.103",
                'PORT':3306,
                'USER':"root",
                'PASSWORD':"kingon", #这里要用引号
            }
}
```

在projectName 目录下的_ _init_ _ _.py文件中添加

```
import pymysql

pymysql.install_as_MySQLdb()
```

7、登录验证和创建加密用户模块

```
from django.contrib.auth.decorators import login_required
from django.contrib.auth import login,authenticate,logout
from django.contrib.auth.models import User
from django.views.decorators.csrf import csrf_exempt

@login_required(login_url='/login.html')

@csrf_exempt
def page_not_found(request):
    return render_to_response('404.html')
    
 user = User.objects.create_user(username='kingon',password='12345') #create_user()创建用户，秘密你是加密，Django验证登录必须是加密密码
```