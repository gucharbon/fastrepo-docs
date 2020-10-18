---
description: Bring enjoyable logging in Python.
---

# Logging Standardization

## The problem

Before writing a more complex application, let's solve a problem that might become a headache in the future. You probably saw in your console that logs were not formatted in a single a way. Let's take this output for example:

![Output with different logging formats](../.gitbook/assets/image%20%2813%29.png)

{% hint style="danger" %}
As time goes your project will use more and more external dependencies and your application might produce **logs in various formats**.

This **might seem not important** at first glance, but in the future, when you will want to monitor your application running in production, and **collect logs in a centralized fashion**, you **won't be able to get anything more than raw text from your logs**, unless you write a **complex parser** that can **extract meaningful information** from all your log formats.
{% endhint %}

## The solution

This problem is not easy to fix, and as such, the code won't be discussed in details, you simply need to add a few files and copy content into them. 

Create a directory named `logging` into the package directory. In this directory create 4 files:

* `__init__.py`: 
* `logger.py`: 
* `settings.py`: 
* `handler.py`: 

{% tabs %}
{% tab title="logger.py" %}
{% code title="src/demo\_fastapi/logging/logger.py" %}
```python
"""This module expose two functions that are useful to setup your application logging.

Functions:

- setup_logger: Setup global logging using loguru.
  This function requires positional or keywork arguments.

- setup_logger_from_settings: Setup global logging using loguru.
  This function requires a single positional argument which must be
  a valid LoggingSettings instance.

"""
from __future__ import annotations
import logging
import sys
from pathlib import Path
from typing import Optional

import loguru
from loguru import logger

from .handler import InterceptHandler
from .settings import LoggingSettings


def setup_logger(
    level: str,
    format: str,
    filepath: Optional[Path] = None,
    rotation: Optional[str] = None,
    retention: Optional[str] = None,
) -> loguru.Logger:
    """Define the global logger to be used by your entire application.

    Arguments:
        level: the minimum log-level to log.
        format: the logformat to use.
        filepath: the path where to store the logfiles.
        rotation: when to rotate the logfile.
        retention: when to remove logfiles.

    Returns:
        the logger to be used by the service.

    References:
        [Loguru: Intercepting logging logs #247](https://github.com/Delgan/loguru/issues/247)
        [Gunicorn: generic logging options #1572](https://github.com/benoitc/gunicorn/issues/1572#issuecomment-638391953)
    """
    # Remove loguru default logger
    logger.remove()
    # Cath all existing loggers
    # The manager attribute from the RootLogger instance cannot be found mypy or any other linters
    # This is due to the attribute being created on the fly  on line 1826 of logging/__init__.py
    # We don't have any other choice than ignoring type check here
    LOGGERS = [logging.getLogger(name) for name in logging.root.manager.loggerDict]  # type: ignore
    # Add stdout logger
    logger.add(
        sys.stdout,
        enqueue=True,
        colorize=True,
        backtrace=True,
        level=level.upper(),
        format=format,
    )
    # Optionally add filepath logger
    if filepath:
        Path(filepath).parent.mkdir(parents=True, exist_ok=True)
        logger.add(
            str(filepath),
            rotation=rotation,
            retention=retention,
            enqueue=True,
            colorize=False,
            backtrace=True,
            level=level.upper(),
            format=format,
        )
    # Overwrite config of standard library root logger
    logging.basicConfig(handlers=[InterceptHandler()], level=0)
    # Overwrite handlers of all existing loggers from standard library logging
    for _logger in LOGGERS:
        _logger.handlers = [InterceptHandler()]
        _logger.propagate = False

    return logger


def setup_logger_from_settings(settings: LoggingSettings) -> loguru.Logger:
    """Define the global logger to be used by your entire application.

    Arguments:
        settings: the logging settings to apply.

    Returns:
        the logger instance.
    """
    return setup_logger(
        settings.level,
        settings.format,
        settings.filepath,
        settings.rotation,
        settings.retention,
    )

```
{% endcode %}
{% endtab %}

