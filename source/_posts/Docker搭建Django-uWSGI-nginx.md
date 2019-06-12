---
title: Docker搭建Django+uWSGI+nginx
date: 2019-06-04 14:46:16
tags:
- Docker
- Django
categories:
- Python
---

## 整体框架

使用`docker-compose`，将`Django`、`MySQL`与`Nginx`分别使用不同的`container`，当做`Docker`学习的练习。

![项目结构](/assets/结构图.jpg)

<!--more-->

## `Django`

```shell 
pure_django_test
├── Dockerfile
├── fun
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── middlewares.py
│   ├── migrations
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
├── pure_django_test
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── requirements.txt
├── start.sh
├── templates
│   └── hello.html
└── uwsgi.ini
```

首先是`Dockerfile`的编写，放置于根目录下：

```yaml 
FROM python:3.6

WORKDIR /pure_django_test

COPY . /pure_django_test

RUN pip3 install -r requirements.txt

EXPOSE 8001
```

接着编写`uwsgi.ini`：

```ini 
[uwsgi]
http = :8001
chdir = %d
wsgi-file =  %dpure_django_test/wsgi.py
py-autoreload = 1
```

> 可以使用`%d`表示当前目录，`py-autoreload`可以使得当代码发生变化之后重启`uWSGI`。

需要启动或者重启时进行数据库迁移以及启动`uWSGI`，所以还需要一个`start.sh`:

```shell 
python manage.py makemigrations&&
python manage.py migrate&&
uwsgi --ini uwsgi.ini
```

## `Nginx`

目录包括：

```shell 
nginx
├── Dockerfile
└── nginx_app.conf
```

首先仍然是`Dockerfile`的编写：

```yaml 
FROM nginx
EXPOSE 80 8001
RUN rm /etc/nginx/conf.d/default.conf

COPY nginx_app.conf /etc/nginx/conf.d/
RUN mkdir -p /usr/share/nginx/html/static
```

接下来是`nginx_app.conf`，替换掉默认的配置：

```shell
upstream django {
    server web:8001;
}
server {
    listen      80;
    server_name localhost;

    location /static {
        alias /usr/share/nginx/html/static;
    }

    location / {
        proxy_pass http://django;
        proxy_set_header Host $host;
        proxy_redirect off;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
```

> 在`upstream`中使用`web`而不是`localhost`，否则会出现`502`错误。

## `docker-compose.yml`

因为数据库直接由镜像构建而来，因此不需要写`Dockerfile`，只需要在`yaml`中指定一下：

```yaml 
version: "3"
services:
  db:
    image: mysql:5.7
    environment:
      - MYSQL_HOST=localhost
      - MYSQL_DATABASE=pure_django
      - MYSQL_USER=root
      - MYSQL_PASSWORD=littlemay
      - MYSQL_ROOT_PASSWORD=littlemay

    volumes:
      - ./data:/var/lib/mysql
    restart: always

  web:
    build: ./pure_django_test
    volumes:
      - ./pure_django_test:/pure_django_test
    command: bash start.sh
    links:
      - db
    depends_on:
      - db
    restart: always

  nginx:
    build: ./nginx
    ports:
      - "80:80"
    volumes:
      - ./blog/static:/usr/share/nginx/html/static
    links:
      - web
    depends_on:
      - web
    restart: always
```

此时还需要修改`Django`中的`settings.py`：

```python 
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'pure_django',
        'USER': 'root',
        'PASSWORD': 'littlemay',
        'HOST': 'db',
        'PORT': '3306',
        'OPTIONS': {'charset': 'utf8mb4'},
    }
}
```

> 要记得在`db`中先创建数据库。

将文件挂载到本地，使用`docker-compose up`之后，可以边修改边查看效果。