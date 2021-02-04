# python
시작부터 진행 방식
1. File에서 New 프로젝트 생성
2. pip install django # 가상환경 설치
3. pip list #설치된 목록 보여줌
4. django-admin startproject mysite(임의지정)
5. cd mysite # 위에서 만든 주소로 이동
6. python manage.py migrate # DB 생성
7. python manage.py createsuperuser  # 관리자 생성
       user id 및 password 등록
8. python manage.py startapp bookmark # 북마크앱 생성
9. mysite.settings.py에서 installed_apps에 추가
10. bookmark.models.py에서 
----------------------------------
from django.db import models

class Bookmark(models.Model): #DB 생성
    title = models.CharField('TITLE', max_length=100, blank=True)
    url = models.URLField('URL', unique=True)

    def __str__(self):
        return self.title
------------------------------------
11. python manage.py makemigrations bookmark
12. python manage.py migrate
13. python manage.py runserver

=====================================
오후 과정 
1. python manage.py startapp blog
2. mysite.settings.py에서 installed_apps에 추가 'blog.apps.BlogConfig',
3. python manage.py makemigrations blog
4. python manage.py migrate blog
5. blog폴더의 models.py에 붙여넣기
----------------------------------
from django.db import models
from django.urls import reverse


class Post(models.Model):
    title = models.CharField(verbose_name='TITLE', max_length=50)
    slug = models.SlugField('SLUG', unique=True, allow_unicode=True, help_text='one word for title alias.')
    description = models.CharField('DESCRIPTION', max_length=100, blank=True, help_text='simple description text.')
    content = models.TextField('CONTENT')
    create_dt = models.DateTimeField('CREATE DATE', auto_now_add=True)
    modify_dt = models.DateTimeField('MODIFY DATE', auto_now=True)

    class Meta:
        verbose_name = 'post'
        verbose_name_plural = 'posts'
        db_table = 'blog_posts'
        ordering = ('-modify_dt',)

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('blog:post_detail', args=(self.slug,))

    def get_previous(self):
        return self.get_previous_by_modify_dt()

    def get_next(self):
        return self.get_next_by_modify_dt()


----------------------------------
6. blog 폴더의 admin.py에 붙여넣기
-----------------------------------
from django.contrib import admin
from blog.models import Post


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display  = ('id', 'title', 'modify_dt')
    list_filter   = ('modify_dt',)
    search_fields = ('title', 'content')
    prepopulated_fields = {'slug': ('title',)}

-----------------------------------
7. mysite의 urls.py에서 include로 변경 & 북마크와 블로그 폴더에 urls.py를 생성하고 ;; 교안 5,6,7page
------------------------------------
#mysite urls.py
from django.urls import path, include


urlpatterns = [
    path('admin/', admin.site.urls),
    # shkim
    path('bookmark/', include('bookmark.urls')),
    path('blog/', include('blog.urls')),
]
------------------------------------
------------------------------------
#bookmark urls.py
from django.urls import path
from bookmark.views import BookmarkLV, BookmarkDV


app_name = 'bookmark'
urlpatterns = [
    path('', BookmarkLV.as_view(), name='index'),
    path('<int:pk>/', BookmarkDV.as_view(), name='detail'),
]
------------------------------------
------------------------------------
#blog urls.py
from django.urls import path, re_path
from blog import views


app_name = 'blog'
urlpatterns = [

    # Example: /blog/
    path('', views.PostLV.as_view(), name='index'),

    # Example: /blog/post/ (same as /blog/)
    path('post/', views.PostLV.as_view(), name='post_list'),

    # Example: /blog/post/django-example/
    re_path(r'^post/(?P<slug>[-\w]+)/$', views.PostDV.as_view(), name='post_detail'),

    # Example: /blog/archive/
    path('archive/', views.PostAV.as_view(), name='post_archive'),

    # Example: /blog/archive/2019/
    path('archive/<int:year>/', views.PostYAV.as_view(), name='post_year_archive'),

    # Example: /blog/archive/2019/nov/
    path('archive/<int:year>/<str:month>/', views.PostMAV.as_view(), name='post_month_archive'),

    # Example: /blog/archive/2019/nov/10/
    path('archive/<int:year>/<str:month>/<int:day>/', views.PostDAV.as_view(), name='post_day_archive'),

    # Example: /blog/archive/today/
    path('archive/today/', views.PostTAV.as_view(), name='post_today_archive'),

]
------------------------------------
8. blog 폴더의 views.py
-------------------------------------
from django.views.generic import ListView, DetailView
from django.views.generic import ArchiveIndexView, YearArchiveView, MonthArchiveView
from django.views.generic import DayArchiveView, TodayArchiveView

