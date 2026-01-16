# Python Examples

## 1. Type Hints and Type Safety {#type-hints}
### Example 1
```python
# ❌ BAD - No type hints
def process_data(data):
    return data["value"] * 2

# ❌ BAD - Using Any without justification
from typing import Any

def process_request(data: Any) -> Any:
    return data

# ✅ GOOD - Proper type hints
from typing import Dict, Optional

def process_data(data: Dict[str, int]) -> int:
    return data["value"] * 2

# ✅ GOOD - Optional for nullable values
def find_user(user_id: str) -> Optional[User]:
    user = db.query(User).filter_by(id=user_id).first()
    return user

# ✅ GOOD - Generic types
from typing import List, TypeVar, Generic

T = TypeVar('T')

def first_element(items: List[T]) -> Optional[T]:
    return items[0] if items else None

# ✅ ACCEPTABLE - Any with justification
from typing import Any

class LegacyApiClient:
    def call_endpoint(self, endpoint: str, data: dict) -> Any:
        """
        Using Any here because legacy API returns different response 
        structures per endpoint. Will be replaced with typed responses 
        when migrating to API v2 (ticket: PROJ-456)
        """
        return self._client.post(endpoint, json=data)
```
### Example 2
```python
# ✅ GOOD - Dataclass for simple data
from dataclasses import dataclass
from datetime import datetime

@dataclass
class User:
    id: str
    name: str
    email: str
    created_at: datetime

# ✅ GOOD - Frozen dataclass for immutability
@dataclass(frozen=True)
class Point:
    x: float
    y: float

# ✅ GOOD - Pydantic model with validation
from pydantic import BaseModel, EmailStr, validator

class UserRequest(BaseModel):
    name: str
    email: EmailStr
    age: int
    
    @validator('name')
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('Name cannot be blank')
        return v
    
    @validator('age')
    def age_must_be_positive(cls, v: int) -> int:
        if v < 0:
            raise ValueError('Age must be positive')
        return v

# ✅ GOOD - Pydantic model for API response
class UserResponse(BaseModel):
    id: str
    name: str
    email: str
    created_at: datetime
    
    class Config:
        orm_mode = True  # Allows creation from ORM models
```

## 2. Error Handling and Exceptions {#error-handling}
### Example 1
```python
# ❌ BAD - Bare except
try:
    process_payment(order)
except:
    print("Error")

# ❌ BAD - Catching too broad exceptions
try:
    process_payment(order)
except Exception:
    pass  # Swallowing exception

# ❌ BAD - Generic exception
def validate_user(user_id: str) -> None:
    if not user_id:
        raise Exception("Invalid user")

# ✅ GOOD - Custom exception classes
class UserNotFoundException(Exception):
    """Raised when user is not found in the database."""
    
    def __init__(self, user_id: str):
        self.user_id = user_id
        super().__init__(f"User not found: {user_id}")

class PaymentFailedException(Exception):
    """Raised when payment processing fails."""
    
    def __init__(self, message: str, order_id: str, cause: Optional[Exception] = None):
        self.order_id = order_id
        self.cause = cause
        super().__init__(message)

class InsufficientBalanceException(Exception):
    """Raised when account balance is insufficient."""
    
    def __init__(self, required_amount: float, available_amount: float):
        self.required_amount = required_amount
        self.available_amount = available_amount
        super().__init__(
            f"Insufficient balance: required {required_amount}, "
            f"available {available_amount}"
        )

# ✅ GOOD - Specific exception handling
import logging

logger = logging.getLogger(__name__)

def process_order(order_id: str) -> Order:
    try:
        order = order_repository.find_by_id(order_id)
        if not order:
            raise OrderNotFoundException(order_id)
        
        payment_service.process_payment(order.payment_details)
        order.status = OrderStatus.PAID
        return order
        
    except PaymentFailedException as e:
        logger.error(f"Payment failed for order {order_id}", exc_info=True)
        raise
    except Exception as e:
        logger.error(f"Unexpected error processing order {order_id}", exc_info=True)
        raise OrderProcessingException(f"Failed to process order {order_id}", order_id, e)

# ✅ GOOD - Context manager for resource management
from contextlib import contextmanager

@contextmanager
def database_transaction(session):
    """Context manager for database transactions."""
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

# Usage
with database_transaction(session) as db:
    user = User(name="John", email="john@example.com")
    db.add(user)
```

