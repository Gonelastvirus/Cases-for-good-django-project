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
from django.shortcuts import redirect
def logged_in_test(user):
    if user.is_authenticated:
        return redirect('home_page')
    return False

admin.site.login = user_passes_test(logged_in_test)(admin.site.login)

```  
*The **user_passes_test** decorator is used to redirect logged in users away from the admin login page. The **logged_in_test** function checks if the user is logged in, and if so, redirects them to the **home_page** view by returning False.*

*__admin.site.login__ is the built-in view for the admin login page. By default, the **admin.site.login** view is accessible to any user, and by wrapping it with __user_passes_test__*

## If the user does not log out from the page before closing the browser. If the user is already logged in, then the system will use the existing session and username.
For example currently loggedin user is : John , he close the browser and visit the site later he should login again.
One solution to this problem would be to set the session to expire when the browser is closed. This way, the user would need to log in again when they reopen the browser. 
*settings.py*
``` python
SESSION_EXPIRE_AT_BROWSER_CLOSE = True
```

