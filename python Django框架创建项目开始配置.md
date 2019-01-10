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

​	