from blog.models import Post


#--- ListView
class PostLV(ListView):
    model = Post
    template_name = 'blog/post_all.html'
    context_object_name = 'posts'
    paginate_by = 2


#--- DetailView
class PostDV(DetailView):
    model = Post


#--- ArchiveView
class PostAV(ArchiveIndexView):
    model = Post
    date_field = 'modify_dt'


class PostYAV(YearArchiveView):
    model = Post
    date_field = 'modify_dt'
    make_object_list = True


class PostMAV(MonthArchiveView):
    model = Post
    date_field = 'modify_dt'


class PostDAV(DayArchiveView):
    model = Post
    date_field = 'modify_dt'


class PostTAV(TodayArchiveView):
    model = Post
    date_field = 'modify_dt'
--------------------------------------
9. blog 폴더에서 templates 폴더 생성하고 그 안에 blog 폴더 생성후 post_all.html 생성
--------------------------------------
<h1>Blog List</h1>
<br>

{% for post in posts %}
    <h3><a href='{{ post.get_absolute_url }}'>{{ post.title }}</a></h3>
    {{ post.modify_dt|date:"N d, Y" }}
    <p>{{ post.description }}</p>
{% endfor %}

<br>

<div>
    <span>
        {% if page_obj.has_previous %}
            <a href="?page={{ page_obj.previous_page_number }}">PreviousPage</a>
        {% endif %}

        Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}

        {% if page_obj.has_next %}
            <a href="?page={{ page_obj.next_page_number }}">NextPage</a>
        {% endif %}
    </span>

</div>
--------------------------------------
10. 9번과 같은 폴더에서 post_detail.html 생성
--------------------------------------
<h2>{{ object.title }}</h2>

<p>
    {% if object.get_previous %}
    <a href="{{ object.get_previous.get_absolute_url }}" title="View previous post">
        &laquo;-- {{ object.get_previous }}
    </a>
    {% endif %}

    {% if object.get_next %}
    | <a href="{{ object.get_next.get_absolute_url }}" title="View next post">
    {{ object.get_next }} --&raquo;
    </a>
    {% endif %}
</p>

<p>{{ object.modify_dt|date:"j F Y" }}</p>
<br>

<div>
    {{ object.content|linebreaks }}
</div>


--------------------------------------
11. 9번과 동일하게 post_archive.html 생성
--------------------------------------
<h1>Post Archives until {% now "N d, Y" %}</h1>
<ul>
    {% for date in date_list %}
    <li style="display: inline;">
        <a href="{% url 'blog:post_year_archive' date|date:'Y' %}">Year-{{ date|date:"Y" }}</a></li>
    {% endfor %}
</ul>
<br/>

<div>
    <ul>
        {% for post in object_list %}
        <li>{{ post.modify_dt|date:"Y-m-d" }}&nbsp;&nbsp;&nbsp;
        <a href="{{ post.get_absolute_url }}"><strong>{{ post.title }}</strong></a></li>
        {% endfor %}
    </ul>
</div>

--------------------------------------
12. 9번과 동일하게 post_archive_month.html 생성
--------------------------------------
<h1>Post Archives for {{ month|date:"N, Y" }}</h1>

<div>
    <ul>
        {% for post in object_list %}
        <li>{{ post.modify_dt|date:"Y-m-d" }}&nbsp;&nbsp;&nbsp;
        <a href="{{ post.get_absolute_url }}"><strong>{{ post.title }}</strong></a></li>
        {% endfor %}
    </ul>
</div>

--------------------------------------
13. 9번과 동일하게 post_archive_day.html 생성
--------------------------------------
<h1>Post Archives for {{ day|date:"N d, Y" }}</h1>