{% tab title="settings.py" %}
{% code title="src/demo\_fastapi/logging/settings.py" %}
```python
"""Custom logging settings for demo_application package.

Settings can be loaded from environment variable.
"""
from enum import Enum
from pathlib import Path
from typing import Optional

from pydantic import BaseSettings


class LoggingLevel(str, Enum):
    """
    Allowed log levels for the application
    """

    CRITICAL: str = "CRITICAL"
    ERROR: str = "ERROR"
    WARNING: str = "WARNING"
    INFO: str = "INFO"
    DEBUG: str = "DEBUG"


class LoggingSettings(BaseSettings):
    """Configure your application logging using a LoggingSettings instance.

    All arguments are optional.

    Arguments:
        level (str): the minimum log-level to log. (default: "DEBUG")
        format (str): the logformat to use. (default: "<green>{time:YYYY-MM-DD HH:mm:ss.SSS}</green> | <level>{level: <8}</level> | <cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> | <level>{message}</level>")
        filepath (Path): the path where to store the logfiles. (default: None)
        rotation (str): when to rotate the logfile. (default: "1 days")
        retention (str): when to remove logfiles. (default: "1 months")
    """

    level: LoggingLevel = LoggingLevel.DEBUG
    format: str = (
        "<green>{time:YYYY-MM-DD HH:mm:ss.SSS}</green> | "
        "<level>{level: <8}</level> | "
        "<cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> | "
        "<level>{message}</level>"
    )
    filepath: Optional[Path] = None
    rotation: str = "1 days"
    retention: str = "1 months"

    class Config:
        env_prefix = "logging_"

```
{% endcode %}
{% endtab %}

{% tab title="handler.py" %}
{% code title="src/demo\_fastapi/logging/handler.py" %}
```python
"""Custom logging intercept handler to forward all log records to loguru."""
import logging
from logging import LogRecord
from loguru import logger


class InterceptHandler(logging.Handler):
    """A custom class to intercept logs emitted using the standard library."""

    def emit(self, record: LogRecord) -> None:
        """Emit any log intercepted using loguru."""
        # Get corresponding Loguru level if it exists
        try:
            level = logger.level(record.levelname).name
        except ValueError:
            level = logging.getLevelName(record.levelno)

        # Find caller from where originated the logged message
        frame, depth = logging.currentframe(), 2
        while frame.f_code.co_filename == logging.__file__:
            if frame.f_back:
                frame = frame.f_back
                depth += 1

        logger.opt(depth=depth, exception=record.exc_info).log(
            level, record.getMessage()
        )

```
{% endcode %}
{% endtab %}

{% tab title="\_\_init\_\_.py" %}
{% code title="src/demo\_fastapi/logging/\_\_init\_\_.py" %}
```python
"""Logging module of demo_fastapi package."""
from .logger import setup_logger, setup_logger_from_settings
from .settings import LoggingLevel, LoggingSettings

```
{% endcode %}
{% endtab %}
{% endtabs %}

Apply logging to your application:

{% code title="src/demo\_fastapi/app.py" %}
```python
"""Rest API written using FastAPI."""
from fastapi import FastAPI

from .logging import setup_logger_from_settings, LoggingSettings
from .routes import router

# Fetch settings from environment variable
logging_settings = LoggingSettings()
# Apply global settings
setup_logger_from_settings(logging_settings)


app = FastAPI(
    title="Demo FastAPI", description="A demo application to show FastAPI features."
)


app.include_router(router)

```
{% endcode %}

Start your application:

![Standardized logs in console](../.gitbook/assets/image%20%289%29.png)

Much better isn't it? 🚀 

{% hint style="danger" %}
It is not possible to format the two first uvicorn logs using this pattern because uvicorn is started before our code is imported.
{% endhint %}

