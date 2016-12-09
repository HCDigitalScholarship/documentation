## Creating a custom 404 page on a Django site
1. Create a template called `404.html` with whatever content you want, and place it in your default template directory.
2. Add the following lines to your `views.py` file (credits to [this StackOverflow post](http://stackoverflow.com/questions/17662928/django-creating-a-custom-500-404-error-page)):

```python
from django.shortcuts import render_to_response
from django.template import RequestContext

def handler404(request):
    response = render_to_response('404.html', {},
                                   context_instance=RequestContext(request))
    response.status_code = 404
    return response
```

### Troubleshooting
If you keep getting the default Django 404 page, make sure that your `404.html` is in the proper directory. If all of your templates live in the folder `app-name/templates/app-name`, then the 404 template should go in `app-name/templates` instead.

If you are in production and your whole site crashes, then check `views.py` for syntax errors.