## 3. Database and ORM Patterns {#database-orm}
### Example 1
```python
# ❌ BAD - SQL injection vulnerability
def find_user_by_email(email: str) -> Optional[User]:
    query = f"SELECT * FROM users WHERE email = '{email}'"
    return db.execute(query).first()

# ❌ BAD - No indexes defined
from sqlalchemy import Column, String, Integer
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    email = Column(String, nullable=False)
    status = Column(String, nullable=False)

# ✅ GOOD - Parameterized query
from sqlalchemy.orm import Session

def find_user_by_email(session: Session, email: str) -> Optional[User]:
    return session.query(User).filter(User.email == email).first()

# ✅ GOOD - Proper indexes and constraints
from sqlalchemy import Column, String, Integer, DateTime, Index
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    __table_args__ = (
        Index('idx_users_email', 'email', unique=True),
        Index('idx_users_status', 'status'),
        Index('idx_users_created_at', 'created_at'),
    )
    
    id = Column(Integer, primary_key=True)
    email = Column(String, nullable=False, unique=True)
    status = Column(String, nullable=False)
    created_at = Column(DateTime, nullable=False, default=datetime.utcnow)

# ✅ GOOD - Composite index
class Order(Base):
    __tablename__ = 'orders'
    __table_args__ = (
        Index('idx_orders_user_status', 'user_id', 'status'),
        Index('idx_orders_status_created', 'status', 'created_at'),
    )
    
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, nullable=False)
    status = Column(String, nullable=False)
    created_at = Column(DateTime, nullable=False, default=datetime.utcnow)

# ✅ GOOD - Session management with context manager
from contextlib import contextmanager
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

engine = create_engine(
    'postgresql://user:pass@localhost/db',
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True  # Verify connections before using
)
SessionLocal = sessionmaker(bind=engine)

@contextmanager
def get_db_session() -> Session:
    """Provide a transactional scope for database operations."""
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

# Usage
with get_db_session() as session:
    user = find_user_by_email(session, "user@example.com")
```

## 4. Async/Await Patterns {#async-await}
### Example 1
```python
# ❌ BAD - Sequential async calls
async def get_user_data(user_ids: List[str]) -> List[User]:
    users = []
    for user_id in user_ids:
        user = await api_client.get_user(user_id)
        users.append(user)
    return users

# ✅ GOOD - Parallel async calls
import asyncio
from typing import List

async def get_user_data(user_ids: List[str]) -> List[User]:
    tasks = [api_client.get_user(user_id) for user_id in user_ids]
    users = await asyncio.gather(*tasks)
    return list(users)

# ✅ GOOD - With timeout
async def get_user_with_timeout(user_id: str, timeout: float = 5.0) -> User:
    try:
        return await asyncio.wait_for(
            api_client.get_user(user_id),
            timeout=timeout
        )
    except asyncio.TimeoutError:
        raise TimeoutException(f"Request timeout after {timeout}s")

# ✅ GOOD - Async context manager
from contextlib import asynccontextmanager
import aiohttp

@asynccontextmanager
async def get_http_session():
    """Async context manager for HTTP session."""
    timeout = aiohttp.ClientTimeout(total=10)
    async with aiohttp.ClientSession(timeout=timeout) as session:
        yield session

# Usage
async def fetch_data(url: str) -> dict:
    async with get_http_session() as session:
        async with session.get(url) as response:
            return await response.json()

# ✅ GOOD - Error handling in async code
async def process_orders(order_ids: List[str]) -> List[Order]:
    async def process_single_order(order_id: str) -> Optional[Order]:
        try:
            return await order_service.process(order_id)
        except OrderNotFoundException:
            logger.warning(f"Order {order_id} not found")
            return None
        except Exception as e:
            logger.error(f"Failed to process order {order_id}", exc_info=True)
            return None
    
    results = await asyncio.gather(
        *[process_single_order(oid) for oid in order_ids],
        return_exceptions=False
    )
    
    return [r for r in results if r is not None]
```

