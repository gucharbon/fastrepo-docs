---
description: Create an HTTP service in minutes
---

# Building your first API

## The foundations

{% hint style="warning" %}
This tutorial assumes that you are already familiar with the HTTP protocol. A good resource to learn about HTTP is the [Overview of HTTP by the Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview).
{% endhint %}

In this section you will learn how to:

* Create a [`FastAPI` ](https://fastapi.tiangolo.com/features/)application
* Create an [`APIRouter`](https://fastapi.tiangolo.com/tutorial/bigger-applications/#import-apirouter)\`\`
* Define a route using the `APIRouter`
* Start your application using [`uvicorn`](https://www.uvicorn.org/)\`\`

### Create the project

Let's create a new project using `fastrepo`:

```
python -m fastrepo --project-name "Demo FastAPI"
```

{% hint style="info" %}
If you don't have `fastrepo` installed, read the[ F.A.Q](../#how-to-install-fastrepo).
{% endhint %}

### Install dependencies

Use `poetry` to add two dependencies:

* fastapi
* uvicorn

```text
poetry add fastapi uvicorn
```

### Create your application

Create a file named `app.py` into the python package directory. This file will define your application:

{% code title="src/demo\_fastapi/app.py" %}
```python
"""Demo FastAPI is a simple REST API built using FastAPI."""
from fastapi import FastAPI


app = FastAPI(
    title="Demo FastAPI",
    description="A simple Rest API written built using FastAPI."
)
```
{% endcode %}

{% hint style="info" %}
Reading [the source code for the FastAPI class](https://github.com/tiangolo/fastapi/blob/master/fastapi/applications.py#L29) is the best way know all options that can be used when creating an application.
{% endhint %}

### Start your application

You can already start your application using the command:

```python
uvicorn demo_fastapi.app:app --reload
```

{% hint style="info" %}
By default, `uvicorn` starts your application using port `8000` and listens to `localhost`. You can change those settings using the `--port` and `--host` option respectively.
{% endhint %}

You should see logs in your console indicating that your application is starting:

![Uvicorn running and serving FastAPI application](../.gitbook/assets/image%20%287%29.png)

{% hint style="success" %}
Python displays debugging information related to asyncio such as this line:

```text
Executing <Task pending name='Task-1' coro=<Server.serve() running at c:\users\gcharbon\workspace\books\demo_fastapi\.venv\lib\site-packages\uvicorn\main.py:424> wait_for=<Future pending cb=[<TaskWakeupMethWrapper object at 0x000001A0CBBAFB80>()] created at C:\Users\gcharbon\AppData\Local\Programs\Python\Python38\lib\asyncio\base_events.py:422> cb=[_run_until_complete_cb() at C:\Users\gcharbon\AppData\Local\Programs\Python\Python38\lib\asyncio\base_events.py:184] created at C:\Users\gcharbon\AppData\Local\Programs\Python\Python38\lib\asyncio\base_events.py:595> took 0.375 seconds
```

when the environment variable `"PYTHONASYNCIODEBUG"` is set to `1`. You should always set this environment variable when developing.

Check the [official documentation](https://docs.python.org/3.8/library/asyncio-dev.html) to learn more about asyncio debug mode.
{% endhint %}

If you access `http://localhost:8000/docs` you should see this page:

![Default Swagger OpenAPI Documentation](../.gitbook/assets/image%20%286%29.png)

Now that the application is created, let's implement some routes.

### Create a router for your application

Create a file `src/demo_fastapi/routes.py`. This file will contains an [`APIRouter`](https://fastapi.tiangolo.com/tutorial/bigger-applications/#import-apirouter), as well as all routes associated with this router.

{% code title="src/demo\_fastapi/routes.py" %}
```python
"""HTTP routes for the demo REST API."""
from fastapi import APIRouter


router = APIRouter()
```
{% endcode %}



{% tabs %}
{% tab title="Notes" %}
{% hint style="info" %}
A `FastAPI` application **always includes a default router**. If you follow the [Beginner Tutorial](https://fastapi.tiangolo.com/tutorial/first-steps/) you will see that there isn't any mention of `APIRouter`. It is only mentionned in the [Bigger Applications - Multiple file section](https://fastapi.tiangolo.com/tutorial/bigger-applications/). This means that **creating a router is optionnal**, but **it should still be used for better separation of concern**.

If you're not in a hurry, check the details tab to learn more.
{% endhint %}
{% endtab %}

{% tab title="Details" %}

{% endtab %}
{% endtabs %}

### Define router lifecycle events

Each router can have its own lifecycle, I.E, `startup` and `shutdown` events:

{% code title="src/demo\_fastapi/routes.py" %}
```python
"""HTTP routes for the demo REST API."""
from fastapi import APIRouter


router = APIRouter()


@router.on_event("startup")
async def on_startup():
    """"This function is called once after application initialization.
    
    If any error is raised in this function, the application will never start.
    """
    logger.debug("Starting router!")


@router.on_event("shutdown")
async def on_shutdown():
    """This function is called once before application is stopped.
    
    If any error is raised in this function, application will exit immediately.
    """
    logger.debug("Stopping router!")
```
{% endcode %}

In practice, **events hooks** provide a simple way to **connect clients before the application starts** and **close connections on shutdown** in order to gracefully stop the application.

### Integrate router within application

Now that there is a router, let's include it into the application:

{% code title="src/demo\_fastapi/app.py" %}
```python
"""Rest API built using python and fastapi."""
from fastapi import FastAPI

from .routes import router as my_custom_router


app = FastAPI(
    title="Demo FastAPI",
    description="A demonstration Rest API written using fastapi."
)

app.include_router(my_custom_router)
```
{% endcode %}

### Write a basic route

Let's write a route that will accept `GET` requests on the `/demo` endpoint and return an empty JSON body with status `HTTP 202 Accepted` 

{% code title="src/demo\_fastapi/routes.py" %}
```python
"""HTTP routes for the demo REST API."""
from fastapi import APIRouter, Response


router = APIRouter()


@router.get("/demo", summary="Get an empty response.")
def demo_response():
    """Return an empty response when successful. This route does not accept any parameter."""
    return Response(status_code=202)
```
{% endcode %}

### Visit the Swagger UI

The Open API documentation should be served on your localhost on [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

You should see this page:

![Swagger OpenAPI Documentation](../.gitbook/assets/image%20%285%29.png)



### Make your first commit

When you perform a commit, several tests must pass in order for the commit to be accepted:

![Console after successfull commit](../.gitbook/assets/image%20%284%29.png)





