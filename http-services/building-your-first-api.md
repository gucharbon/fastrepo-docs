---
description: Create an HTTP service in minutes
---

# Building your first API

## Create the project

First of all you need to create a new project using `fastrepo`:

```
python -m fastrepo --project-name "Demo FastAPI"
```

{% hint style="info" %}
Work in progres... Check the [roadmap](../roadmap.md).
{% endhint %}

## Add `fastapi` dependency

Use `poetry` to add dependencies:

```text
poetry add fastapi uvicorn
```

## Create your application

Create a file `src/demo_fastapi/app.py`.

{% code title="src/demo\_fastapi/app.py" %}
```python
"""Rest API built using python and fastapi."""
from fastapi import FastAPI


app = FastAPI(
    title="Demo FastAPI",
    description="A demonstration Rest API written using fastapi."
)
```
{% endcode %}

## Create a router for your application

Create a file `src/demo_fastapi/routes.py`.

{% code title="src/demo\_fastapi/routes.py" %}
```python
"""HTTP routes for the demo REST API."""
from fastapi import APIRouter


router = APIRouter()
```
{% endcode %}

## Integrate router within application

Let's include the router into the application:

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

## Write a basic route

Let's write a route that will accept `GET` requests on the `/demo` endpoint and return an empty JSON body with status `HTTP 202 Accepted` :

{% api-method method="get" host="/" path="demo" %}
{% api-method-summary %}
Get an empty response
{% endapi-method-summary %}

{% api-method-description %}
Return an empty response when successful. This route does not accept any parameter.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=202 %}
{% api-method-response-example-description %}
Empty response.
{% endapi-method-response-example-description %}

```

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

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

## Start your application

Now that there is a valid route, let's start the application:

```python
uvicorn demo_fastapi.app:app --reload
```

{% hint style="info" %}
By default Uvicorn uses simple changes detection strategy that compares python files modification times few times a second. If this approach doesn't work for your project \(eg. because of its complexity\), you can install Uvicorn with optional `watchgod` dependency to use filesystem events instead:

```text
poetry add --dev uvicorn --extras watchgodreload
```
{% endhint %}

Now that your application is started, you should 

## Make your first commit

When you perform a commit, several tests must pass in order for the commit to be accepted:

![](../.gitbook/assets/image%20%284%29.png)