## 5. Dependency Injection and Configuration {#dependency-injection}
### Example 1
```python
# ❌ BAD - Hardcoded dependencies
class UserService:
    def __init__(self):
        self.repository = UserRepository()  # Hardcoded
        self.email_service = EmailService()  # Hardcoded
    
    def create_user(self, data: dict) -> User:
        user = self.repository.save(data)
        self.email_service.send_welcome_email(user.email)
        return user

# ✅ GOOD - Constructor injection
from typing import Protocol

class UserRepositoryProtocol(Protocol):
    """Protocol for user repository."""
    
    def save(self, data: dict) -> User: ...
    def find_by_id(self, user_id: str) -> Optional[User]: ...

class EmailServiceProtocol(Protocol):
    """Protocol for email service."""
    
    def send_welcome_email(self, email: str) -> None: ...

class UserService:
    def __init__(
        self,
        repository: UserRepositoryProtocol,
        email_service: EmailServiceProtocol
    ):
        self.repository = repository
        self.email_service = email_service
    
    def create_user(self, data: dict) -> User:
        user = self.repository.save(data)
        self.email_service.send_welcome_email(user.email)
        return user

# ✅ GOOD - FastAPI dependency injection
from fastapi import Depends, FastAPI
from sqlalchemy.orm import Session

app = FastAPI()

def get_db() -> Session:
    """Dependency for database session."""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

def get_user_service(db: Session = Depends(get_db)) -> UserService:
    """Dependency for user service."""
    repository = UserRepository(db)
    email_service = EmailService()
    return UserService(repository, email_service)

@app.post("/users")
async def create_user(
    user_data: UserRequest,
    service: UserService = Depends(get_user_service)
) -> UserResponse:
    user = service.create_user(user_data.dict())
    return UserResponse.from_orm(user)
```
### Example 2
```python
# ✅ GOOD - Pydantic Settings for configuration
from pydantic import BaseSettings, Field, validator
from typing import List

class AppSettings(BaseSettings):
    """Application configuration."""
    
    # Database
    database_url: str = Field(..., env='DATABASE_URL')
    database_pool_size: int = Field(10, env='DATABASE_POOL_SIZE')
    
    # API
    api_key: str = Field(..., env='API_KEY')
    api_timeout: float = Field(30.0, env='API_TIMEOUT')
    
    # Features
    allowed_origins: List[str] = Field(
        default=['http://localhost:3000'],
        env='ALLOWED_ORIGINS'
    )
    
    @validator('api_timeout')
    def validate_timeout(cls, v: float) -> float:
        if v <= 0:
            raise ValueError('API timeout must be positive')
        return v
    
    class Config:
        env_file = '.env'
        env_file_encoding = 'utf-8'

# Usage
settings = AppSettings()

# ✅ GOOD - Using configuration
from sqlalchemy import create_engine

engine = create_engine(
    settings.database_url,
    pool_size=settings.database_pool_size
)
```

