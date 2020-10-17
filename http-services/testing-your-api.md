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

## Create a fixture



