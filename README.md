![image](https://github.com/moeinbaghsheikhi/Django-Redis/assets/81008392/bad1cf15-8de0-45cd-a5c1-1dba27be39bc)# Django-Redis

برای ایجاد یک پروژه Django که به Redis متصل شود، مراحل زیر را دنبال کنید. این شامل نصب Python، ایجاد پروژه Django، نصب کتابخانه‌های مورد نیاز برای Redis و تنظیمات مرتبط می‌شود.

### مراحل ایجاد پروژه Django

#### 1. نصب Python

ابتدا باید Python را روی سیستم خود نصب کنید. می‌توانید دستورالعمل‌های قبلی را برای نصب Python بر روی سیستم خود دنبال کنید.

#### 2. ایجاد محیط مجازی

پس از نصب Python، یک محیط مجازی برای پروژه خود ایجاد کنید:

```sh
mkdir redis_django_project
cd redis_django_project
python -m venv env
source env/bin/activate  # on Windows use `env\Scripts\activate`
```

#### 3. نصب Django و Redis

کتابخانه‌های مورد نیاز را نصب کنید:

```sh
pip install django django-redis redis
```

#### 4. ایجاد پروژه و اپلیکیشن Django

یک پروژه و یک اپلیکیشن در Django ایجاد کنید:

```sh
django-admin startproject myproject
cd myproject
django-admin startapp myapp
```

#### 5. پیکربندی Redis در تنظیمات Django

فایل `settings.py` را باز کنید و تنظیمات کش را به آن اضافه کنید:

```python
# myproject/settings.py

INSTALLED_APPS = [
    ...
    'myapp',
    'django.contrib.sessions',  # Ensure sessions are enabled
]

CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Session Engine
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
```

#### 6. ایجاد نمای ساده برای کار با Redis

فایل `views.py` در اپلیکیشن `myapp` را به‌روز کنید تا شامل نماهایی برای تعامل با Redis باشد:

```python
# myapp/views.py

from django.shortcuts import HttpResponse
from django.core.cache import cache
import redis
import json

# اتصال به Redis با استفاده از redis-py
r = redis.Redis(host='localhost', port=6379, db=0)

def set_cache(request):
    cache.set('mykey', 'Hello, Redis!', timeout=60*15)
    return HttpResponse("Key set successfully")

def get_cache(request):
    value = cache.get('mykey')
    return HttpResponse(f"The value of 'mykey' is: {value}")

def set_json(request):
    data = {
        'name': 'John Doe',
        'age': 30,
        'email': 'john.doe@example.com'
    }
    r.set('user:1000', json.dumps(data))
    return HttpResponse("JSON data set successfully")

def get_json(request):
    json_data = r.get('user:1000')
    data = json.loads(json_data) if json_data else {}
    return HttpResponse(f"The user data is: {data}")
```

#### 7. تنظیم URLها

فایل `urls.py` در اپلیکیشن `myapp` را به‌روز کنید تا شامل مسیرهای جدید باشد:

```python
# myapp/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('set/', views.set_cache, name='set_cache'),
    path('get/', views.get_cache, name='get_cache'),
    path('setjson/', views.set_json, name='set_json'),
    path('getjson/', views.get_json, name='get_json'),
]
```

و فایل `urls.py` در پروژه اصلی (`myproject/urls.py`) را به‌روز کنید:

```python
# myproject/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

#### 8. اجرای سرور

سرور Django را اجرا کنید:

```sh
python manage.py migrate
python manage.py runserver
```

### تست API

می‌توانید از ابزارهایی مانند Postman یا مرورگر خود برای تست API استفاده کنید.

1. **تنظیم یک کلید**:
   - URL: `http://localhost:8000/set/`
   - روش: GET

2. **دریافت مقدار کلید**:
   - URL: `http://localhost:8000/get/`
   - روش: GET

3. **ذخیره داده JSON**:
   - URL: `http://localhost:8000/setjson/`
   - روش: GET

4. **بازیابی داده JSON**:
   - URL: `http://localhost:8000/getjson/`
   - روش: GET

### توضیحات کد

- `cache.set` و `cache.get`: این دستورات برای تنظیم و دریافت مقدار یک کلید با استفاده از کش Django که به Redis متصل است، استفاده می‌شوند.
- `redis.Redis`: این خط یک کلاینت Redis با استفاده از `redis-py` ایجاد می‌کند.
- `r.set` و `r.get`: این دستورات برای تنظیم و دریافت مقدار یک کلید به صورت مستقیم از Redis استفاده می‌شوند.
- `json.dumps` و `json.loads`: این توابع برای تبدیل داده‌های JSON به رشته و بالعکس استفاده می‌شوند.

این پروژه ساده نشان می‌دهد که چگونه می‌توانید از Redis در یک برنامه Django استفاده کنید. با این شروع ساده، می‌توانید پروژه‌های پیچیده‌تری ایجاد کنید که از Redis برای کش کردن داده‌ها، مدیریت نشست‌ها (sessions)، و ذخیره‌سازی داده‌های موقت استفاده می‌کنند.
