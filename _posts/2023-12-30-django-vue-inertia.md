---
layout: post
title: How to integrate Django with Vue without REST API
subtitle: How to get Django working with Vue via Inertia.js
# cover-img: /assets/img/path.jpg
# thumbnail-img: /assets/img/thumb.png
# share-img: /assets/img/path.jpg
# gh-repo: daattali/beautiful-jekyll 
# gh-badge: [star, fork, follow]
# comments: true
tags: [programming, web development]
---

Django, the web framework for perfectionists with deadlines. Based on Django's official site, Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of web development, so you can focus on writing your app without needing to reinvent the wheel. It’s free and open source. In the other hand, Vue.js based on the official site is an approachable, performant and versatile framework for building web user interfaces. But how to work with these two if you only want to make a monolith SaaS without API? And there's come Inertia.js which is a new approach to building classic server-driven web apps. Based on the Inerita.js's official docs, Inertia allows you to create fully client-side rendered, single-page apps, without the complexity that comes with modern SPAs. It does this by leveraging existing server-side patterns that you already love.
## Prequisites
I recommend to you guys to have a virtual environment for each of projects you made because using virtual environments is considered a best practice as it prevents package version conflicts between projects that require different module versions. For example, Project A may require Selenium 3.x while Project B relies on Selenium 4.x. With virtual environments, you can install Selenium 3.x for Project A and Selenium 4.x for Project B without compatibility issues.