<div>
    <ul>
        {% for post in object_list %}
        <li>{{ post.modify_dt|date:"Y-m-d" }}&nbsp;&nbsp;&nbsp;
        <a href="{{ post.get_absolute_url }}"><strong>{{ post.title }}</strong></a></li>
        {% endfor %}
    </ul>
</div>


--------------------------------------
14. 9번과 동일하게 post_archive_year.html 생성
--------------------------------------
<h1>Post Archives for {{ year|date:"Y" }}</h1>

<ul>
    {% for date in date_list %}
    <li style="display: inline;">
        <a href="{% url 'blog:post_month_archive' year|date:'Y' date|date:'b' %}">{{ date|date:"F" }}</a></li>
    {% endfor %}
</ul>
<br>

<div>
    <ul>
        {% for post in object_list %}
        <li>{{ post.modify_dt|date:"Y-m-d" }}&nbsp;&nbsp;&nbsp;
        <a href="{{ post.get_absolute_url }}"><strong>{{ post.title }}</strong></a></li>
        {% endfor %}
    </ul>
</div>


--------------------------------------
15. 서버 구동해보기 아마 북마크 리스트는 오류가 뜰거임 <터미널에서>
--------------------------------------
python manage.py runserver
--------------------------------------
16. bookmark 폴더의 templates의 북마크에서 bookmark_list.html에서 수정 bookmark: 추가
--------------------------------------

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Django Bookmark List</title>
</head>
<body>

<div id="content">
    <h1>Bookmark List</h1>
    <ul>
        {% for bookmark in object_list %}
            <li><a href="{% url 'bookmark:detail' bookmark.id %}">{{ bookmark }}</a></li>
        {% endfor %}
    </ul>
</div>

</body>
</html>
--------------------------------------

17. 부트스트랩 적용 시작 14page url 추가
--------------------------------------
#mysite/urls.py
from django.contrib import admin
from django.urls import path, include
from mysite.views import HomeView


urlpatterns = [
    path('admin/', admin.site.urls),
    # shkim
    path('', HomeView.as_view(), name='home'),
    path('bookmark/', include('bookmark.urls')),
    path('blog/', include('blog.urls')),
]

--------------------------------------
18. mysite 폴더 내에는 views.py가 없어 생성 및 코딩
--------------------------------------
from django.views.generic import TemplateView

#---TemplateView
class HomeView(TemplateView):
    template_name = 'home.html'
--------------------------------------

--------------------------------------
19. mysite/ Setting.py에서 수정 'DIRS': [os.path.join(BASE_DIR, 'templates')],
--------------------------------------
"""
Django settings for mysite project.

Generated by 'django-admin startproject' using Django 2.2.2.

For more information on this file, see
https://docs.djangoproject.com/en/2.2/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/2.2/ref/settings/
"""

import os

# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/2.2/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'qg0y+a@)c3%)(i*z#e6)7n30r%k^^m@hqssc$&iomf2$q0&l(y'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = ['192.168.56.101', 'localhost', '127.0.0.1']


# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # shkim
    'bookmark.apps.BookmarkConfig',
    'blog.apps.BlogConfig',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'mysite.urls'

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

WSGI_APPLICATION = 'mysite.wsgi.application'


# Database
# https://docs.djangoproject.com/en/2.2/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}


# Password validation
# https://docs.djangoproject.com/en/2.2/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/2.2/topics/i18n/

LANGUAGE_CODE = 'en-us'

# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/Seoul'

USE_I18N = True

USE_L10N = True

# USE_TZ = True
USE_TZ = False


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/2.2/howto/static-files/

STATIC_URL = '/static/'

# shkim

STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')


--------------------------------------
20. 가장 바깥에 는 mysite에서 templates 폴더 생성후 home.html 생성
--------------------------------------
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>{% block title %}Django Web Programming{% endblock %}</title>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">

    {% block extra-style %}{% endblock %}
</head>

<body style="padding-top:90px;">

