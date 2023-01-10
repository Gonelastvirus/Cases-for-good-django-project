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

## If the user does not log out from the page before closing the browser. If the user is already logged in, then the system will use the existing session and username.
For example currently loggedin user is : John , he close the browser and visit the site later he should login again.
One solution to this problem would be to set the session to expire when the browser is closed. This way, the user would need to log in again when they reopen the browser. 
**settings.py**

``` python
SESSION_EXPIRE_AT_BROWSER_CLOSE = True
```
Should be written on top above Debug []

**Output**

![ezgif-2-eb7e72c021](https://user-images.githubusercontent.com/67478827/211652104-d7382785-dc84-481c-b944-16c75e34579f.gif)