So, let's begin! I work with Linux/GNU Debian environment here so I'm just giving how you do on UNIX based operating systems. First, just create a directory name for the project such as `my_project` and `cd` to that directory. Create a Python virtual environment with `python -m venv <venv_name>`, in this example let's just call it `.venv` since it's a standard convention. Source the virtual environment by executing `source .venv/bin/activate`. Install the required packages for creating the project. Install the packages one by one:
```
pip install django
pip install inertia-django
pip install django-vite
pip install django-js-routes
```
And simply create the project by executing `django-admin startproject my_project .` the dot tells Django to create the project in the current directory and not in a sub-directory. Let's head up to the back-end side.
## Back-end
We have to configure the back-end first.
### Configurations
Once you are already in the project, you could configure the `settings.py`  inside the `my_project` sub-directory. In this file, add the inertia-django, django-vite and django-js-routes.
```python
INSTALLED_APPS = [
	...
    "django.contrib.staticfiles",
    "django_vite",
    "inertia",
    "js_routes",
]
```
Also add the Inertia's middleware:
```python
MIDDLEWARE = [
	...
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
    'inertia.middleware.InertiaMiddleware',
]
```
You could change the database you want to use in the `DATABASES` variable. See [Django Database Configurations](https://docs.djangoproject.com/en/4.2/ref/settings/#databases), but in this tutorial I prefer to use the default database, sqlite3.
In the `TEMPLATES` variable, add a value `"templates"` in the `"DIRS"` key of the dictionary.
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': ["templates"], # <- Add this
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
Copy this lines below the `STATIC_URL` variable.
```python
STATIC_ROOT = BASE_DIR / "static"
STATICFILES_DIRS = [
    BASE_DIR / "web" / "dist"
]
```
Copy the Vite's configuration below the `DEFAULT_AUTO_FIELD` variable.
```python
DJANGO_VITE_DEV_MODE = True
DJANGO_VITE_ASSETS_PATH = BASE_DIR / "web" / "dist"
# This tells Django to load the manifest from the front end build folder
# Comment it out if you run python manage.py collectstatic
DJANGO_VITE_MANIFEST_PATH = DJANGO_VITE_ASSETS_PATH / "manifest.json"
DJANGO_VITE_DEV_SERVER_PORT = 5173
STATICFILES_DIRS.append(DJANGO_VITE_ASSETS_PATH)
```
Copy the Inertia's configuration below the Vite's configurations.
```python
CSRF_HEADER_NAME = 'HTTP_X_XSRF_TOKEN'
CSRF_COOKIE_NAME = 'XSRF-TOKEN'
INERTIA_LAYOUT = "base.html"
INERTIA_VERSION = "1.0"
INERTIA_SSR_ENABLED = False
INERTIA_SSR_URL = "http://localhost:13714"
```
Copy the configurations for [django-js-routes](https://pypi.org/project/django-js-routes/). 
```python
JS_ROUTES_INCLUSION_LIST = [
    "example"
]
```
### Create An App
We gonna make an example of an app by using `python manage.py startapp <example_app>`.
````
.
├── db.sqlite3
├── example_app
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── LICENSE
├── manage.py
├── README.md
├── templates
└── my_project
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
````
Add the application that we made in to the `INSTALLED_APPS` in `settings.py`:
```python
INSTALLED_APPS = [
    ...
    "inertia",
    "js_routes",
    "example_app",  # Our example_app
]
```
Now, we're all set in the back-end.
## Front-end
Let's get to the front-end.
### Create The Base Template
We have to create the base template for Django to load the JavaScript components. Create `templates/` folder in the root directory of the project, and make a `_base.html` file inside the template folder.
![base.html](https://i.imgur.com/NOwQEc8.png)
### Create The Vite App
In the root project directory, run `npm create vite@latest`, in this example the project name would be called "web" because we already declared it in the `STATICFILE_DIRS` variable in `settings.py`.
![npm create vite@latest](https://i.imgur.com/0SC6uPT.png)
### Configure Entry Point
Okay,  then go to the project directory, so `cd web` and then run `npm i @inertiajs/vue3`. Head up to `src/main.js` and then replace the script inside with:
```vue
import { createApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/vue3'

createInertiaApp({
  resolve: name => {
    const pages = import.meta.glob('./Pages/**/*.vue', { eager: true })
    return pages[`./Pages/${name}.vue`]
  },
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
})
```
### Configure Vite's Config
And replace the script inside the `vite.config.js` with this:
```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

// https://vitejs.dev/config/
export default defineConfig({
  base: "/static/",
  plugins: [vue()],
  build: {
    outDir: resolve('./dist'),
    manifest: true,
    rollupOptions: {
      input: './src/main.js'
    }
  },
  server: {
    origin: 'http://localhost:5173',
    cors: {
      allowedHeaders: '*'
    }
  }
})
```
### Create A Page
The configurations are all done, let's make some pages inside `Pages/` folder, create the `Pages/` folder first inside the `src/` directory: `mkdir src/Pages/` and then create let's say `App.vue` inside the `Pages/` like this:
```vue
<template>
  <div class="app">
    <h1>Welcome!</h1>
    <p>Welcome to Django + Vue application.</p>
  </div>
</template>

<script setup>
</script>

<style>
.app {
  text-align: center;
  margin-top: 20px;
}
</style>
```
Okay, the front-end all are set. Let's make the route!
## Integrate Django With Inertia
### Configure The App's View
Head up to `example_app/views.py` and make a logic to render the pages inside `Pages/` in the front-end. We could do it like this:
```py
# example_app/views.py
from inertia import render


def app_view(request):
    return render(request, "App", props={"inertia": True})
```
### Configure The App's Route
Create `urls.py` inside `example_app/` and paste this:
```py
# example_app/urls.py
from django.urls import path
from .views import app_view

urlpatterns = [
    path("", app_view, name="app"),
]
```
### Configure The Main Route
Now head up to `urls.py` inside the `my_project` directory and copy this:
```py
# my_project/urls.py
from django.conf.urls.static import static
from django.conf import settings
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include("example_app.urls"))
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```
## Run The Server
Run `python manage.py makemigrations` and `python manage.py migrate` then, create two terminals, one of the terminal runs `python manage.py runserver` and the other runs `npm run dev` inside the `web/` directory.