<nav class="navbar navbar-expand-lg navbar-dark bg-primary fixed-top">
    <span class="navbar-brand mx-5 mb-0 font-weight-bold font-italic">Django - Python Web Programming</span>
    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent">
        <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav mr-auto">
            <li class="nav-item mx-1 btn btn-primary">
                <a class="nav-link text-white" href="{% url 'home' %}">Home</a></li>
            <li class="nav-item mx-1 btn btn-primary">
                <a class="nav-link text-white" href="{% url 'bookmark:index' %}">Bookmark</a></li>
            <li class="nav-item mx-1 btn btn-primary">
                <a class="nav-link text-white" href="{% url 'blog:index' %}">Blog</a></li>
            <li class="nav-item mx-1 btn btn-primary">
                <a class="nav-link text-white" href="">Photo</a></li>

            <li class="nav-item dropdown mx-1 btn btn-primary">
                <a class="nav-link dropdown-toggle text-white" href="#" data-toggle="dropdown">Util</a>
                <div class="dropdown-menu">
                    <a class="dropdown-item" href="{% url 'admin:index' %}">Admin</a>
                    <div class="dropdown-divider"></div>
                    <a class="dropdown-item" href="{% url 'blog:post_archive' %}">Archive</a>
                    <a class="dropdown-item" href="">Search</a>
                </div>
            </li>
        </ul>

        <form class="form-inline my-2" action="" method="post"> {% csrf_token %}
            <input class="form-control mr-sm-2" type="search" placeholder="global search" name="search_word">
        </form>
    </div>
</nav>


<div class="container">
    {% block content %}{% endblock %}
</div>


{% block footer %}{% endblock %}


<script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
<script src="https://kit.fontawesome.com/c998a172fe.js"></script>

{% block extra-script %}{% endblock %}

</body>
</html>
--------------------------------------
21.가장 바깥에 는 mysite에서 templates 폴더 생성후 base.html 생성
--------------------------------------
{% extends 'base.html' %}

{% load static %}

{% block title %}home.html{% endblock %}

{% block extra-style %}
<style type="text/css">
.home-image {
    background-image: url("{% static 'img/lion.jpg' %}");
    background-repeat: no-repeat;
    background-position: center;
    background-size: 100%;
    height: 500px;
    border-top: 10px solid #ccc;
    border-bottom: 10px solid #ccc;
    padding: 20px 0 0 20px;
}
.title {
    color: #c80;
    font-weight: bold;
}
.powered {
    position: relative;
    top: 77%;
    color: #cc0;
    font-style: italic;
}
</style>
{% endblock %}

{% block content %}
    <div class="home-image">
        <h2 class="title">Django - Python Web Programming</h2>
        <h4 class="powered"><i class="fas fa-arrow-circle-right"></i> powered by django and bootstrap.</h4>
    </div>

    <hr style="margin: 10px 0;">

    <div class="row text-center">
        <div class="col-sm-6">
            <h3>Bookmark App</h3>
            <p>Bookmark is a Uniform Resource Identifier (URI)
                that is stored for later retrieval in any of various storage formats.
                You can store your own bookmarks by Bookmark application.
                It's also possible to update or delete your bookmarks.</p>
        </div>
        <div class="col-sm-6">
            <h3>Blog App</h3>
            <p>This application makes it possible to log daily events or write your own interests
                such as hobbies, techniques, etc.
                A typical blog combines text, digital images, and links to other blogs, web pages,
                and other media related to its topic.</p>
        </div>
    </div>
{% endblock content %}

{% block footer %}
<footer class="fixed-bottom bg-info">
    <div class="text-white font-italic text-right mr-5">Copyright &copy; 2019 DjangoBook by shkim</div>
</footer>
{% endblock %}
--------------------------------------
22. settings.py에서 제일 하단에 STATIC_URL 부분을 수정
--------------------------------------
STATIC_URL = '/static/'

STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static'),
]

--------------------------------------
23. 메인 페이지에사진을  넣고 싶다면 
blog, bookmark,templates가 보이는 화면에서 static 폴더 생성, 그안에 img 폴더 생성하고 사진 넣기
--------------------------------------
#home.html에서 ctrl + f 로 lion 검색하여 넣은 사진파일명 넣기
--------------------------------------

