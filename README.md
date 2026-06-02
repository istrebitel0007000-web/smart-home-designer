# 🏠 Smart Home Designer — AI Powered

A full-featured floor plan and interior design tool with Claude AI advisor.

## Features

- 2D floor plan canvas with 44+ object types
- Multi-floor support (Ground, 1st, 2nd floor)
- AI placement advisor (powered by Claude)
- Drag & drop objects, zoom, pan, undo/redo
- 6 room presets (Bedroom, Living, Kitchen, Office, Bathroom, Studio)
- Layers panel, minimap, context menu, color picker
- Export as PNG

## Live Demo

Deployed at: https://smart-home-designer.onrender.com

## Tech Stack

- Pure HTML + CSS + JavaScript (no framework, no build step)
- Canvas API for drawing
- Anthropic Claude API for AI advisor

## Deploy on Render

1. Fork this repository
2. Go to [render.com](https://render.com)
3. New → Static Site
4. Connect your GitHub repo
5. Build command: (leave empty)
6. Publish directory: `.`
7. Click Deploy

## Local Development

Just open `index.html` in any browser — no server needed.

---

## 📐 Layers

### Introduction

Views should focus solely on handling HTTP requests and responses. They should avoid complex business logic, serving primarily to convert incoming request data into a specific format using serializers.

### Views

Views should focus solely on handling HTTP requests and responses. They should avoid complex business logic, serving primarily to convert incoming request data into a specific format using serializers.

### Serializer

Serializers are responsible for validating and transforming data but should not contain business logic or database queries. Their primary purpose is to ensure that the data conforms to expected formats and rules.

### Example Implementation

Here's an example of how to implement a view and serializer for creating a user:

```python
from rest_framework.decorators import api_view
from rest_framework import serializers
from rest_framework.response import Response
from rest_framework import status
from app.services import create_user

class CreateUserSerializerRequest(serializers.Serializer):
    first_name = serializers.CharField(required=True, max_length=255)
    last_name = serializers.CharField(required=True, max_length=255)
    is_active = serializers.BooleanField(required=True)
    email = serializers.EmailField(required=True)
    password = serializers.CharField(required=True, max_length=128)

class CreateUserSerializerResponse(serializers.Serializer):
    id = serializers.IntegerField()

@api_view(["POST"])
def create_user_handler(request):
    # Validate incoming data using the request serializer
    request_serializer = CreateUserSerializerRequest(data=request.data)
    request_serializer.is_valid(raise_exception=True)

    # Create the user using the validated data
    user = create_user(request.user, request_serializer.validated_data)

    # Prepare the response serializer with the newly created user's ID
    response_serializer = CreateUserSerializerResponse({'id': user.id})

    # Return a successful response with the user's ID
    return Response(status=status.HTTP_201_CREATED, data=response_serializer.data)
```

**Key Points:**
- **Views:** Designed to handle HTTP requests and responses, keeping the logic minimal and clean.
- **Serializers:** Focused on simple data validation and transformation, ensuring that input data meets the required criteria.

By maintaining clear separation between views and serializers, the application remains modular and easier to maintain.

---

## 📋 Code of Conduct

> Language: Python 3.9  
> This document outlines the set of coding policies that must be followed by developers in order to ensure consistent developer experience.

### 1. URL Naming Conventions

- **1.1** Names used in APIs should be in correct American English.
- **1.2** Use intuitive, familiar terminology where possible (`delete` is preferred over `erase`, `clear`, `destroy`).
- **1.3** Use the same name or term for the same concept across all APIs.
- **1.4** Use plural to indicate a collection type (e.g. `/api/v1/customers/` not `/api/v1/customer/`).
- **1.5** Use hyphens (`-`) to improve the readability of URIs (e.g. `/api/v1/orders/{id}/get-tracking/`).
- **1.6** Use CRUD function names in URIs:

```
GET    /api/v1/orders/{id}/detail/
GET    /api/v1/orders/list/
POST   /api/v1/orders/create/
PUT    /api/v1/orders/{id}/update/
PATCH  /api/v1/orders/{id}/partial-update/
```

- **1.7** Provide a unique URI for a resource when performing an action on it (e.g. `/api/v1/orders/{orderId}/trips/{tripId}/add-stop/`).

### 2. App Structure

```
order/
  __init__.py
  migrations/
  models/
    __init__.py
    order.py
    trip.py
  views/
    __init__.py
    create_order.py
    update_order.py
  services/
    __init__.py
    create_order.py
    update_order.py
  admin.py
  apps.py
  urls.py
```

#### 2.1 Serializers

- **2.1.1** Use `VerbNoun` form in naming. Compose the name with the corresponding view class name, adding `Request` or `Response` suffix:

```python
# Good
class CreateOrderRequestSerializer(Serializer): pass
class CreateOrderResponseSerializer(Serializer): pass
class CreateOrderView(APIView): pass
```

- **2.1.2** Use `Serializer` as base class. Avoid `ModelSerializer`. All serializers should inherit from `apps.core.serializer.Serializer`.

- **2.1.3** Serializers should be in their simplest form — no custom validation beyond basic field validations.

- **2.1.4** Provide default values for non-required fields (except in PATCH APIs):

```python
# Good
class CreateOrderRequestSerializer(Serializer):
    number = serializers.CharField(required=True)
    city = serializers.CharField(required=False, default=None)
    discount = serializers.DecimalField(required=False, default=0)
```

- **2.1.5** Define serializers in their corresponding view files — no need to separate them.

#### 2.2 Services

- **2.2.1** Structure service files in the immediate `services/` folder, avoid nesting unless it makes sense.
- **2.2.2** Follow Single Responsibility Principle — each service file handles only one action.
- **2.2.3** Use `verb_noun` naming convention (e.g. `create_order.py`, not `order_create.py`).
- **2.2.4** Only the main function should be public; internal helpers must be prefixed with `_`:

```python
# Good
def create_order(*args, **kwargs):
    _validate()
    _notify_driver()

def _validate(*args, **kwargs): pass
def _notify_driver(): pass
```

#### 2.3 Views

- **2.3.1** Use `VerbNoun` form in naming views (e.g. `CreateOrderView`, not `OrderCreateView`).
- **2.3.2** Use `verb_noun` form in naming view files (e.g. `create_order.py`).
- **2.3.3** Structure view files in the immediate `views/` folder, avoid nesting unless necessary.
- **2.3.4** Avoid `ViewSets` and function views — use Class-Based Views (`APIView`):

```python
# Good
class CreateOrderView(APIView):
    def post(self, request): pass

class UpdateOrderView(APIView):
    def put(self, request): pass
```

- **2.3.5** Keep views simple — no business logic, only request/response handling:

```python
# Good
class UpdateOrderView(APIView):
    def put(self, request, pk):
        serializer = UpdateOrderRequestSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        order = update_order(pk, serializer.data)
        serializer = UpdateOrderResponseSerializer(order)
        return Response(status=status.HTTP_200_OK, data=serializer.data)
```

#### 2.4 Models

- **2.4.1** Define foreign key relationships in quotes:

```python
# Good
class Trip(BaseModel):
    order = models.ForeignKey("Order", on_delete=models.CASCADE)
```

- **2.4.2** Avoid placing any logic in model definitions — all logic belongs in the service layer.

- **2.4.3** Define Enums inside the model and use singular naming:

```python
# Good
class Order(BaseModel):
    class Status(models.TextChoices):
        BOOKED = "booked", "Booked"
        IN_TRANSIT = "in_transit", "In Transit"
        DELIVERED = "delivered", "Delivered"

    status = models.CharField(max_length=35, choices=Status.choices, default=Status.BOOKED)
```

- **2.4.4** Use `JSONField` when applicable to group related fields:

```python
# Good
class Customer(BaseModel):
    class BillingAddress(TypedDict):
        type: str
        terms_days: int
        payment_method: str
        email: str
        address1: str
        address2: str
        city: str
        state: str
        zip_code: str

    billing_address: BillingAddress = models.JSONField()
```

### 3. Testing

#### 3.1 Structure

```
tests/
  order/
    fixtures/
      test_create_order/
      test_update_order/
    __init__.py
    test_create_order.py
    test_update_order.py
```

- **3.1.2** Avoid nesting unless it makes sense — keep related test files at the same level.
- **3.1.3** Use `test_verb_noun` convention in naming test files (e.g. `test_create_order.py`).

#### 3.2 Tests

- **3.2.1** One endpoint per test file — do not mix different endpoint tests.
- **3.2.2** Use separate fixtures per test file, named after the test file.
- **3.2.3** Avoid overloading tests with unnecessary fixtures. Only include what is needed.
- **3.2.4** Use generic `case_N` names with docstrings:

```python
# Good
def test_cancel_order_case_1(self):
    """
    Case: Attempt to cancel booked order
    Expected: Success
    """
    pass

def test_cancel_order_case_2(self):
    """
    Case: Attempt to cancel an order whose settlement is in paid status
    Expected: Failure
    """
    pass
```

- **3.2.5** Use model name with proper casing in fixture `"model"` field (e.g. `"order.OrderStop"`).
- **3.2.6** Avoid mixing different model fixtures into a single file — one model per fixture file.
- **3.2.7** Use hardcoded values in assert statements, not variables:

```python
# Good
self.assertEqual(order.status, Order.BOOKED)
self.assertEqual(order.note, "Lorem ipsum dolor sit amet")
```

- **3.2.8** Pass query params as a dict argument, not inline in the URL:

```python
# Good
query_params = {"page": 1, "page_size": 20}
response = self.client.get("/api/v1/orders/list/", query_params)
```

#### 3.3 Mocking

- **3.3.1** Use `mock_{mocked_function_name}` as the mock parameter name.
- **3.3.2** Only mock external calls — never mock internal functions.
- **3.3.3** Use mocking for datetime-dependent tests:

```python
# Good
@mock.patch("django.utils.timezone.now")
def test_list_weekly_orders(self, mock_now: Mock):
    mock_now.return_value = timezone.datetime(2024, 1, 5, tzinfo=timezone.utc)
    response = self.client.get("/api/v1/orders/list-weekly-orders/")
```

---

## 🧪 Tests

### Introduction

Tests are a crucial part of any project. Without them, supporting or refactoring becomes much more difficult and error-prone.

### Testing Approaches

There are many types of tests — unit, integration, and end-to-end (E2E) — as well as concepts like black-box and white-box testing. Each type has its own strengths and weaknesses, but combining them strikes the right balance.

### How to Write Tests

Django offers excellent built-in tools for testing: fixtures, `unittest`, and the request factory. Each test must follow these rules:

- Include at least two cases — one for success, one for failure.
- Tests must be independent — no test should rely on or call another.
- External resources must be mocked to isolate the functionality under test.
- Test code should be written in a declarative style — avoid loops or other constructs that make code harder to read.

### Testing Structure

```
tests/
  users/
    fixtures/
      test_user_list/
      test_user_detail/
      test_user_create/
        company.json
        token.json
    test_user_list.py
    test_user_detail.py
    test_user_create.py
```

### Example Fixture

```json
[
  {
    "model": "app.User",
    "pk": 1,
    "fields": {
      "email": "user1@gmail.com",
      "password": "password1",
      "company": 1,
      "first_name": "User 1",
      "last_name": "Kim",
      "is_superuser": true,
      "is_active": true,
      "created_at": "2024-03-23T04:48:47.402555+00",
      "updated_at": "2024-03-23T04:48:47.402555+00"
    }
  }
]
```

### Example Test

```python
from rest_framework.test import APITestCase

class TestUserCreate(APITestCase):
    fixtures = [
        "app/tests/users/fixtures/test_user_create/company.json",
        "app/tests/users/fixtures/test_user_create/token.json",
    ]

    def test_user_create(self):
        data = {
            'first_name': "Bekhzod",
            'last_name': "Tillakhanov",
            'is_active': True,
            'email': "admin@gmail.com",
            'password': "123456",
        }
        headers = {"Authorization": "Token TDgk8ATW132ZG-DYkfCw6Hhf55715SSs56DY"}
        response = self.client.post("/api/v1/users/create/", data, headers=headers)

        self.assertEqual(response.status_code, 201, response)
        self.assertIsNotNone(response.data["id"])

        user = User.objects.filter(id=response.data["id"]).get()
        self.assertEqual(user.first_name, data['first_name'])
        self.assertEqual(user.last_name, data['last_name'])
        self.assertEqual(user.company.id, 1)
        self.assertEqual(user.is_active, data['is_active'])
        self.assertEqual(user.email, data['email'])
```

### Mock

Mocks allow testing interactions with external resources outside of our control. Never use mocks to test internal features.

```python
from unittest.mock import patch, MagicMock
from rest_framework.test import APITestCase

class TestUserSignUpGoogle(APITestCase):
    fixtures = [
        "app/tests/users/fixtures/test_user_create/company.json",
        "app/tests/users/fixtures/test_user_create/token.json",
    ]

    @patch("apps.auth.services.google.requests")
    def test_user_create(self, mock_requests):
        mock_response = MagicMock()
        mock_response.status_code = 200
        mock_response.json.return_value = {"id": 1, "username": "Bekhzod"}
        mock_requests.get.return_value = mock_response

        data = {"code": "5Tihxd2KV0VsUFE_-VD4_Sq2yAMRDNuDnMDj0cF"}
        response = self.client.post("/api/v1/auth/google/sign-up/", data)

        self.assertEqual(response.status_code, 201, response)
        self.assertIsNotNone(response.data["token"])
```
