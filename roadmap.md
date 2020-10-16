# Roadmap

## Milestone 0

### Project generation

* [ ] Default template available
  * [x] Dependencies management is done using Poetry:
    * [x] `poetry add [--dev] [--extra <EXTRA>] <PACKAGE>`
  * [ ] Package management is done using invoke and four tasks are available:
    * [ ] `inv install`
    * [ ] `inv update`
    * [x] `inv build`
    * [x] `inv publish [--repository <REPOSITORY>]`
  * [x]  Code style is enforced by flake8
    * [x] The `setup.cfg`file configures flake8
    * [x] `inv flake8` can be used to lint code.
  * [x] Code is formatted using black
    * [x] `inv format` can be used to format code.
  * [x] Module imports are sorted using isort
    * [x] `inv format` can be used to format imports.
  * [ ] Type checking is performed using mypy
    * [ ] `inv typecheck`
  * [x] Tests are ran using pytest
    * [x] `inv test`
  * [x] Documentation is built using mkdocs
    * [x] `inv docs`
  * [x] When building package, docs is also built by default. It can optionnaly be disabled:
    * [x] `inv build --no-docs`
  * [x] When building package, tests are also executed by default. It can optionnaly be disabled:
    * [x] `inv build --no-coverage`

### Documentation

* [ ] HTTP Services Documentation
  * [ ] Building your first API
  * [ ] Testing your API
  * [ ] Logging Standardization
  * [ ] Distributed Tracing
  * [ ] Using S3 storage
  * [ ] Using PostgreSQL Database
  * [ ] Publishing to NATS
  * [ ] Websockets & NATS subscriptions

## Milestone 1

### Project Generation

* [ ] HTTP Service generator
  * [ ] FastAPI Integration
  * [ ] Built-in CLI. Start your application using the command line:
    * [ ] `my-service start --port 8000 --host 0.0.0.0`
  * [ ] FastSTAN Integration
    * [ ] Inject NATS client in the application:

      ```python
      from fastnfurious import FastNFuriousAPI
      from fastapi import HTTPException


      from .my_custom_models import SomethingResult


      app = FastNFuriousAPI(title="Demo application")


      @app.get(
          "/something/{resource_id}",
          summary="Get something from a resource",
          # If you know in which format your service will reply, you don't need to worry about JSON serialization
          response_model=SomethingResult
      )
      async def fetch_something_associated_with_resource(resource_id: int):
          """In this endpoint with access the NATS client as an attribute if the application."""
          res = await app.nats.request_json(f"something", {"resource_id": resource_id})
          if res.status:
              # Simply return the value as it will be automatically serialized
              return res.value
          else:
              # This will return an HTTP response with status code 500 Internal Server Error and error message as response body.
              raise HTTPException(500, res.error, media_type="text/plain")
      ```

      NATS Client configuration should be parsed from environment variables.

    * [ ] Let use create other clients with same configuration when he wants to:

      ```python
      from fastnfurious import FastNFuriousAPI
      from fastapi import WebSocket
      from pydantic import BaseModel


      class NewMessage(BaseModel):
          author: str
          content: str
          timestamp: int


      app = FastNFuriousAPI(title="Websocket demo")


      @app.websocket("/ws")
      async def realtime_susbcription(websocket: Websocket):
          """Accept websocket connections, subscribe to NATS and send subscription events in real time."""
          async with app.create_nats_client() as client:
              # You will accept the websocket connection only if NATS can be reached
              await websocket.accept()
              # Define your subscription with your newly created client
              @client.subscribe("messages", start=True)
              async def on_message(msg: NewMessage):
                  # For each received message, send it through the websocket
                  await websocket.send_json(msg)
      ```
  * [ ] Minio/S3 Integration
    * [ ] Inject MinIO client in the application:

      ```python
      from fastnfurious import FastNFuriousAPI
      from fastapi import Response

      MODELS_BUCKET = "models"

      app = FastNFuriousAPI(title="Demo MinIO usage")

      @app.get("/models/{model_id}", summary="Retrieve a model as bytes")
      async def fetch_model(model_id: int, version: str = "latest"):
          """In this endpoint we access MinIO client as an attribute of the application."""
          filename = f"{model_id}.onnx"
          model = await app.s3[MODELS_BUCKET].get(filename)
         return Response(model, media_type="application/octet-stream")
      ```
  * [ ] Keycloak/OIDC Integration

    * [ ] Allow developpers to get user as an endpoint dependency

    ```python
    from fastnfurious import FastNFuriousAPI
    from fastnfurious.schemas import UserClaims


    app = FastNFuriousAPI(title="Demo Keycloak Integration")

    @app.get("/users/me", summary="Display user profile", response_model=UserClaims)
    async def me(user: UserClaims = app.current_user):
        """In this function we access the current user through dependency injection."""
        return user
    ```

  * [ ] Jaeger/OpenTracing Integration
    * [ ] Allow developpers to fetch tracer and span as dependency:

      ```python
      from typing import List
      from fastnfurious import FastNFuriousAPI
      from fastnfurious.tracing

      from .models import Product


      app = FastNFuriousAPI(title="Demo tracing")

      @app.get("products/", summary="Fetch list of products", response_model=List[Product])
      async def get_products(limit: int = 100, offset: int = 0):
          """Query the database and return a list of products. This operation will be traced in a span."""
          async with app.tracer.create_child_span("get_products_in_db") as new_span:
              # Capture query parameters
              new_span.set_tags({"limit": limit, "offset": offset})
              # Get cursor
              cursor = app.db.execute("SELECT * FROM products LIMIT :limit OFFSET :offset")
              # Convert curst to list
              results = await list(cursor)
              # Capture result length
              new_span.set_tags({"nb_rows": len(results)})
          # Return list of products
          return results
      ```

    * [ ] The application should also have built-in tracing
  * [ ] Standardized logs using loguru
    * [ ] All logs are intercepted and formatted using single formatter
  * [ ] Prometheus integration
    * [ ] Built-in metrics
    * [ ] Let user define its custom metrics
  * [ ] Tests using Tavern
  * [ ] Tests with TestClient
  * [ ] Mock implementation for NATS Request/Reply
  * [ ] Mock implementation for NATS Subscribe
  * [ ] Mock implementation for minio
* [x] NATS Service generator
  * [x] Deploy subscriptions from python script
  * [x] Deploy services from python script

### **Documentation**

* [ ] HTTP Services Documentation
  * [ ] OIDC Authentication
  * [ ] CI/CD Best Practices
  * [ ] Metrics Monitoring



