---
description: Tips to make testing your API easier
---

# Testing your API

## Install test dependencies

In order to run the tests, we need to install two dependencies:

* [`pytest-asyncio`](https://pypi.org/project/pytest-asyncio/)\`\`
* [`http`x](https://www.python-httpx.org/)

```text
poetry add --dev pytest-asyncio httpx
```

## Configure tests

Before writing tests, let's create a `conftest.py` file that will define some `fixtures`.

We will define three fixtures:

* A fixture named `caplog` that returns a `LogCaptureFixture` to enable checking emitted logs during tests.
* A fixture named `application` that returns an instance of our application.
* A fixture named `client` that returns an `AsyncClient` we can use to test the appliation

{% code title="tests/conftest.py" %}
```python
""" A dummy conftest to illustrate pytest configuration """
import logging
from logging import LogRecord
from typing import Generator, AsyncGenerator

import pytest
from _pytest.logging import caplog as _caplog  # noqa: F401
from _pytest.logging import LogCaptureFixture
from loguru import logger
from fastapi import FastAPI
from httpx import AsyncClient

from demo_fastapi.app import app


@pytest.fixture
def caplog(
    _caplog: LogCaptureFixture,  # noqa: F811
) -> Generator[LogCaptureFixture, None, None]:
    """Access and control log capturing.

    Captured logs are available through the following properties/methods::

    - caplog.messages        -> list of format-interpolated log messages
    - caplog.text            -> string containing formatted log output
    - caplog.records         -> list of logging.LogRecord instances
    - caplog.record_tuples   -> list of (logger_name, level, message) tuples
    - caplog.clear()         -> clear captured records and formatted log output string

    The code is taken from loguru documentation. Type annotations were added to play nice with mypy.
    See https://loguru.readthedocs.io/en/stable/resources/migration.html#making-things-work-with-pytest-and-caplog
    """

    class PropogateHandler(logging.Handler):
        def emit(self, record: LogRecord) -> None:
            logging.getLogger(record.name).handle(record)

    handler_id = logger.add(PropogateHandler(), format="{message}")
    yield _caplog
    logger.remove(handler_id)


@pytest.fixture
def application() -> FastAPI:
    """Return the demo_fastapi application."""
    return app


@pytest.mark.asyncio
@pytest.fixture
async def client(application: FastAPI) -> AsyncGenerator[AsyncClient, None]:
    """Yield an asynchronous client to test the demo_fastapi application.

    Examples:

    - Make a GET request
        >>> get_response = await client.get('/something')
    - Make a POST request
        >>> post_response = await client.post('/something', data={'key': 'value'})
    ```
    """
    # Need to start the event manually here
    for hook in application.router.on_startup:
        await hook()
    async with AsyncClient(app=application, base_url="http://test") as client:
        yield client
    # Need to run shutdown events manually
    for hook in application.router.on_shutdown:
        await hook()

```
{% endcode %}

{% hint style="info" %}
As stated in comment, the `caplog` fixture is taken from [loguru documentation](https://loguru.readthedocs.io/en/stable/resources/migration.html#making-things-work-with-pytest-and-caplog).
{% endhint %}

{% hint style="warning" %}
Don't forget to add `@pytest.mark.asyncio` to any async fixture or test else `pytest` will fail.
{% endhint %}

Those fixtures can now be used anywhere in your tests, without importing them.

## Testing events

Let's write a first test that ensure event hooks work properly:

{% code title="tests/test\_events.py" %}
```python
"""Test lifecycle events (startup, shutdown)"""
from fastapi import FastAPI
from _pytest.logging import LogCaptureFixture
from fastapi.testclient import TestClient


def test_startup_event(application: FastAPI, caplog: LogCaptureFixture) -> None:
    """Test that:
        - some log is indeed printed to stdout when we start the application.
        - some log is indeed printed to stdout when we stop the application.

    In the future this test should be updated when new event hooks are added to the application.
    """
    with TestClient(application):
        pass
    captured = caplog.text
    assert "Starting router!" in captured
    captured = caplog.text
    assert "Stopping router!" in captured

```
{% endcode %}

{% hint style="info" %}
The `test_startup_event` declares an argument named `application`. Because **we defined a fixture** named application in `tests/conftest.py`, the **returned value of the fixture** will **automatically be injected** as the **argument value** of the function when running the test.
{% endhint %}

## Testing the route

{% code title="tests/test\_router.py" %}
```python
"""Test router of demo_fastapi application."""
import pytest
from httpx import AsyncClient
from _pytest.logging import LogCaptureFixture


@pytest.mark.asyncio
async def test_root(client: AsyncClient) -> None:
    """Test that the root endpoint returns an empty response with status code 202."""
    response = await client.get("/")
    # Status code must be 202 Accepted
    assert response.status_code == 202
    # Response body must be empty
    assert response.text == ""

```
{% endcode %}

{% hint style="info" %}
In this test, we use the `client` fixture to perform a `GET` request on the application.
{% endhint %}



