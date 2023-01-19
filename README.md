# Cases-for-good-django-project
## Loggedin user shouldn't visit admin page
The reason is we use  auth_user table for all user including super user there will be problem if loggedin user get admin page and log in.
On 
```python 
request.user.id 
```
you will get that superuser id not currently loggedin user id.

Here is an example of how you can use the **user_passes_test** decorator to redirect logged in users away from the admin login page in your **admin.py** file:
```python
from django.contrib.auth.decorators import user_passes_test
def admin_only(user):
# check if the user is a staff member or superuser
    if not user.is_staff and not user.is_superuser:
        return False
    return True
# Apply this decorator to the login view
admin.site.login = user_passes_test(admin_only, login_url='permission_denied')(admin.site.login)
```  
*The **user_passes_test** decorator is used to redirect logged in users away from the admin login page. The **logged_in_test** function checks if the user is logged in, and if so, redirects them to the **permission_denied** view by returning False.*

*__admin.site.login__ is the built-in view for the admin login page. By default, the **admin.site.login** view is accessible to any user, and by wrapping it with __user_passes_test__*

**if you want logout user visit admin page**

This will return True if user is anonymous or if user is a staff member or superuser.
It will check for anonymous first and then for staff and superuser.
```python
def admin_only(user):
    return user.is_anonymous or (user.is_staff or user.is_superuser)
```
**urls.py**
```python
from django.urls import path
urlpatterns = [
 path("permission_denied/",views.permission_denied,name="permission_denied"),
 ]
```
**views.py**
```python
from django.shortcuts import render
def permission_denied(request):
    return render(request,"permission_denied.html")
```
**permission_denied.html**
```html
<html>
    <head>  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.0.0/dist/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
    </head>
    <title>403 No permission</title>
    <body>
        <img src="{%static 'iotapp/403.gif' %}" class="img-fluid" alt="Responsive image">
    </body>
    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/popper.js@1.12.9/dist/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@4.0.0/dist/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
```
**Output**

![ezgif-2-c64a113201](https://user-images.githubusercontent.com/67478827/211650514-7fa5c6ed-308a-4efd-9572-70a73c41e578.gif)

## IF the user does not log out from the page before closing the browser. If the user is already logged in, then the system will use the existing session and username.
For example currently loggedin user is : John , he close the browser and visit the site later he should login again.
One solution to this problem would be to set the session to expire when the browser is closed. This way, the user would need to log in again when they reopen the browser. 

**settings.py**
``` python
SESSION_EXPIRE_AT_BROWSER_CLOSE = True
```
Should be written on top above Debug [ ]

**Output**

![ezgif-2-eb7e72c021](https://user-images.githubusercontent.com/67478827/211652104-d7382785-dc84-481c-b944-16c75e34579f.gif)

## Get Nepal timezone in django. Get timestamp

get into your virtualenv and use pip to install pytz
```
pip install pytz
```
you need to use django default datetime module.
```python
from pytz import timezone
import datetime
time_zone = timezone('Asia/Kathmandu')
timestamp = time_zone.localize(datetime.datetime.now())
print(timestamp)
```
**Output**
```
2023-01-18 13:52:18.036763+05:45
```
* **Note:** if you are working on django & try to store this timezone then it will automatically store in utc format.
* You can get utc time of kathmandu nepal: https://24timezones.com/difference/utc/kathmandu

**My personal django channel consumer.py code:**
```python
from pytz import timezone
import datetime
class DashConsumer(AsyncJsonWebsocketConsumer):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.time_zone = timezone('Asia/Kathmandu')
    async def connect(self):
        #.......................
    async def disconnect(self,code_close):
        #.....................
    async def receive():
        #.................
        timestamp = self.time_zone.localize(datetime.datetime.now())
        date = timestamp.date()  # get the date from the timestamp
        
```
# Set Templates in setting.py to access from anywhere
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, "templates"),],
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
# Middleware.py for django channel to authenticate token in database and token getting receive from edge device 
```python
import urllib.parse
from django.contrib.auth import get_user_model
import urllib.parse
from channels.db import database_sync_to_async
@database_sync_to_async
def get_user(token):
    User = get_user_model()
    try:
        return User.objects.get(token=token)
    except User.DoesNotExist:
        return None

class CustomUserAuthMiddleware:
    def __init__(self, inner):
        self.inner = inner

    async def __call__(self, scope,receive,send):
        # Get the token from the query string
        token = urllib.parse.parse_qs(scope["query_string"])[b'token'][0].decode('utf-8')
        # Get the user using the token
        user = await get_user(token)
        if user is None:
            # If no user was found, return a 401 Unauthorized response
            return {
                "close": True,
                "headers": [
                    (b"status", b"401 Unauthorized"),
                ],
            }
        # If a user was found, set the user on the scope so that it can be accessed
        # by the inner application
        scope["user"] = user
        return await self.inner(scope,receive,send)
   ```
 **You need to add this middleware in asgi.py before authmiddleware stack so that this custom middliware runs first.**
 ```python
 import os

from django.core.asgi import get_asgi_application
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'iotag.settings')

from channels.routing import ProtocolTypeRouter,URLRouter
from channels.auth import AuthMiddlewareStack
from iotag.middlewaretokenauth import CustomUserAuthMiddleware
from django.urls import path
from iotapp import consumer
websocket_urlPattern=[
    path('show',consumer.DashConsumer.as_asgi()),
]
application=ProtocolTypeRouter({
    # 'http':
    'websocket':CustomUserAuthMiddleware(AuthMiddlewareStack(URLRouter(websocket_urlPattern)))

})
 
 ```
# Extend auth_user table in django.

Sometimes you need some extra field in auth_user table. In such case you can do following:

**models.py**

```python
from django.contrib.auth.models import AbstractUser
class CustomUser(AbstractUser):
    token=models.CharField(max_length=17)
```
**settings.py**
```python
AUTH_USER_MODEL = 'your_app_name.CustomUser'
```
If you are going to create some tables in your project for example table:**SensorData**. As you can use a custom user model that you have defined, rather than being limited to the default Django user model.

**Add this on your models.py so your models.py becomes:** 

```python
from django.db import models
from django.conf import settings
from django.contrib.auth.models import AbstractUser
class CustomUser(AbstractUser):
    token=models.CharField(max_length=17)
class SensorData(models.Model):
    user= models.ForeignKey(settings.AUTH_USER_MODEL,on_delete=models.CASCADE)
    sensor_value= models.FloatField()
    timestamp = models.DateTimeField()
```
The SensorData model has a foreign key field called user which references a user model. Instead of hardcoding the user model class name in the foreign key field, the **settings.AUTH_USER_MODEL** is used. Now your **SensorData** table contain **user_id** field with specific user id from **CustomUser table**

**CustomUser table:**

![Screenshot from 2023-01-18 21-38-41](https://user-images.githubusercontent.com/67478827/213224007-ee9e9818-5e27-49d1-b61d-8d81860c92ef.png)

**SensorData Table:**

![Screenshot from 2023-01-18 21-41-10](https://user-images.githubusercontent.com/67478827/213223526-f2986f56-059d-480e-b185-3eabf380ddba.png)