## 6. Testing Best Practices {#testing}
### Example 1
```python
# ✅ GOOD - Using fixtures
import pytest
from unittest.mock import Mock, patch

@pytest.fixture
def user_repository():
    """Fixture for user repository."""
    return Mock(spec=UserRepository)

@pytest.fixture
def email_service():
    """Fixture for email service."""
    return Mock(spec=EmailService)

@pytest.fixture
def user_service(user_repository, email_service):
    """Fixture for user service."""
    return UserService(user_repository, email_service)

# ✅ GOOD - Testing with fixtures
def test_create_user_success(user_service, user_repository, email_service):
    # Arrange
    user_data = {"name": "John", "email": "john@example.com"}
    expected_user = User(id="1", name="John", email="john@example.com")
    user_repository.save.return_value = expected_user
    
    # Act
    result = user_service.create_user(user_data)
    
    # Assert
    assert result == expected_user
    user_repository.save.assert_called_once_with(user_data)
    email_service.send_welcome_email.assert_called_once_with("john@example.com")

# ✅ GOOD - Parametrized tests
@pytest.mark.parametrize("email,expected", [
    ("user@example.com", True),
    ("invalid-email", False),
    ("", False),
    ("user@", False),
    ("@example.com", False),
])
def test_email_validation(email: str, expected: bool):
    assert is_valid_email(email) == expected

# ✅ GOOD - Testing exceptions
def test_create_user_raises_validation_error(user_service):
    with pytest.raises(ValidationError) as exc_info:
        user_service.create_user({"name": "", "email": "invalid"})
    
    assert "Name cannot be blank" in str(exc_info.value)

# ✅ GOOD - Async test
@pytest.mark.asyncio
async def test_fetch_user_data_async():
    async with get_http_session() as session:
        data = await fetch_data("https://api.example.com/users/1")
        assert data["id"] == "1"
```

## 7. Code Style and Formatting {#code-style}
### Example 1
```python
# ✅ GOOD - Import ordering (isort)
# Standard library imports
import os
import sys
from datetime import datetime
from typing import List, Optional

# Third-party imports
import requests
from fastapi import FastAPI
from sqlalchemy import Column, String

# Local imports
from app.models import User
from app.services import UserService
from app.utils import validate_email

# ✅ GOOD - Function and variable naming
def calculate_total_price(items: List[Item]) -> float:
    """Calculate total price of items."""
    total = 0.0
    for item in items:
        total += item.price * item.quantity
    return total

# ✅ GOOD - Class naming
class UserNotFoundException(Exception):
    """Exception raised when user is not found."""
    pass

class PaymentProcessor:
    """Process payments for orders."""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    def process(self, amount: float) -> bool:
        """Process payment for given amount."""
        # Implementation
        pass

# ✅ GOOD - Constants
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30.0
API_BASE_URL = "https://api.example.com"
```

## 8. Accessibility & Localization {#accessibility}
### Example 1
```python
{% load i18n %}
<button type="submit" aria-label="{% trans 'Save changes' %}">
  {% trans "Save" %}
</button>
```

## 9. Testing & Tooling Enhancements {#testing-tooling}
### Example 1
```python
@pytest.mark.asyncio
async def test_fetch_user(async_client):
    response = await async_client.get("/users/123")
    assert response.status_code == 200

from hypothesis import given, strategies as st

@given(st.text())
def test_slugify_has_no_spaces(value: str):
    assert " " not in slugify(value)
```

## 10. Security & Configuration {#security}
### Example 1
```python
class Settings(BaseSettings):
    database_url: AnyUrl
    jwt_secret: str
    api_timeout: PositiveFloat = 5.0

    class Config:
        env_file = ".env"
```

## Tools for Code Quality
### Example 1
```bash
# Format code
black src/
isort src/

# Linting
pylint src/
flake8 src/
mypy src/

# Security scanning
bandit -r src/

# Testing
pytest tests/ --cov=src --cov-report=html
```
### Example 2
```toml
[tool.black]
line-length = 88
target-version = ['py39']

[tool.isort]
profile = "black"
line_length = 88

[tool.mypy]
python_version = "3.9"
strict = true
warn_return_any = true
warn_unused_configs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
```
