*Document Version: 2.0*  
*Last Updated: August 3, 2025*  
*Document Owner: Aditya Tripathi*

---

# Universal Authentication Platform (UAP)
## Technical Architecture Design Document

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Technology Stack](#technology-stack)
3. [System Architecture](#system-architecture)
4. [Core Components](#core-components)
5. [Database Architecture](#database-architecture)
6. [API Design](#api-design)
7. [Integration Patterns](#integration-patterns)
8. [Security Architecture](#security-architecture)
9. [Deployment Architecture](#deployment-architecture)
10. [Performance & Scalability](#performance--scalability)
11. [Monitoring & Observability](#monitoring--observability)
12. [Development Guidelines](#development-guidelines)

---

## Architecture Overview

### Design Principles

1. **Microservice-First**: UAP operates as a standalone microservice that integrates seamlessly into existing application stacks
2. **Zero-Migration**: Works with existing user databases and APIs without requiring data migration
3. **Configuration-Driven**: All authentication behaviors controlled through configuration, not code changes
4. **Extensible Middleware**: Plugin-based architecture for custom authentication workflows
5. **Cloud-Native**: Containerized, stateless, and horizontally scalable
6. **Security-First**: Built-in security best practices with comprehensive audit logging

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Host Application Ecosystem                    │
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │
│  │   Frontend   │    │   Backend   │    │   Existing User     │ │
│  │ Applications │    │  Services   │    │   Database/APIs     │ │
│  └──────┬──────┘    └──────┬──────┘    └──────────┬──────────┘ │
│         │                  │                      │            │
│         │                  │                      │            │
│         └──────────────────┼──────────────────────┘            │
│                            │                                   │
│                    ┌───────▼───────┐                          │
│                    │  Load Balancer │                          │
│                    └───────┬───────┘                          │
│                            │                                   │
│         ┌──────────────────┼──────────────────┐               │
│         │          UAP Microservice           │               │
│         │                  │                  │               │
│         │  ┌───────────────▼───────────────┐  │               │
│         │  │        API Gateway           │  │               │
│         │  │   (FastAPI + Middleware)     │  │               │
│         │  └───────────────┬───────────────┘  │               │
│         │                  │                  │               │
│         │  ┌───────────────▼───────────────┐  │               │
│         │  │     Authentication Core       │  │               │
│         │  │  ┌─────────────────────────┐  │  │               │
│         │  │  │  Configuration Manager  │  │  │               │
│         │  │  └─────────────────────────┘  │  │               │
│         │  │  ┌─────────────────────────┐  │  │               │
│         │  │  │  Authentication Engine  │  │  │               │
│         │  │  └─────────────────────────┘  │  │               │
│         │  │  ┌─────────────────────────┐  │  │               │
│         │  │  │    Session Manager      │  │  │               │
│         │  │  └─────────────────────────┘  │  │               │
│         │  │  ┌─────────────────────────┐  │  │               │
│         │  │  │   Data Access Layer     │  │◄─┼───────────────┘
│         │  │  └─────────────────────────┘  │  │
│         │  └─────────────────────────────────┘  │
│         │                  │                    │
│         │  ┌───────────────▼───────────────┐    │
│         │  │      Middleware Stack         │    │
│         │  │ ┌─────────┐ ┌─────────────┐   │    │
│         │  │ │CAPTCHA  │ │Rate Limiter │   │    │
│         │  │ └─────────┘ └─────────────┘   │    │
│         │  │ ┌─────────┐ ┌─────────────┐   │    │
│         │  │ │   MFA   │ │Custom Logic │   │    │
│         │  │ └─────────┘ └─────────────┘   │    │
│         │  └─────────────────────────────────┘    │
│         │                  │                      │
│         │  ┌───────────────▼───────────────┐      │
│         │  │        Support Services       │      │
│         │  │ ┌─────────┐ ┌─────────────┐   │      │
│         │  │ │ Caching │ │   Logging   │   │      │
│         │  │ └─────────┘ └─────────────┘   │      │
│         │  │ ┌─────────┐ ┌─────────────┐   │      │
│         │  │ │Monitoring│ │Health Check │   │      │
│         │  │ └─────────┘ └─────────────┘   │      │
│         │  └─────────────────────────────────┘      │
│         └──────────────────────────────────────────┘
└─────────────────────────────────────────────────────────────────┘
```

---

## Technology Stack

### Core Framework
- **Python 3.9+**: Primary programming language
- **FastAPI**: Modern, fast web framework for building APIs
- **Pydantic**: Data validation and settings management
- **Uvicorn**: ASGI server for production deployment

### Database & Caching
- **SQLAlchemy 2.0+**: ORM for database interactions
- **Alembic**: Database migration management
- **Redis**: Session storage and caching
- **PostgreSQL**: Primary database for UAP metadata (recommended)
- **Multi-database support**: MySQL, SQLite, SQL Server, Oracle

### Security & Authentication
- **passlib**: Password hashing and verification
- **python-jose[cryptography]**: JWT token handling
- **cryptography**: Encryption and cryptographic operations
- **bcrypt/argon2**: Password hashing algorithms

### HTTP & Networking
- **httpx**: Async HTTP client for external API calls
- **websockets**: WebSocket support for real-time features
- **python-multipart**: File upload support

### Middleware & Extensions
- **slowapi**: Rate limiting middleware
- **prometheus-client**: Metrics collection
- **structlog**: Structured logging
- **sentry-sdk**: Error tracking and monitoring

### Development & Testing
- **pytest**: Testing framework
- **pytest-asyncio**: Async testing support
- **black**: Code formatting
- **flake8**: Code linting
- **mypy**: Static type checking
- **pre-commit**: Git hooks for code quality

### Deployment & Infrastructure
- **Docker**: Containerization
- **Docker Compose**: Local development environment
- **Kubernetes**: Container orchestration
- **Helm**: Kubernetes package management
- **GitHub Actions**: CI/CD pipeline

---

## System Architecture

### Layered Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  REST APIs  │  │  GraphQL    │  │  WebSocket APIs     │ │
│  │  (FastAPI)  │  │  (Optional) │  │  (Real-time)        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway Layer                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ Request     │  │  Response   │  │  Error Handling     │ │
│  │ Validation  │  │ Formatting  │  │  & Logging          │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────┐
│                   Middleware Layer                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   CORS      │  │Rate Limiting│  │  Authentication     │ │
│  │ Middleware  │  │ Middleware  │  │  Middleware         │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Security   │  │   Custom    │  │   Monitoring        │ │
│  │ Middleware  │  │ Middleware  │  │   Middleware        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────┐
│                   Business Logic Layer                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │Authentication│  │   Session   │  │  Configuration      │ │
│  │   Engine     │  │  Manager    │  │    Manager          │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   User      │  │   Token     │  │   Middleware        │ │
│  │ Validator   │  │  Manager    │  │   Orchestrator      │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────┐
│                   Data Access Layer                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Database   │  │    Cache    │  │   External API      │ │
│  │  Adapters   │  │  Adapters   │  │    Adapters         │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────┐
│                   Infrastructure Layer                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Database   │  │    Redis    │  │   External APIs     │ │
│  │ (Host Data) │  │   Cache     │  │   (SMS, Email)      │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Component Interaction Flow

```
┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client    │───▶│   API Gateway   │───▶│   Middleware    │
│ Application │    │   (FastAPI)     │    │     Stack       │
└─────────────┘    └─────────────────┘    └─────────────────┘
                                                    │
                                                    ▼
┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Response  │◄───│  Authentication │◄───│  Configuration  │
│  Formatter  │    │     Engine      │    │    Manager      │
└─────────────┘    └─────────────────┘    └─────────────────┘
                             │
                             ▼
┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Session   │───▶│  Data Access    │───▶│   Host Data     │
│   Manager   │    │     Layer       │    │   Source        │
└─────────────┘    └─────────────────┘    └─────────────────┘
```

---

## Core Components

### 1. Configuration Manager

**Purpose**: Manages all UAP configuration including authentication methods, database connections, and middleware settings.

```python
# core/config/manager.py
from typing import Dict, Any, Optional
from pydantic import BaseSettings, validator
from enum import Enum

class AuthenticationType(str, Enum):
    USERNAME_PASSWORD = "username_password"
    EMAIL_PASSWORD = "email_password"
    PHONE_PASSWORD = "phone_password"
    CUSTOM = "custom"

class DatabaseType(str, Enum):
    POSTGRESQL = "postgresql"
    MYSQL = "mysql"
    SQLITE = "sqlite"
    SQL_SERVER = "sqlserver"
    ORACLE = "oracle"

class ConfigurationManager(BaseSettings):
    # UAP Core Settings
    service_name: str = "UAP"
    version: str = "1.0.0"
    debug: bool = False
    
    # Authentication Configuration
    auth_type: AuthenticationType
    username_field: str = "username"
    password_field: str = "password"
    email_field: Optional[str] = "email"
    phone_field: Optional[str] = "phone"
    
    # Database Configuration (Host Application Data)
    host_db_type: DatabaseType
    host_db_url: str
    host_user_table: str
    host_role_table: Optional[str] = None
    
    # Session Configuration
    session_duration: int = 3600  # seconds
    max_session_duration: int = 86400  # 24 hours
    multi_device_sessions: bool = True
    
    # Security Configuration
    password_hash_algorithm: str = "bcrypt"
    jwt_secret_key: str
    jwt_algorithm: str = "HS256"
    jwt_expiration: int = 3600
    
    # Redis Configuration (UAP Internal)
    redis_url: str = "redis://localhost:6379"
    
    # Rate Limiting
    rate_limit_enabled: bool = True
    max_attempts: int = 5
    rate_limit_window: int = 300  # 5 minutes
    
    # Middleware Configuration
    middleware_stack: List[Dict[str, Any]] = []
    
    class Config:
        env_file = ".env"
        case_sensitive = False

    @validator('middleware_stack')
    def validate_middleware_stack(cls, v):
        required_fields = ['name', 'enabled']
        for middleware in v:
            for field in required_fields:
                if field not in middleware:
                    raise ValueError(f"Middleware missing required field: {field}")
        return v
```

### 2. Authentication Engine

**Purpose**: Core authentication logic that validates credentials against host application data.

```python
# core/auth/engine.py
from typing import Optional, Dict, Any, Union
from abc import ABC, abstractmethod
import hashlib
import bcrypt
from passlib.context import CryptContext

class AuthenticationResult:
    def __init__(self, success: bool, user_data: Optional[Dict] = None, 
                 error: Optional[str] = None, metadata: Optional[Dict] = None):
        self.success = success
        self.user_data = user_data
        self.error = error
        self.metadata = metadata or {}

class PasswordHasher:
    def __init__(self, algorithm: str = "bcrypt"):
        self.pwd_context = CryptContext(
            schemes=["bcrypt", "argon2", "pbkdf2_sha256"],
            deprecated="auto"
        )
        self.algorithm = algorithm

    def verify_password(self, plain_password: str, hashed_password: str) -> bool:
        return self.pwd_context.verify(plain_password, hashed_password)

    def hash_password(self, password: str) -> str:
        return self.pwd_context.hash(password)

    def verify_and_upgrade_password(self, plain_password: str, hashed_password: str) -> (bool, Optional[str]):
        """
        Verifies a password and checks if the hash needs to be upgraded.
        Returns a tuple of (is_valid, new_hash).
        new_hash is not None if the password is valid and the hash uses a deprecated algorithm.
        """
        is_valid, new_hash = self.pwd_context.verify_and_update(plain_password, hashed_password)
        return is_valid, new_hash

class AuthenticationEngine:
    def __init__(self, config: ConfigurationManager, data_access_layer):
        self.config = config
        self.dal = data_access_layer
        self.password_hasher = PasswordHasher(config.password_hash_algorithm)
    
    async def authenticate(self, credentials: Dict[str, Any]) -> AuthenticationResult:
        """
        Main authentication method that validates credentials
        """
        try:
            # Extract authentication fields based on configuration
            auth_field_value = self._extract_auth_field(credentials)
            password = credentials.get(self.config.password_field)
            
            if not auth_field_value or not password:
                return AuthenticationResult(
                    success=False, 
                    error="Missing required credentials"
                )
            
            # Fetch user data from host application
            user_data = await self.dal.get_user_by_auth_field(
                field_name=self._get_auth_field_name(),
                field_value=auth_field_value
            )
            
            if not user_data:
                return AuthenticationResult(
                    success=False,
                    error="User not found"
                )
            
            # Verify password and check for potential upgrade
            stored_password_hash = user_data.get(self.config.password_field)
            is_valid, new_hash = self.password_hasher.verify_and_upgrade_password(password, stored_password_hash)

            if not is_valid:
                return AuthenticationResult(
                    success=False,
                    error="Invalid credentials"
                )

            # Remove sensitive data from response
            safe_user_data = {k: v for k, v in user_data.items()
                              if k != self.config.password_field}

            # Include metadata about hash upgrade
            auth_metadata = {"auth_method": self.config.auth_type}
            if new_hash:
                auth_metadata["new_hash"] = new_hash
                auth_metadata["hash_upgraded"] = True

            return AuthenticationResult(
                success=True,
                user_data=safe_user_data,
                metadata=auth_metadata
            )
            
        except Exception as e:
            return AuthenticationResult(
                success=False,
                error=f"Authentication error: {str(e)}"
            )
    
    def _extract_auth_field(self, credentials: Dict[str, Any]) -> Optional[str]:
        """Extract the authentication field value based on auth type"""
        if self.config.auth_type == AuthenticationType.USERNAME_PASSWORD:
            return credentials.get(self.config.username_field)
        elif self.config.auth_type == AuthenticationType.EMAIL_PASSWORD:
            return credentials.get(self.config.email_field)
        elif self.config.auth_type == AuthenticationType.PHONE_PASSWORD:
            return credentials.get(self.config.phone_field)
        else:  # CUSTOM
            return credentials.get(self.config.username_field)
    
    def _get_auth_field_name(self) -> str:
        """Get the database field name for authentication"""
        if self.config.auth_type == AuthenticationType.USERNAME_PASSWORD:
            return self.config.username_field
        elif self.config.auth_type == AuthenticationType.EMAIL_PASSWORD:
            return self.config.email_field
        elif self.config.auth_type == AuthenticationType.PHONE_PASSWORD:
            return self.config.phone_field
        else:  # CUSTOM
            return self.config.username_field
```

### 3. Session Manager

**Purpose**: Handles session creation, validation, and cleanup with role-based expiration policies.

```python
# core/session/manager.py
from typing import Optional, Dict, Any
import jwt
import redis.asyncio as redis
from datetime import datetime, timedelta
import json
import uuid

class SessionToken:
    def __init__(self, token: str, user_id: str, expires_at: datetime, 
                 device_id: Optional[str] = None, metadata: Optional[Dict] = None):
        self.token = token
        self.user_id = user_id
        self.expires_at = expires_at
        self.device_id = device_id
        self.metadata = metadata or {}

class SessionManager:
    def __init__(self, config: ConfigurationManager):
        self.config = config
        self.redis_client = redis.from_url(config.redis_url)
        self.jwt_secret = config.jwt_secret_key
        self.jwt_algorithm = config.jwt_algorithm
    
    async def create_session(self, user_data: Dict[str, Any], 
                           device_id: Optional[str] = None) -> SessionToken:
        """Create a new session for authenticated user"""
        user_id = str(user_data.get('id', user_data.get('user_id')))
        session_id = str(uuid.uuid4())
        
        # Determine session duration based on user role
        session_duration = await self._get_session_duration(user_data)
        expires_at = datetime.utcnow() + timedelta(seconds=session_duration)
        
        # Create JWT token
        payload = {
            'session_id': session_id,
            'user_id': user_id,
            'device_id': device_id,
            'exp': expires_at,
            'iat': datetime.utcnow(),
            'iss': self.config.service_name
        }
        
        token = jwt.encode(payload, self.jwt_secret, algorithm=self.jwt_algorithm)
        
        # Store session in Redis
        session_data = {
            'user_id': user_id,
            'device_id': device_id,
            'created_at': datetime.utcnow().isoformat(),
            'expires_at': expires_at.isoformat(),
            'user_data': user_data
        }
        
        await self.redis_client.setex(
            f"session:{session_id}",
            session_duration,
            json.dumps(session_data, default=str)
        )
        
        # Handle multi-device sessions
        if not self.config.multi_device_sessions:
            await self._invalidate_other_sessions(user_id, session_id)
        
        return SessionToken(
            token=token,
            user_id=user_id,
            expires_at=expires_at,
            device_id=device_id,
            metadata={'session_id': session_id}
        )
    
    async def validate_session(self, token: str) -> Optional[Dict[str, Any]]:
        """Validate session token and return user data"""
        try:
            # Decode JWT token
            payload = jwt.decode(
                token, 
                self.jwt_secret, 
                algorithms=[self.jwt_algorithm]
            )
            
            session_id = payload.get('session_id')
            if not session_id:
                return None
            
            # Check session in Redis
            session_data = await self.redis_client.get(f"session:{session_id}")
            if not session_data:
                return None
            
            session_info = json.loads(session_data)
            return {
                'user_data': session_info['user_data'],
                'session_id': session_id,
                'device_id': session_info.get('device_id'),
                'expires_at': session_info['expires_at']
            }
            
        except jwt.ExpiredSignatureError:
            return None
        except jwt.InvalidTokenError:
            return None
        except Exception:
            return None
    
    async def invalidate_session(self, token: str) -> bool:
        """Invalidate a specific session"""
        try:
            payload = jwt.decode(
                token, 
                self.jwt_secret, 
                algorithms=[self.jwt_algorithm],
                options={"verify_exp": False}  # Allow expired tokens for logout
            )
            
            session_id = payload.get('session_id')
            if session_id:
                await self.redis_client.delete(f"session:{session_id}")
                return True
            return False
        except Exception:
            return False
    
    async def refresh_session(self, token: str) -> Optional[SessionToken]:
        """Refresh an existing session"""
        session_info = await self.validate_session(token)
        if not session_info:
            return None
        
        # Invalidate old session
        await self.invalidate_session(token)
        
        # Create new session
        return await self.create_session(
            session_info['user_data'],
            session_info.get('device_id')
        )
    
    async def _get_session_duration(self, user_data: Dict[str, Any]) -> int:
        """Determine session duration based on user role"""
        # Default session duration
        duration = self.config.session_duration
        
        # Check for role-based duration (if role table is configured)
        user_role = user_data.get('role', user_data.get('user_role'))
        if user_role:
            # Role-based session duration logic
            role_durations = {
                'admin': 7200,      # 2 hours
                'user': 3600,       # 1 hour
                'guest': 1800       # 30 minutes
            }
            duration = role_durations.get(user_role.lower(), duration)
        
        # Ensure duration doesn't exceed maximum
        return min(duration, self.config.max_session_duration)
    
    async def _invalidate_other_sessions(self, user_id: str, current_session_id: str):
        """Invalidate all other sessions for a user (single device policy)"""
        user_sessions_key = f"user_sessions:{user_id}"
        
        # Get all previous session IDs for the user
        other_session_ids = await self.redis_client.smembers(user_sessions_key)
        
        # Invalidate all old sessions
        if other_session_ids:
            session_keys_to_delete = [f"session:{sid}" for sid in other_session_ids]
            await self.redis_client.delete(*session_keys_to_delete)
        
        # Clear the user's session set and add only the new session
        pipe = self.redis_client.pipeline()
        pipe.delete(user_sessions_key)
        pipe.sadd(user_sessions_key, current_session_id)
        await pipe.execute()
```

### 4. Data Access Layer

**Purpose**: Abstracts access to host application's user data through database connections or API calls.

```python
# core/data/access_layer.py
from typing import Optional, Dict, Any, List
from abc import ABC, abstractmethod
import sqlalchemy as sa
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker
import httpx

class DataAccessLayer(ABC):
    @abstractmethod
    async def get_user_by_auth_field(self, field_name: str, field_value: str) -> Optional[Dict[str, Any]]:
        pass
    
    @abstractmethod
    async def get_user_roles(self, user_id: str) -> List[str]:
        pass
    
    @abstractmethod
    async def health_check(self) -> bool:
        pass

class DatabaseAccessLayer(DataAccessLayer):
    def __init__(self, config: ConfigurationManager):
        self.config = config
        self.engine = create_async_engine(
            config.host_db_url,
            echo=config.debug,
            pool_pre_ping=True,
            pool_recycle=3600
        )
        self.async_session = sessionmaker(
            self.engine, 
            class_=AsyncSession, 
            expire_on_commit=False
        )
    
    async def get_user_by_auth_field(self, field_name: str, field_value: str) -> Optional[Dict[str, Any]]:
        """Fetch user data from host database"""
        async with self.async_session() as session:
            # Build dynamic query based on configuration
            query = sa.text(f"""
                SELECT * FROM {self.config.host_user_table} 
                WHERE {field_name} = :field_value
            """)
            
            result = await session.execute(query, {"field_value": field_value})
            row = result.fetchone()
            
            if row:
                return dict(row._mapping)
            return None
    
    async def get_user_roles(self, user_id: str) -> List[str]:
        """Fetch user roles from host database"""
        if not self.config.host_role_table:
            return []
        
        async with self.async_session() as session:
            query = sa.text(f"""
                SELECT role_name FROM {self.config.host_role_table} 
                WHERE user_id = :user_id
            """)
            
            result = await session.execute(query, {"user_id": user_id})
            rows = result.fetchall()
            
            return [row[0] for row in rows]
    
    async def health_check(self) -> bool:
        """Check database connectivity"""
        try:
            async with self.async_session() as session:
                await session.execute(sa.text("SELECT 1"))
                return True
        except Exception:
            return False

class APIAccessLayer(DataAccessLayer):
    def __init__(self, config: ConfigurationManager):
        self.config = config
        self.client = httpx.AsyncClient(timeout=30.0)
        # API configuration would be part of config
        self.base_url = config.host_api_base_url
        self.api_key = config.host_api_key
    
    async def get_user_by_auth_field(self, field_name: str, field_value: str) -> Optional[Dict[str, Any]]:
        """Fetch user data from host API"""
        try:
            response = await self.client.post(
                f"{self.base_url}/auth/validate",
                json={field_name: field_value},
                headers={"Authorization": f"Bearer {self.api_key}"}
            )
            
            if response.status_code == 200:
                return response.json()
            return None
        except Exception:
            return None
    
    async def get_user_roles(self, user_id: str) -> List[str]:
        """Fetch user roles from host API"""
        try:
            response = await self.client.get(
                f"{self.base_url}/users/{user_id}/roles",
                headers={"Authorization": f"Bearer {self.api_key}"}
            )
            
            if response.status_code == 200:
                data = response.json()
                return data.get('roles', [])
            return []
        except Exception:
            return []
    
    async def health_check(self) -> bool:
        """Check API connectivity"""
        try:
            response = await self.client.get(
                f"{self.base_url}/health",
                headers={"Authorization": f"Bearer {self.api_key}"}
            )
            return response.status_code == 200
        except Exception:
            return False

class DataAccessFactory:
    @staticmethod
    def create_data_access_layer(config: ConfigurationManager) -> DataAccessLayer:
        """Factory method to create appropriate data access layer based on explicit config."""
        if getattr(config, 'data_source_type', 'database') == 'api':
            return APIAccessLayer(config)
        else:
            return DatabaseAccessLayer(config)
```

---

## Database Architecture

### UAP Internal Database Schema

UAP maintains its own lightweight database for configuration, audit logs, and operational data:

```sql
-- UAP Internal Database Schema (PostgreSQL)

-- Configuration storage
CREATE TABLE uap_configurations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id VARCHAR(255) NOT NULL,
    config_name VARCHAR(255) NOT NULL,
    config_data JSONB NOT NULL,
    version INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by VARCHAR(255),
    UNIQUE(organization_id, config_name)
);

-- Active sessions metadata (for monitoring and cleanup)
CREATE TABLE uap_sessions (
    session_id UUID PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    device_id VARCHAR(255),
    organization_id VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    last_activity TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    ip_address INET,
    user_agent TEXT,
    metadata JSONB
);

-- Audit log for all authentication events
CREATE TABLE uap_audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id VARCHAR(255) NOT NULL,
    event_type VARCHAR(100) NOT NULL, -- LOGIN, LOGOUT, FAILED_LOGIN, etc.
    user_id VARCHAR(255),
    session_id UUID,
    ip_address INET,
    user_agent TEXT,
    success BOOLEAN NOT NULL,
    error_message TEXT,
    metadata JSONB,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Rate limiting data
CREATE TABLE uap_rate_limits (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    identifier VARCHAR(255) NOT NULL, -- IP address or user ID
    identifier_type VARCHAR(50) NOT NULL, -- 'ip' or 'user'
    organization_id VARCHAR(255) NOT NULL,
    attempt_count INTEGER DEFAULT 1,
    window_start TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    blocked_until TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(identifier, identifier_type, organization_id)
);

-- Middleware execution logs
CREATE TABLE uap_middleware_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID,
    organization_id VARCHAR(255) NOT NULL,
    middleware_name VARCHAR(255) NOT NULL,
    execution_order INTEGER NOT NULL,
    execution_time_ms INTEGER,
    success BOOLEAN NOT NULL,
    error_message TEXT,
    input_data JSONB,
    output_data JSONB,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create indexes for performance
CREATE INDEX idx_uap_configurations_org_active ON uap_configurations(organization_id, is_active);
CREATE INDEX idx_uap_sessions_user_org ON uap_sessions(user_id, organization_id);
CREATE INDEX idx_uap_sessions_expires_at ON uap_sessions(expires_at);
CREATE INDEX idx_uap_audit_logs_org_timestamp ON uap_audit_logs(organization_id, timestamp);
CREATE INDEX idx_uap_audit_logs_user_timestamp ON uap_audit_logs(user_id, timestamp);
CREATE INDEX idx_uap_rate_limits_identifier_org ON uap_rate_limits(identifier, organization_id);
```

### Host Application Database Integration

UAP connects to existing host databases without requiring schema changes:

```python
# core/data/schema_adapter.py
from typing import Dict, Any, Optional
import sqlalchemy as sa
from sqlalchemy import MetaData, Table, inspect

class SchemaAdapter:
    def __init__(self, engine, config: ConfigurationManager):
        self.engine = engine
        self.config = config
        self.metadata = MetaData()
        self._user_table = None
        self._role_table = None
    
    async def initialize(self):
        """Initialize schema information from host database"""
        async with self.engine.connect() as conn:
            # Reflect user table structure
            self._user_table = Table(
                self.config.host_user_table,
                self.metadata,
                autoload_with=conn.sync_engine
            )
            
            # Reflect role table if configured
            if self.config.host_role_table:
                self._role_table = Table(
                    self.config.host_role_table,
                    self.metadata,
                    autoload_with=conn.sync_engine
                )
    
    def get_user_table_columns(self) -> Dict[str, Any]:
        """Get user table column information"""
        if not self._user_table:
            return {}
        
        return {col.name: str(col.type) for col in self._user_table.columns}
    
    def validate_configuration(self) -> Dict[str, Any]:
        """Validate that configured fields exist in host tables"""
        validation_result = {
            "valid": True,
            "errors": [],
            "warnings": []
        }
        
        if not self._user_table:
            validation_result["valid"] = False
            validation_result["errors"].append("Cannot access user table")
            return validation_result
        
        # Check required fields exist
        user_columns = [col.name for col in self._user_table.columns]
        
        # Validate authentication field
        auth_field = self._get_auth_field_name()
        if auth_field not in user_columns:
            validation_result["valid"] = False
            validation_result["errors"].append(f"Authentication field '{auth_field}' not found in user table")
        
        # Validate password field
        if self.config.password_field not in user_columns:
            validation_result["valid"] = False
            validation_result["errors"].append(f"Password field '{self.config.password_field}' not found in user table")
        
        return validation_result
    
    def _get_auth_field_name(self) -> str:
        """Get the database field name for authentication"""
        if self.config.auth_type == AuthenticationType.USERNAME_PASSWORD:
            return self.config.username_field
        elif self.config.auth_type == AuthenticationType.EMAIL_PASSWORD:
            return self.config.email_field
        elif self.config.auth_type == AuthenticationType.PHONE_PASSWORD:
            return self.config.phone_field
        else:  # CUSTOM
            return self.config.username_field
```

---

## API Design

### REST API Endpoints

```python
# api/v1/auth.py
from fastapi import APIRouter, Depends, HTTPException, Request
from fastapi.security import HTTPBearer
from pydantic import BaseModel
from typing import Optional, Dict, Any
import structlog

logger = structlog.get_logger()

class AuthenticationRequest(BaseModel):
    username: Optional[str] = None
    email: Optional[str] = None
    phone: Optional[str] = None
    password: str
    device_id: Optional[str] = None
    metadata: Optional[Dict[str, Any]] = None

class AuthenticationResponse(BaseModel):
    success: bool
    token: Optional[str] = None
    user_data: Optional[Dict[str, Any]] = None
    expires_at: Optional[str] = None
    error: Optional[str] = None

class SessionValidationResponse(BaseModel):
    valid: bool
    user_data: Optional[Dict[str, Any]] = None
    expires_at: Optional[str] = None
    session_id: Optional[str] = None

class LogoutResponse(BaseModel):
    success: bool
    message: str

router = APIRouter(prefix="/api/v1/auth", tags=["authentication"])
security = HTTPBearer()

@router.post("/authenticate", response_model=AuthenticationResponse)
async def authenticate(
    request: AuthenticationRequest,
    http_request: Request,
    auth_engine = Depends(get_auth_engine),
    session_manager = Depends(get_session_manager),
    audit_logger = Depends(get_audit_logger)
):
    """
    Authenticate user credentials and create session
    """
    client_ip = http_request.client.host
    user_agent = http_request.headers.get("User-Agent", "")
    
    try:
        # Convert request to dictionary for auth engine
        credentials = request.dict(exclude_none=True)
        
        # Authenticate user
        auth_result = await auth_engine.authenticate(credentials)
        
        if not auth_result.success:
            # Log failed attempt
            await audit_logger.log_event(
                event_type="FAILED_LOGIN",
                ip_address=client_ip,
                user_agent=user_agent,
                success=False,
                error_message=auth_result.error,
                metadata={"credentials_provided": list(credentials.keys())}
            )
            
            return AuthenticationResponse(
                success=False,
                error=auth_result.error
            )
        
        # Create session
        session_token = await session_manager.create_session(
            auth_result.user_data,
            device_id=request.device_id
        )
        
        # Log successful login
        await audit_logger.log_event(
            event_type="LOGIN",
            user_id=str(auth_result.user_data.get('id')),
            session_id=session_token.metadata.get('session_id'),
            ip_address=client_ip,
            user_agent=user_agent,
            success=True,
            metadata={
                "auth_method": auth_result.metadata.get("auth_method"),
                "device_id": request.device_id
            }
        )
        
        return AuthenticationResponse(
            success=True,
            token=session_token.token,
            user_data=auth_result.user_data,
            expires_at=session_token.expires_at.isoformat()
        )
        
    except Exception as e:
        logger.error("Authentication error", error=str(e))
        await audit_logger.log_event(
            event_type="AUTHENTICATION_ERROR",
            ip_address=client_ip,
            user_agent=user_agent,
            success=False,
            error_message=str(e)
        )
        
        raise HTTPException(
            status_code=500,
            detail="Internal authentication error"
        )

@router.get("/validate", response_model=SessionValidationResponse)
async def validate_session(
    credentials = Depends(security),
    session_manager = Depends(get_session_manager)
):
    """
    Validate session token and return user information
    """
    token = credentials.credentials
    
    session_info = await session_manager.validate_session(token)
    
    if not session_info:
        return SessionValidationResponse(valid=False)
    
    return SessionValidationResponse(
        valid=True,
        user_data=session_info['user_data'],
        expires_at=session_info['expires_at'],
        session_id=session_info['session_id']
    )

@router.post("/logout", response_model=LogoutResponse)
async def logout(
    http_request: Request,
    credentials = Depends(security),
    session_manager = Depends(get_session_manager),
    audit_logger = Depends(get_audit_logger)
):
    """
    Logout user and invalidate session
    """
    token = credentials.credentials
    client_ip = http_request.client.host
    
    # Validate session first to get user info for logging
    session_info = await session_manager.validate_session(token)
    
    success = await session_manager.invalidate_session(token)
    
    if success and session_info:
        await audit_logger.log_event(
            event_type="LOGOUT",
            user_id=session_info['user_data'].get('id'),
            session_id=session_info['session_id'],
            ip_address=client_ip,
            success=True
        )
    
    return LogoutResponse(
        success=success,
        message="Logged out successfully" if success else "Session not found"
    )

@router.post("/refresh", response_model=AuthenticationResponse)
async def refresh_session(
    credentials = Depends(security),
    session_manager = Depends(get_session_manager)
):
    """
    Refresh session token
    """
    token = credentials.credentials
    
    new_session = await session_manager.refresh_session(token)
    
    if not new_session:
        return AuthenticationResponse(
            success=False,
            error="Cannot refresh session"
        )
    
    return AuthenticationResponse(
        success=True,
        token=new_session.token,
        expires_at=new_session.expires_at.isoformat()
    )
```

### Configuration API

```python
# api/v1/config.py
from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel
from typing import Dict, Any, List, Optional

class ConfigurationRequest(BaseModel):
    # Note: In a real implementation, sensitive fields like host_db_url 
    # should be encrypted client-side before being sent to this API.
    data_source_type: str = "database" # 'database' or 'api'
    auth_type: str
    username_field: Optional[str] = "username"
    password_field: str = "password"
    email_field: Optional[str] = None
    phone_field: Optional[str] = None
    host_db_url: Optional[str] = None
    host_user_table: Optional[str] = None
    host_api_base_url: Optional[str] = None
    session_duration: int = 3600
    max_session_duration: int = 86400
    multi_device_sessions: bool = True
    middleware_stack: List[Dict[str, Any]] = []

class ConfigurationResponse(BaseModel):
    success: bool
    message: str
    validation_errors: Optional[List[str]] = None

class HealthCheckResponse(BaseModel):
    status: str
    components: Dict[str, str]
    timestamp: str

router = APIRouter(prefix="/api/v1/config", tags=["configuration"])

@router.put("/", response_model=ConfigurationResponse)
async def update_configuration(
    config_request: ConfigurationRequest,
    config_manager = Depends(get_config_manager),
    data_access_factory = Depends(get_data_access_factory)
    # Note: This endpoint must be protected by a privileged API key,
    # MFA, and IP whitelisting as per NFR-SEC-002.
):
    """
    Update UAP configuration.
    """
    try:
        # Create temporary config for validation
        temp_config = ConfigurationManager(**config_request.dict())
        
        # Test data access with new configuration
        dal = data_access_factory.create_data_access_layer(temp_config)
        
        # Validate connection
        if not await dal.health_check():
            return ConfigurationResponse(
                success=False,
                message="Cannot connect to host data source",
                validation_errors=["Database/API connection failed"]
            )
        
        # Validate schema if database connection
        if isinstance(dal, DatabaseAccessLayer):
            schema_adapter = SchemaAdapter(dal.engine, temp_config)
            await schema_adapter.initialize()
            validation_result = schema_adapter.validate_configuration()
            
            if not validation_result["valid"]:
                return ConfigurationResponse(
                    success=False,
                    message="Configuration validation failed",
                    validation_errors=validation_result["errors"]
                )
        
        # Update configuration
        await config_manager.update_configuration(temp_config)
        
        return ConfigurationResponse(
            success=True,
            message="Configuration updated successfully"
        )
        
    except Exception as e:
        return ConfigurationResponse(
            success=False,
            message="Configuration update failed",
            validation_errors=[str(e)]
        )

@router.get("/health", response_model=HealthCheckResponse)
async def health_check(
    config_manager = Depends(get_config_manager),
    data_access_layer = Depends(get_data_access_layer),
    session_manager = Depends(get_session_manager)
):
    """
    Health check for UAP service and dependencies
    """
    from datetime import datetime
    
    components = {}
    
    # Check data access layer
    try:
        db_healthy = await data_access_layer.health_check()
        components["database"] = "healthy" if db_healthy else "unhealthy"
    except Exception:
        components["database"] = "unhealthy"
    
    # Check Redis (session store)
    try:
        redis_healthy = await session_manager.redis_client.ping()
        components["redis"] = "healthy" if redis_healthy else "unhealthy"
    except Exception:
        components["redis"] = "unhealthy"
    
    # Overall status
    overall_status = "healthy" if all(
        status == "healthy" for status in components.values()
    ) else "unhealthy"
    
    return HealthCheckResponse(
        status=overall_status,
        components=components,
        timestamp=datetime.utcnow().isoformat()
    )
```

---

## Integration Patterns

### 1. SDK Integration Pattern

```python
# sdk/python/uap_client.py
import httpx
from typing import Optional, Dict, Any
import asyncio

class UAPClient:
    def __init__(self, base_url: str, api_key: Optional[str] = None):
        self.base_url = base_url.rstrip('/')
        self.client = httpx.AsyncClient(
            base_url=self.base_url,
            headers={"X-API-Key": api_key} if api_key else {}
        )
    
    async def authenticate(self, credentials: Dict[str, Any]) -> Dict[str, Any]:
        """Authenticate user with UAP"""
        response = await self.client.post("/api/v1/auth/authenticate", json=credentials)
        return response.json()
    
    async def validate_session(self, token: str) -> Dict[str, Any]:
        """Validate session token"""
        response = await self.client.get(
            "/api/v1/auth/validate",
            headers={"Authorization": f"Bearer {token}"}
        )
        return response.json()
    
    async def logout(self, token: str) -> Dict[str, Any]:
        """Logout user"""
        response = await self.client.post(
            "/api/v1/auth/logout",
            headers={"Authorization": f"Bearer {token}"}
        )
        return response.json()

# Example usage in host application
class HostAppAuth:
    def __init__(self):
        self.uap_client = UAPClient("http://uap-service:8000")
    
    async def login_user(self, username: str, password: str):
        result = await self.uap_client.authenticate({
            "username": username,
            "password": password
        })
        
        if result["success"]:
            # Store token in your preferred way (cookies, headers, etc.)
            return {
                "token": result["token"],
                "user": result["user_data"]
            }
        else:
            raise AuthenticationError(result["error"])
```

### 2. Middleware Integration Pattern

```python
# integrations/fastapi_middleware.py
from fastapi import Request, HTTPException
from starlette.middleware.base import BaseHTTPMiddleware
from typing import Callable

class UAPAuthMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, uap_client: UAPClient, excluded_paths: list = None):
        super().__init__(app)
        self.uap_client = uap_client
        self.excluded_paths = excluded_paths or ["/health", "/docs", "/openapi.json"]
    
    async def dispatch(self, request: Request, call_next: Callable):
        # Skip authentication for excluded paths
        if request.url.path in self.excluded_paths:
            return await call_next(request)
        
        # Extract token from Authorization header
        auth_header = request.headers.get("Authorization")
        if not auth_header or not auth_header.startswith("Bearer "):
            raise HTTPException(status_code=401, detail="Missing or invalid token")
        
        token = auth_header.split(" ")[1]
        
        # Validate token with UAP
        validation_result = await self.uap_client.validate_session(token)
        
        if not validation_result.get("valid"):
            raise HTTPException(status_code=401, detail="Invalid or expired token")
        
        # Add user info to request state
        request.state.user = validation_result["user_data"]
        request.state.session_id = validation_result["session_id"]
        
        return await call_next(request)

# Usage in host FastAPI application
from fastapi import FastAPI

app = FastAPI()
uap_client = UAPClient("http://uap-service:8000")

app.add_middleware(
    UAPAuthMiddleware,
    uap_client=uap_client,
    excluded_paths=["/health", "/login", "/docs"]
)
```

### 3. Framework-Specific Integrations

#### Django Integration

```python
# integrations/django/middleware.py
from django.http import JsonResponse
from django.utils.deprecation import MiddlewareMixin
import asyncio

class UAPAuthMiddleware(MiddlewareMixin):
    def __init__(self, get_response):
        self.get_response = get_response
        self.uap_client = UAPClient("http://uap-service:8000")
        super().__init__(get_response)
    
    def process_request(self, request):
        # Skip for certain paths
        excluded_paths = ['/admin/', '/health/', '/login/']
        if any(request.path.startswith(path) for path in excluded_paths):
            return None
        
        # Extract token
        auth_header = request.META.get('HTTP_AUTHORIZATION', '')
        if not auth_header.startswith('Bearer '):
            return JsonResponse({'error': 'Authentication required'}, status=401)
        
        token = auth_header.split(' ')[1]
        
        # Validate with UAP (sync wrapper for async call)
        try:
            validation_result = asyncio.run(
                self.uap_client.validate_session(token)
            )
            
            if not validation_result.get('valid'):
                return JsonResponse({'error': 'Invalid token'}, status=401)
            
            # Add user to request
            request.uap_user = validation_result['user_data']
            
        except Exception as e:
            return JsonResponse({'error': 'Authentication error'}, status=500)
        
        return None
```

#### Express.js Integration

```javascript
// integrations/nodejs/uap-middleware.js
const axios = require('axios');

class UAPClient {
    constructor(baseUrl, apiKey = null) {
        this.baseUrl = baseUrl.replace(/\/$/, '');
        this.client = axios.create({
            baseURL: this.baseUrl,
            headers: apiKey ? { 'X-API-Key': apiKey } : {}
        });
    }
    
    async validateSession(token) {
        try {
            const response = await this.client.get('/api/v1/auth/validate', {
                headers: { Authorization: `Bearer ${token}` }
            });
            return response.data;
        } catch (error) {
            return { valid: false };
        }
    }
}

function createUAPMiddleware(uapClient, options = {}) {
    const excludedPaths = options.excludedPaths || ['/health', '/login'];
    
    return async (req, res, next) => {
        // Skip excluded paths
        if (excludedPaths.some(path => req.path.startsWith(path))) {
            return next();
        }
        
        // Extract token
        const authHeader = req.headers.authorization;
        if (!authHeader || !authHeader.startsWith('Bearer ')) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        const token = authHeader.split(' ')[1];
        
        // Validate with UAP
        const validation = await uapClient.validateSession(token);
        
        if (!validation.valid) {
            return res.status(401).json({ error: 'Invalid or expired token' });
        }
        
        // Add user to request
        req.user = validation.user_data;
        req.sessionId = validation.session_id;
        
        next();
    };
}

module.exports = { UAPClient, createUAPMiddleware };

// Usage
const express = require('express');
const { UAPClient, createUAPMiddleware } = require('./uap-middleware');

const app = express();
const uapClient = new UAPClient('http://uap-service:8000');

app.use(createUAPMiddleware(uapClient, {
    excludedPaths: ['/health', '/login', '/register']
}));
```

---

## Security Architecture

### 1. Security Layers

```python
# security/middleware/security_stack.py
from typing import List, Dict, Any, Optional
from abc import ABC, abstractmethod
import time
import hashlib
from datetime import datetime, timedelta

class SecurityMiddleware(ABC):
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.enabled = config.get('enabled', True)
    
    @abstractmethod
    async def pre_authenticate(self, request_data: Dict[str, Any]) -> Dict[str, Any]:
        """Execute before authentication"""
        pass
    
    @abstractmethod
    async def post_authenticate(self, request_data: Dict[str, Any], 
                              auth_result: Dict[str, Any]) -> Dict[str, Any]:
        """Execute after successful authentication"""
        pass
    
    async def on_failure(self, request_data: Dict[str, Any], 
                        error: str) -> Dict[str, Any]:
        """Execute on authentication failure"""
        return {"action": "continue"}

class RateLimitMiddleware(SecurityMiddleware):
    def __init__(self, config: Dict[str, Any], redis_client):
        super().__init__(config)
        self.redis_client = redis_client
        self.max_attempts = config.get('max_attempts', 5)
        self.window_seconds = config.get('window_seconds', 300)
        self.block_duration = config.get('block_duration', 900)
    
    async def pre_authenticate(self, request_data: Dict[str, Any]) -> Dict[str, Any]:
        if not self.enabled:
            return {"action": "continue"}
        
        identifier = self._get_identifier(request_data)
        key = f"rate_limit:{identifier}"
        
        # Check current attempts
        current_attempts = await self.redis_client.get(key)
        
        if current_attempts:
            attempts = int(current_attempts)
            if attempts >= self.max_attempts:
                # Check if still blocked
                ttl = await self.redis_client.ttl(key)
                if ttl > 0:
                    return {
                        "action": "block",
                        "error": f"Too many attempts. Try again in {ttl} seconds.",
                        "retry_after": ttl
                    }
        
        return {"action": "continue"}
    
    async def post_authenticate(self, request_data: Dict[str, Any], 
                              auth_result: Dict[str, Any]) -> Dict[str, Any]:
        # Reset counter on successful authentication
        if auth_result.get('success'):
            identifier = self._get_identifier(request_data)
            key = f"rate_limit:{identifier}"
            await self.redis_client.delete(key)
        
        return {"action": "continue"}
    
    async def on_failure(self, request_data: Dict[str, Any], error: str) -> Dict[str, Any]:
        identifier = self._get_identifier(request_data)
        key = f"rate_limit:{identifier}"
        
        # Increment attempt counter
        pipe = self.redis_client.pipeline()
        pipe.incr(key)
        pipe.expire(key, self.block_duration)
        await pipe.execute()
        
        return {"action": "continue"}
    
    def _get_identifier(self, request_data: Dict[str, Any]) -> str:
        # Use IP address as primary identifier
        ip = request_data.get('client_ip', 'unknown')
        username = request_data.get('username', request_data.get('email', ''))
        return f"{ip}:{username}" if username else ip

class CaptchaMiddleware(SecurityMiddleware):
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.provider = config.get('provider', 'recaptcha')
        self.secret_key = config.get('secret_key')
        self.threshold = config.get('threshold', 0.5)
    
    async def pre_authenticate(self, request_data: Dict[str, Any]) -> Dict[str, Any]:
        if not self.enabled:
            return {"action": "continue"}
        
        captcha_token = request_data.get('captcha_token')
        if not captcha_token:
            return {
                "action": "block",
                "error": "CAPTCHA verification required"
            }
        
        # Verify CAPTCHA
        if self.provider == 'recaptcha':
            verification_result = await self._verify_recaptcha(captcha_token)
        else:
            verification_result = {"success": False, "error": "Unsupported CAPTCHA provider"}
        
        if not verification_result.get('success'):
            return {
                "action": "block",
                "error": verification_result.get('error', 'CAPTCHA verification failed')
            }
        
        return {"action": "continue"}
    
    async def post_authenticate(self, request_data: Dict[str, Any], 
                              auth_result: Dict[str, Any]) -> Dict[str, Any]:
        return {"action": "continue"}
    
    async def _verify_recaptcha(self, token: str) -> Dict[str, Any]:
        import httpx
        
        try:
            async with httpx.AsyncClient() as client:
                response = await client.post(
                    'https://www.google.com/recaptcha/api/siteverify',
                    data={
                        'secret': self.secret_key,
                        'response': token
                    }
                )
                
                result = response.json()
                
                if result.get('success') and result.get('score', 0) >= self.threshold:
                    return {"success": True}
                else:
                    return {"success": False, "error": "CAPTCHA verification failed"}
                    
        except Exception as e:
            return {"success": False, "error": f"CAPTCHA verification error: {str(e)}"}
```

### 2. Encryption and Token Security

```python
# security/crypto/token_manager.py
import jwt
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64
import os
from datetime import datetime, timedelta
from typing import Dict, Any, Optional

class TokenManager:
    def __init__(self, config: ConfigurationManager):
        self.config = config
        self.jwt_secret = config.jwt_secret_key.encode()
        self.jwt_algorithm = config.jwt_algorithm
        self.encryption_key = self._derive_encryption_key()
        self.cipher_suite = Fernet(self.encryption_key)
    
    def _derive_encryption_key(self) -> bytes:
        """Derive encryption key from JWT secret"""
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=b'uap_salt_2025',  # In production, use random salt
            iterations=100000,
        )
        key = base64.urlsafe_b64encode(kdf.derive(self.jwt_secret))
        return key
    
    def create_jwt_token(self, payload: Dict[str, Any], expires_delta: Optional[timedelta] = None) -> str:
        """Create JWT token with payload"""
        to_encode = payload.copy()
        
        if expires_delta:
            expire = datetime.utcnow() + expires_delta
        else:
            expire = datetime.utcnow() + timedelta(seconds=self.config.jwt_expiration)
        
        to_encode.update({"exp": expire, "iat": datetime.utcnow()})
        
        encoded_jwt = jwt.encode(to_encode, self.jwt_secret, algorithm=self.jwt_algorithm)
        return encoded_jwt
    
    def verify_jwt_token(self, token: str) -> Optional[Dict[str, Any]]:
        """Verify and decode JWT token"""
        try:
            payload = jwt.decode(token, self.jwt_secret, algorithms=[self.jwt_algorithm])
            return payload
        except jwt.ExpiredSignatureError:
            return None
        except jwt.InvalidTokenError:
            return None
    
    def encrypt_sensitive_data(self, data: str) -> str:
        """Encrypt sensitive data"""
        encrypted_data = self.cipher_suite.encrypt(data.encode())
        return base64.urlsafe_b64encode(encrypted_data).decode()
    
    def decrypt_sensitive_data(self, encrypted_data: str) -> Optional[str]:
        """Decrypt sensitive data"""
        try:
            decoded_data = base64.urlsafe_b64decode(encrypted_data.encode())
            decrypted_data = self.cipher_suite.decrypt(decoded_data)
            return decrypted_data.decode()
        except Exception:
            return None

class SecurityHeaders:
    """Security headers middleware for FastAPI"""
    
    @staticmethod
    def add_security_headers(response):
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        response.headers["Content-Security-Policy"] = "default-src 'self'"
        return response
```

### 3. Audit and Compliance

```python
# security/audit/audit_logger.py
from typing import Dict, Any, Optional
import structlog
from datetime import datetime
import json
from enum import Enum

class AuditEventType(str, Enum):
    LOGIN = "LOGIN"
    LOGOUT = "LOGOUT"
    FAILED_LOGIN = "FAILED_LOGIN"
    SESSION_EXPIRED = "SESSION_EXPIRED"
    PASSWORD_CHANGE = "PASSWORD_CHANGE"
    ACCOUNT_LOCKED = "ACCOUNT_LOCKED"
    SUSPICIOUS_ACTIVITY = "SUSPICIOUS_ACTIVITY"
    CONFIGURATION_CHANGE = "CONFIGURATION_CHANGE"
    DATA_ACCESS = "DATA_ACCESS"
    API_ACCESS = "API_ACCESS"

class AuditLogger:
    def __init__(self, config: ConfigurationManager, database_session):
        self.config = config
        self.db_session = database_session
        self.logger = structlog.get_logger("audit")
    
    async def log_event(
        self,
        event_type: AuditEventType,
        user_id: Optional[str] = None,
        session_id: Optional[str] = None,
        ip_address: Optional[str] = None,
        user_agent: Optional[str] = None,
        success: bool = True,
        error_message: Optional[str] = None,
        metadata: Optional[Dict[str, Any]] = None
    ):
        """Log audit event to database and structured logs"""
        
        audit_record = {
            "organization_id": getattr(self.config, 'organization_id', 'default'),
            "event_type": event_type,
            "user_id": user_id,
            "session_id": session_id,
            "ip_address": ip_address,
            "user_agent": user_agent,
            "success": success,
            "error_message": error_message,
            "metadata": json.dumps(metadata) if metadata else None,
            "timestamp": datetime.utcnow()
        }
        
        # Log to structured logger
        self.logger.info(
            "audit_event",
            **audit_record
        )
        
        # Store in database
        try:
            from core.models import UAPAuditLog
            
            audit_log = UAPAuditLog(**audit_record)
            self.db_session.add(audit_log)
            await self.db_session.commit()
            
        except Exception as e:
            self.logger.error("Failed to store audit log", error=str(e))
    
    async def get_audit_logs(
        self,
        user_id: Optional[str] = None,
        event_type: Optional[AuditEventType] = None,
        start_date: Optional[datetime] = None,
        end_date: Optional[datetime] = None,
        limit: int = 100
    ) -> List[Dict[str, Any]]:
        """Retrieve audit logs with filters"""
        
        from core.models import UAPAuditLog
        from sqlalchemy import and_
        
        query = self.db_session.query(UAPAuditLog)
        
        filters = []
        if user_id:
            filters.append(UAPAuditLog.user_id == user_id)
        if event_type:
            filters.append(UAPAuditLog.event_type == event_type)
        if start_date:
            filters.append(UAPAuditLog.timestamp >= start_date)
        if end_date:
            filters.append(UAPAuditLog.timestamp <= end_date)
        
        if filters:
            query = query.filter(and_(*filters))
        
        results = await query.order_by(UAPAuditLog.timestamp.desc()).limit(limit).all()
        
        return [
            {
                "id": str(result.id),
                "event_type": result.event_type,
                "user_id": result.user_id,
                "timestamp": result.timestamp.isoformat(),
                "success": result.success,
                "ip_address": result.ip_address,
                "metadata": json.loads(result.metadata) if result.metadata else {}
            }
            for result in results
        ]

class ComplianceManager:
    """Manages compliance requirements and data retention"""
    
    def __init__(self, config: ConfigurationManager):
        self.config = config
        self.retention_days = getattr(config, 'audit_retention_days', 2555)  # 7 years default
    
    async def cleanup_expired_logs(self, database_session):
        """Clean up audit logs older than retention period"""
        
        from core.models import UAPAuditLog
        from datetime import datetime, timedelta
        
        cutoff_date = datetime.utcnow() - timedelta(days=self.retention_days)
        
        await database_session.query(UAPAuditLog).filter(
            UAPAuditLog.timestamp < cutoff_date
        ).delete()
        
        await database_session.commit()
    
    async def generate_compliance_report(
        self,
        start_date: datetime,
        end_date: datetime,
        database_session
    ) -> Dict[str, Any]:
        """Generate compliance report for specified period"""
        
        from core.models import UAPAuditLog
        from sqlalchemy import func
        
        # Authentication statistics
        auth_stats = await database_session.query(
            UAPAuditLog.event_type,
            func.count(UAPAuditLog.id).label('count'),
            func.sum(UAPAuditLog.success.cast(Integer)).label('successful')
        ).filter(
            UAPAuditLog.timestamp.between(start_date, end_date)
        ).group_by(UAPAuditLog.event_type).all()
        
        # Failed login analysis
        failed_logins = await database_session.query(
            UAPAuditLog.ip_address,
            func.count(UAPAuditLog.id).label('attempts')
        ).filter(
            and_(
                UAPAuditLog.event_type == AuditEventType.FAILED_LOGIN,
                UAPAuditLog.timestamp.between(start_date, end_date)
            )
        ).group_by(UAPAuditLog.ip_address).order_by(
            func.count(UAPAuditLog.id).desc()
        ).limit(10).all()
        
        return {
            "report_period": {
                "start_date": start_date.isoformat(),
                "end_date": end_date.isoformat()
            },
            "authentication_statistics": [
                {
                    "event_type": stat.event_type,
                    "total_attempts": stat.count,
                    "successful_attempts": stat.successful,
                    "failure_rate": round((stat.count - stat.successful) / stat.count * 100, 2)
                }
                for stat in auth_stats
            ],
            "top_failed_login_sources": [
                {
                    "ip_address": login.ip_address,
                    "failed_attempts": login.attempts
                }
                for login in failed_logins
            ]
        }
```

---

## Deployment Architecture

### 1. Docker Configuration

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PYTHONPATH=/app

# Set work directory
WORKDIR /app

# Install system dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        gcc \
        libpq-dev \
        curl \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . .

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser
RUN chown -R appuser:appuser /app
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/api/v1/config/health || exit 1

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  uap-service:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - UAP_DEBUG=false
      - UAP_JWT_SECRET_KEY=${JWT_SECRET_KEY}
      - UAP_REDIS_URL=redis://redis:6379
      - UAP_DATABASE_URL=postgresql://postgres:password@postgres:5432/uap
    depends_on:
      - redis
      - postgres
    volumes:
      - ./logs:/app/logs
    networks:
      - uap-network
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - uap-network
    restart: unless-stopped
    command: redis-server --appendonly yes

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=uap
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - uap-network
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - uap-service
    networks:
      - uap-network
    restart: unless-stopped

volumes:
  redis-data:
  postgres-data:

networks:
  uap-network:
    driver: bridge
```

### 2. Kubernetes Deployment

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: uap-system
  labels:
    name: uap-system

---
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: uap-config
  namespace: uap-system
data:
  UAP_DEBUG: "false"
  UAP_SERVICE_NAME: "UAP"
  UAP_REDIS_URL: "redis://redis-service:6379"
  UAP_DATABASE_URL: "postgresql://postgres:password@postgres-service:5432/uap"

---
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: uap-secrets
  namespace: uap-system
type: Opaque
data:
  JWT_SECRET_KEY: <base64-encoded-jwt-secret>
  DB_PASSWORD: <base64-encoded-db-password>

---
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: uap-service
  namespace: uap-system
  labels:
    app: uap-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: uap-service
  template:
    metadata:
      labels:
        app: uap-service
    spec:
      containers:
      - name: uap-service
        image: uap/service:latest
        ports:
        - containerPort: 8000
        env:
        - name: UAP_JWT_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: uap-secrets
              key: JWT_SECRET_KEY
        envFrom:
        - configMapRef:
            name: uap-config
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /api/v1/config/health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/v1/config/health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5

---
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: uap-service
  namespace: uap-system
spec:
  selector:
    app: uap-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: ClusterIP

---
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: uap-ingress
  namespace: uap-system
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - uap.yourdomain.com
    secretName: uap-tls
  rules:
  - host: uap.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: uap-service
            port:
              number: 80

---
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: uap-hpa
  namespace: uap-system
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: uap-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 3. Helm Chart Structure

```yaml
# helm/uap/Chart.yaml
apiVersion: v2
name: uap
description: Universal Authentication Platform Helm Chart
type: application
version: 1.0.0
appVersion: "1.0.0"

---
# helm/uap/values.yaml
replicaCount: 3

image:
  repository: uap/service
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: ClusterIP
  port: 80
  targetPort: 8000

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: uap.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: uap-tls
      hosts:
        - uap.example.com

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

config:
  debug: false
  serviceName: "UAP"
  redisUrl: "redis://redis-service:6379"
  databaseUrl: "postgresql://postgres:password@postgres-service:5432/uap"

secrets:
  jwtSecretKey: "your-jwt-secret-key"
  dbPassword: "your-db-password"

redis:
  enabled: true
  architecture: standalone
  auth:
    enabled: false

postgresql:
  enabled: true
  auth:
    postgresPassword: "password"
    database: "uap"

monitoring:
  enabled: true
  serviceMonitor:
    enabled: true
    interval: 30s
```

---

## Performance & Scalability

### 1. Performance Optimization

```python
# performance/optimization.py
import asyncio
from functools import lru_cache
from typing import Dict, Any, Optional
import aiocache
from contextlib import asynccontextmanager

class PerformanceOptimizer:
    def __init__(self, config: ConfigurationManager):
        self.config = config
        self.cache = aiocache.Cache(aiocache.Cache.REDIS, endpoint=config.redis_url)
        self.connection_pool = None
    
    @asynccontextmanager
    async def get_db_connection(self):
        """Connection pool management"""
        if not self.connection_pool:
            self.connection_pool = await self._create_connection_pool()
        
        async with self.connection_pool.acquire() as conn:
            yield conn
    
    async def _create_connection_pool(self):
        """Create optimized connection pool"""
        import asyncpg
        
        return await asyncpg.create_pool(
            self.config.host_db_url,
            min_size=10,
            max_size=50,
            max_queries=50000,
            max_inactive_connection_lifetime=300,
            command_timeout=60
        )
    
    @aiocache.cached(ttl=300, key_builder=lambda f, self, user_id: f"user:{user_id}")
    async def get_cached_user_data(self, user_id: str) -> Optional[Dict[str, Any]]:
        """Cache user data to reduce database calls"""
        # This would typically fetch from database
        pass
    
    async def batch_validate_sessions(self, tokens: list) -> Dict[str, Dict[str, Any]]:
        """Validate multiple sessions in batch for better performance"""
        results = {}
        
        # Use asyncio.gather for concurrent validation
        validation_tasks = [
            self._validate_single_session(token) for token in tokens
        ]
        
        validation_results = await asyncio.gather(*validation_tasks, return_exceptions=True)
        
        for token, result in zip(tokens, validation_results):
            if isinstance(result, Exception):
                results[token] = {"valid": False, "error": str(result)}
            else:
                results[token] = result
        
        return results
    
    async def _validate_single_session(self, token: str) -> Dict[str, Any]:
        """Validate single session with caching"""
        # Check cache first
        cache_key = f"session_validation:{token}"
        cached_result = await self.cache.get(cache_key)
        
        if cached_result:
            return cached_result
        
        # Validate session (implementation depends on session manager)
        # result = await session_manager.validate_session(token)
        
        # Cache result for short duration
        # await self.cache.set(cache_key, result, ttl=60)
        
        # return result
        pass

class ConnectionPoolManager:
    def __init__(self):
        self.pools = {}
    
    async def get_pool(self, database_url: str, pool_name: str = "default"):
        """Get or create connection pool"""
        if pool_name not in self.pools:
            self.pools[pool_name] = await self._create_pool(database_url)
        return self.pools[pool_name]
    
    async def _create_pool(self, database_url: str):
        """Create optimized connection pool"""
        if database_url.startswith('postgresql'):
            import asyncpg
            return await asyncpg.create_pool(
                database_url,
                min_size=5,
                max_size=20,
                max_queries=50000,
                max_inactive_connection_lifetime=300
            )
        else:
            # Handle other database types
            pass
    
    async def close_all_pools(self):
        """Close all connection pools"""
        for pool in self.pools.values():
            await pool.close()
        self.pools.clear()
```

### 2. Caching Strategy

```python
# performance/caching.py
from typing import Dict, Any, Optional, Union
import json
import hashlib
from datetime import datetime, timedelta
import redis.asyncio as redis

class CacheManager:
    def __init__(self, redis_url: str):
        self.redis_client = redis.from_url(redis_url)
        self.default_ttl = 300  # 5 minutes
    
    def _generate_cache_key(self, prefix: str, **kwargs) -> str:
        """Generate consistent cache key"""
        key_data = json.dumps(kwargs, sort_keys=True)
        key_hash = hashlib.md5(key_data.encode()).hexdigest()
        return f"{prefix}:{key_hash}"
    
    async def get_cached_auth_result(self, credentials: Dict[str, Any]) -> Optional[Dict[str, Any]]:
        """Get cached authentication result"""
        # Don't cache actual authentication for security reasons
        # This is just an example - in practice, you wouldn't cache auth results
        return None
    
    async def cache_user_metadata(self, user_id: str, metadata: Dict[str, Any], ttl: int = None):
        """Cache user metadata (non-sensitive data)"""
        cache_key = f"user_metadata:{user_id}"
        ttl = ttl or self.default_ttl
        
        await self.redis_client.setex(
            cache_key,
            ttl,
            json.dumps(metadata, default=str)
        )
    
    async def get_cached_user_metadata(self, user_id: str) -> Optional[Dict[str, Any]]:
        """Get cached user metadata"""
        cache_key = f"user_metadata:{user_id}"
        cached_data = await self.redis_client.get(cache_key)
        
        if cached_data:
            return json.loads(cached_data)
        return None
    
    async def cache_configuration(self, org_id: str, config: Dict[str, Any]):
        """Cache configuration data"""
        cache_key = f"config:{org_id}"
        
        await self.redis_client.setex(
            cache_key,
            3600,  # 1 hour
            json.dumps(config, default=str)
        )
    
    async def invalidate_user_cache(self, user_id: str):
        """Invalidate all cached data for a user"""
        pattern = f"*{user_id}*"
        keys = await self.redis_client.keys(pattern)
        
        if keys:
            await self.redis_client.delete(*keys)
    
    async def get_cache_stats(self) -> Dict[str, Any]:
        """Get cache performance statistics"""
        info = await self.redis_client.info()
        
        return {
            "connected_clients": info.get("connected_clients", 0),
            "used_memory": info.get("used_memory_human", "0B"),
            "keyspace_hits": info.get("keyspace_hits", 0),
            "keyspace_misses": info.get("keyspace_misses", 0),
            "hit_rate": self._calculate_hit_rate(
                info.get("keyspace_hits", 0),
                info.get("keyspace_misses", 0)
            )
        }
    
    def _calculate_hit_rate(self, hits: int, misses: int) -> float:
        """Calculate cache hit rate"""
        total = hits + misses
        return (hits / total * 100) if total > 0 else 0.0
```

### 3. Load Balancing Configuration

```nginx
# nginx/nginx.conf
upstream uap_backend {
    least_conn;
    server uap-service-1:8000 max_fails=3 fail_timeout=30s;
    server uap-service-2:8000 max_fails=3 fail_timeout=30s;
    server uap-service-3:8000 max_fails=3 fail_timeout=30s;
    
    # Health check
    keepalive 32;
}

server {
    listen 80;
    listen 443 ssl http2;
    server_name uap.example.com;
    
    # SSL configuration
    ssl_certificate /etc/nginx/ssl/uap.crt;
    ssl_certificate_key /etc/nginx/ssl/uap.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=auth:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=api:10m rate=100r/s;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self'" always;
    
    # Authentication endpoints - stricter rate limiting
    location /api/v1/auth/ {
        limit_req zone=auth burst=20 nodelay;
        
        proxy_pass http://uap_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeout settings
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 10s;
    }
    
    # Other API endpoints
    location /api/ {
        limit_req zone=api burst=200 nodelay;
        
        proxy_pass http://uap_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # Health check endpoint (no rate limiting)
    location /health {
        access_log off;
        proxy_pass http://uap_backend;
    }
}
```

---

## Monitoring & Observability

### 1. Metrics Collection

```python
# monitoring/metrics.py
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import time
from functools import wraps
from typing import Callable

# Define metrics
AUTHENTICATION_REQUESTS = Counter(
    'uap_authentication_requests_total',
    'Total authentication requests',
    ['method', 'status']
)

AUTHENTICATION_DURATION = Histogram(
    'uap_authentication_duration_seconds',
    'Authentication request duration',
    ['method']
)

ACTIVE_SESSIONS = Gauge(
    'uap_active_sessions',
    'Number of active sessions'
)

MIDDLEWARE_EXECUTION_TIME = Histogram(
    'uap_middleware_execution_seconds',
    'Middleware execution time',
    ['middleware_name']
)

DATABASE_CONNECTIONS = Gauge(
    'uap_database_connections_active',
    'Active database connections'
)

CACHE_HIT_RATE = Gauge(
    'uap_cache_hit_rate',
    'Cache hit rate percentage'
)

class MetricsCollector:
    def __init__(self):
        self.start_time = time.time()
    
    def record_authentication_request(self, method: str, success: bool, duration: float):
        """Record authentication request metrics"""
        status = 'success' if success else 'failure'
        AUTHENTICATION_REQUESTS.labels(method=method, status=status).inc()
        AUTHENTICATION_DURATION.labels(method=method).observe(duration)
    
    def update_active_sessions(self, count: int):
        """Update active sessions gauge"""
        ACTIVE_SESSIONS.set(count)
    
    def record_middleware_execution(self, middleware_name: str, duration: float):
        """Record middleware execution time"""
        MIDDLEWARE_EXECUTION_TIME.labels(middleware_name=middleware_name).observe(duration)
    
    def update_database_connections(self, count: int):
        """Update database connections gauge"""
        DATABASE_CONNECTIONS.set(count)
    
    def update_cache_hit_rate(self, hit_rate: float):
        """Update cache hit rate"""
        CACHE_HIT_RATE.set(hit_rate)

def metrics_middleware(metrics_collector: MetricsCollector):
    """FastAPI middleware for metrics collection"""
    def middleware(request: Request, call_next: Callable):
        start_time = time.time()
        
        response = await call_next(request)
        
        duration = time.time() - start_time
        method = request.url.path
        success = response.status_code < 400
        
        # Record metrics based on endpoint
        if '/auth/' in method:
            metrics_collector.record_authentication_request(
                method=method,
                success=success,
                duration=duration
            )
        
        return response
    
    return middleware

# Metrics endpoint
async def get_metrics():
    """Prometheus metrics endpoint"""
    return Response(generate_latest(), media_type="text/plain")
```

### 2. Structured Logging

```python
# monitoring/logging.py
import structlog
import logging
import sys
from typing import Dict, Any
from datetime import datetime

def configure_logging(debug: bool = False, log_format: str = "json"):
    """Configure structured logging"""
    
    # Configure stdlib logging
    logging.basicConfig(
        format="%(message)s",
        stream=sys.stdout,
        level=logging.DEBUG if debug else logging.INFO,
    )
    
    # Configure structlog
    processors = [
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.stdlib.PositionalArgumentsFormatter(),
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
        structlog.processors.format_exc_info,
    ]
    
    if log_format == "json":
        processors.append(structlog.processors.JSONRenderer())
    else:
        processors.extend([
            structlog.dev.ConsoleRenderer(colors=debug)
        ])
    
    structlog.configure(
        processors=processors,
        context_class=dict,
        logger_factory=structlog.stdlib.LoggerFactory(),
        wrapper_class=structlog.stdlib.BoundLogger,
        cache_logger_on_first_use=True,
    )

class UAP_Logger:
    def __init__(self, name: str):
        self.logger = structlog.get_logger(name)
    
    def log_authentication_attempt(self, 
                                 user_id: str = None,
                                 ip_address: str = None,
                                 user_agent: str = None,
                                 success: bool = False,
                                 error: str = None,
                                 metadata: Dict[str, Any] = None):
        """Log authentication attempt"""
        log_data = {
            "event_type": "authentication_attempt",
            "user_id": user_id,
            "ip_address": ip_address,
            "user_agent": user_agent,
            "success": success,
            "error": error,
            "timestamp": datetime.utcnow().isoformat(),
        }
        
        if metadata:
            log_data.update(metadata)
        
        if success:
            self.logger.info("Authentication successful", **log_data)
        else:
            self.logger.warning("Authentication failed", **log_data)
    
    def log_session_event(self,
                         event_type: str,
                         session_id: str,
                         user_id: str = None,
                         metadata: Dict[str, Any] = None):
        """Log session-related events"""
        log_data = {
            "event_type": f"session_{event_type}",
            "session_id": session_id,
            "user_id": user_id,
            "timestamp": datetime.utcnow().isoformat(),
        }
        
        if metadata:
            log_data.update(metadata)
        
        self.logger.info(f"Session {event_type}", **log_data)
    
    def log_security_event(self,
                          event_type: str,
                          severity: str = "medium",
                          ip_address: str = None,
                          user_id: str = None,
                          description: str = None,
                          metadata: Dict[str, Any] = None):
        """Log security-related events"""
        log_data = {
            "event_type": f"security_{event_type}",
            "severity": severity,
            "ip_address": ip_address,
            "user_id": user_id,
            "description": description,
            "timestamp": datetime.utcnow().isoformat(),
        }
        
        if metadata:
            log_data.update(metadata)
        
        if severity == "high":
            self.logger.error(f"Security event: {event_type}", **log_data)
        elif severity == "medium":
            self.logger.warning(f"Security event: {event_type}", **log_data)
        else:
            self.logger.info(f"Security event: {event_type}", **log_data)
    
    def log_performance_metric(self,
                              metric_name: str,
                              value: float,
                              unit: str = "seconds",
                              metadata: Dict[str, Any] = None):
        """Log performance metrics"""
        log_data = {
            "event_type": "performance_metric",
            "metric_name": metric_name,
            "value": value,
            "unit": unit,
            "timestamp": datetime.utcnow().isoformat(),
        }
        
        if metadata:
            log_data.update(metadata)
        
        self.logger.info(f"Performance metric: {metric_name}", **log_data)
```

### 3. Health Checks and Monitoring

```python
# monitoring/health.py
from typing import Dict, Any, List
from datetime import datetime, timedelta
import asyncio
import httpx

class HealthChecker:
    def __init__(self, config: ConfigurationManager):
        self.config = config
        self.checks = []
    
    def add_health_check(self, name: str, check_func, timeout: int = 30):
        """Add a health check"""
        self.checks.append({
            "name": name,
            "check_func": check_func,
            "timeout": timeout
        })
    
    async def run_all_checks(self) -> Dict[str, Any]:
        """Run all health checks"""
        results = {}
        overall_healthy = True
        
        for check in self.checks:
            try:
                result = await asyncio.wait_for(
                    check["check_func"](),
                    timeout=check["timeout"]
                )
                results[check["name"]] = {
                    "status": "healthy" if result else "unhealthy",
                    "checked_at": datetime.utcnow().isoformat()
                }
                if not result:
                    overall_healthy = False
            except asyncio.TimeoutError:
                results[check["name"]] = {
                    "status": "timeout",
                    "checked_at": datetime.utcnow().isoformat()
                }
                overall_healthy = False
            except Exception as e:
                results[check["name"]] = {
                    "status": "error",
                    "error": str(e),
                    "checked_at": datetime.utcnow().isoformat()
                }
                overall_healthy = False
        
        return {
            "status": "healthy" if overall_healthy else "unhealthy",
            "checks": results,
            "timestamp": datetime.utcnow().isoformat()
        }

async def check_database_connection(data_access_layer) -> bool:
    """Check database connectivity"""
    try:
        return await data_access_layer.health_check()
    except Exception:
        return False

async def check_redis_connection(redis_client) -> bool:
    """Check Redis connectivity"""
    try:
        await redis_client.ping()
        return True
    except Exception:
        return False

async def check_external_api(api_url: str, timeout: int = 10) -> bool:
    """Check external API connectivity"""
    try:
        async with httpx.AsyncClient(timeout=timeout) as client:
            response = await client.get(f"{api_url}/health")
            return response.status_code == 200
    except Exception:
        return False

class AlertManager:
    def __init__(self, config: ConfigurationManager):
        self.config = config
        self.alert_thresholds = {
            "failed_auth_rate": 10,  # per minute
            "response_time_p95": 1000,  # milliseconds
            "error_rate": 5,  # percentage
            "active_sessions": 10000
        }
    
    async def check_alerts(self, metrics: Dict[str, Any]) -> List[Dict[str, Any]]:
        """Check for alert conditions"""
        alerts = []
        
        # Check failed authentication rate
        failed_auth_rate = metrics.get("failed_auth_rate", 0)
        if failed_auth_rate > self.alert_thresholds["failed_auth_rate"]:
            alerts.append({
                "type": "high_failed_auth_rate",
                "severity": "medium",
                "message": f"High failed authentication rate: {failed_auth_rate}/min",
                "value": failed_auth_rate,
                "threshold": self.alert_thresholds["failed_auth_rate"]
            })
        
        # Check response time
        response_time = metrics.get("response_time_p95", 0)
        if response_time > self.alert_thresholds["response_time_p95"]:
            alerts.append({
                "type": "high_response_time",
                "severity": "medium",
                "message": f"High response time (95th percentile): {response_time}ms",
                "value": response_time,
                "threshold": self.alert_thresholds["response_time_p95"]
            })
        
        # Check error rate
        error_rate = metrics.get("error_rate", 0)
        if error_rate > self.alert_thresholds["error_rate"]:
            alerts.append({
                "type": "high_error_rate",
                "severity": "high" if error_rate > 15 else "medium",
                "message": f"High error rate: {error_rate}%",
                "value": error_rate,
                "threshold": self.alert_thresholds["error_rate"]
            })
        
        return alerts
    
    async def send_alert(self, alert: Dict[str, Any]):
        """Send alert notification"""
        # Implementation would depend on notification system
        # (Slack, email, PagerDuty, etc.)
        logger = structlog.get_logger("alerts")
        logger.warning("Alert triggered", **alert)
```

---

## Development Guidelines

### 1. Code Structure and Standards

```python
# Project structure
"""
uap/
├── core/
│   ├── __init__.py
│   ├── auth/
│   │   ├── __init__.py
│   │   ├── engine.py
│   │   └── validators.py
│   ├── config/
│   │   ├── __init__.py
│   │   └── manager.py
│   ├── data/
│   │   ├── __init__.py
│   │   ├── access_layer.py
│   │   └── models.py
│   ├── session/
│   │   ├── __init__.py
│   │   └── manager.py
│   └── middleware/
│       ├── __init__.py
│       ├── sdk.py
│       ├── base.py
│       ├── rate_limit.py
│       ├── captcha.py
│       └── custom_loader.py
├── api/
│   ├── __init__.py
│   ├── v1/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── config.py
│   │   └── health.py
│   └── dependencies.py
├── security/
│   ├── __init__.py
│   ├── crypto/
│   │   ├── __init__.py
│   │   └── token_manager.py
│   ├── audit/
│   │   ├── __init__.py
│   │   └── audit_logger.py
│   └── middleware/
│       ├── __init__.py
│       └── security_stack.py
├── integrations/
│   ├── __init__.py
│   ├── fastapi/
│   ├── django/
│   ├── flask/
│   └── nodejs/
├── monitoring/
│   ├── __init__.py
│   ├── metrics.py
│   ├── logging.py
│   └── health.py
├── performance/
│   ├── __init__.py
│   ├── optimization.py
│   └── caching.py
├── tests/
│   ├── __init__.py
│   ├── unit/
│   ├── integration/
│   └── performance/
├── scripts/
│   ├── setup.py
│   ├── migrate.py
│   └── seed_data.py
├── deployment/
│   ├── docker/
│   ├── k8s/
│   └── helm/
├── docs/
│   ├── api/
│   ├── integration/
│   └── deployment/
├── requirements/
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── main.py
├── Dockerfile
├── docker-compose.yml
└── README.md
"""

# Code standards and conventions
class CodingStandards:
    """
    UAP Coding Standards and Conventions
    
    1. Python Code Style:
       - Follow PEP 8 with line length of 88 characters (Black default)
       - Use type hints for all function parameters and return values
       - Use descriptive variable and function names
       - Add docstrings for all classes and public methods
    
    2. Async/Await:
       - Use async/await for all I/O operations
       - Avoid blocking operations in async contexts
       - Use asyncio.gather() for concurrent operations
    
    3. Error Handling:
       - Use custom exceptions for specific error types
       - Log errors with appropriate context
       - Return structured error responses
    
    4. Security:
       - Never log sensitive data (passwords, tokens)
       - Validate all input data
       - Use parameterized queries for database operations
       - Implement proper rate limiting
    
    5. Testing:
       - Write unit tests for all business logic
       - Use integration tests for API endpoints
       - Mock external dependencies in tests
       - Aim for >90% code coverage
    
    6. Documentation:
       - Document all public APIs with OpenAPI/Swagger
       - Include usage examples in docstrings
       - Maintain up-to-date README files
    """
    pass
```

### 2. Testing Framework

```python
# tests/conftest.py
import pytest
import asyncio
from httpx import AsyncClient
from fastapi.testclient import TestClient
import redis.asyncio as redis
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

# Test configuration
@pytest.fixture
def test_config():
    from core.config.manager import ConfigurationManager
    
    return ConfigurationManager(
        auth_type="username_password",
        username_field="username",
        password_field="password",
        host_db_url="sqlite+aiosqlite:///./test.db",
        host_user_table="test_users",
        jwt_secret_key="test-secret-key",
        redis_url="redis://localhost:6379/1",  # Use test database
        debug=True
    )

@pytest.fixture
async def test_db_engine(test_config):
    engine = create_async_engine(test_config.host_db_url, echo=True)
    yield engine
    await engine.dispose()

@pytest.fixture
async def test_db_session(test_db_engine):
    async_session = sessionmaker(
        test_db_engine, class_=AsyncSession, expire_on_commit=False
    )
    
    async with async_session() as session:
        yield session

@pytest.fixture
async def test_redis():
    redis_client = redis.from_url("redis://localhost:6379/1")
    yield redis_client
    await redis_client.flushdb()  # Clean test database
    await redis_client.close()

@pytest.fixture
async def test_client():
    from main import app
    
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

# Test data fixtures
@pytest.fixture
def sample_user_data():
    return {
        "id": 1,
        "username": "testuser",
        "email": "test@example.com",
        "password": "$2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewdBPj1z8.KjQ/b2",  # "password123"
        "role": "user",
        "is_active": True
    }

@pytest.fixture
def sample_credentials():
    return {
        "username": "testuser",
        "password": "password123"
    }

# tests/unit/test_auth_engine.py
import pytest
from unittest.mock import AsyncMock, MagicMock
from core.auth.engine import AuthenticationEngine, AuthenticationResult

@pytest.mark.asyncio
async def test_successful_authentication(test_config, sample_user_data):
    # Mock data access layer
    mock_dal = AsyncMock()
    mock_dal.get_user_by_auth_field.return_value = sample_user_data
    
    # Create auth engine
    auth_engine = AuthenticationEngine(test_config, mock_dal)
    
    # Test authentication
    credentials = {"username": "testuser", "password": "password123"}
    result = await auth_engine.authenticate(credentials)
    
    assert result.success is True
    assert result.user_data["username"] == "testuser"
    assert "password" not in result.user_data  # Sensitive data removed

@pytest.mark.asyncio
async def test_failed_authentication_wrong_password(test_config, sample_user_data):
    mock_dal = AsyncMock()
    mock_dal.get_user_by_auth_field.return_value = sample_user_data
    
    auth_engine = AuthenticationEngine(test_config, mock_dal)
    
    credentials = {"username": "testuser", "password": "wrongpassword"}
    result = await auth_engine.authenticate(credentials)
    
    assert result.success is False
    assert "Invalid credentials" in result.error

@pytest.mark.asyncio
async def test_failed_authentication_user_not_found(test_config):
    mock_dal = AsyncMock()
    mock_dal.get_user_by_auth_field.return_value = None
    
    auth_engine = AuthenticationEngine(test_config, mock_dal)
    
    credentials = {"username": "nonexistent", "password": "password123"}
    result = await auth_engine.authenticate(credentials)
    
    assert result.success is False
    assert "User not found" in result.error

# tests/integration/test_auth_api.py
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_authentication_endpoint(test_client: AsyncClient, sample_credentials):
    """Test authentication API endpoint"""
    
    response = await test_client.post("/api/v1/auth/authenticate", json=sample_credentials)
    
    assert response.status_code == 200
    data = response.json()
    
    if data["success"]:
        assert "token" in data
        assert "user_data" in data
        assert "expires_at" in data
    else:
        assert "error" in data

@pytest.mark.asyncio
async def test_session_validation_endpoint(test_client: AsyncClient):
    """Test session validation API endpoint"""
    
    # First authenticate to get a token
    auth_response = await test_client.post("/api/v1/auth/authenticate", json={
        "username": "testuser",
        "password": "password123"
    })
    
    if auth_response.json().get("success"):
        token = auth_response.json()["token"]
        
        # Validate the session
        response = await test_client.get(
            "/api/v1/auth/validate",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        assert response.status_code == 200
        data = response.json()
        assert data["valid"] is True

@pytest.mark.asyncio
async def test_logout_endpoint(test_client: AsyncClient):
    """Test logout API endpoint"""
    
    # First authenticate
    auth_response = await test_client.post("/api/v1/auth/authenticate", json={
        "username": "testuser",
        "password": "password123"
    })
    
    if auth_response.json().get("success"):
        token = auth_response.json()["token"]
        
        # Logout
        response = await test_client.post(
            "/api/v1/auth/logout",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        assert response.status_code == 200
        data = response.json()
        assert data["success"] is True

# tests/performance/test_load.py
import pytest
import asyncio
import time
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_concurrent_authentication_requests():
    """Test concurrent authentication performance"""
    
    async def single_auth_request(client: AsyncClient):
        start_time = time.time()
        response = await client.post("/api/v1/auth/authenticate", json={
            "username": "testuser",
            "password": "password123"
        })
        end_time = time.time()
        return {
            "status_code": response.status_code,
            "response_time": end_time - start_time
        }
    
    # Create multiple concurrent requests
    async with AsyncClient(base_url="http://test") as client:
        tasks = [single_auth_request(client) for _ in range(100)]
        results = await asyncio.gather(*tasks)
    
    # Analyze results
    successful_requests = [r for r in results if r["status_code"] == 200]
    response_times = [r["response_time"] for r in successful_requests]
    
    assert len(successful_requests) > 0
    assert max(response_times) < 5.0  # Max 5 seconds response time
    assert sum(response_times) / len(response_times) < 1.0  # Average under 1 second
```

### 3. CI/CD Pipeline

```yaml
# .github/workflows/ci-cd.yml
name: UAP CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  PYTHON_VERSION: 3.11
  NODE_VERSION: 18

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_uap
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
    
    - name: Cache Python dependencies
      uses: actions/cache@v3
      with:
        path: ~/.cache/pip
        key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
        restore-keys: |
          ${{ runner.os }}-pip-
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements/dev.txt
    
    - name: Run code formatting check
      run: |
        black --check .
        isort --check-only .
    
    - name: Run linting
      run: |
        flake8 .
        mypy .
    
    - name: Run security checks
      run: |
        bandit -r core/ api/ security/
        safety check
    
    - name: Run unit tests
      run: |
        pytest tests/unit/ -v --cov=core --cov=api --cov-report=xml
      env:
        UAP_DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_uap
        UAP_REDIS_URL: redis://localhost:6379/1
    
    - name: Run integration tests
      run: |
        pytest tests/integration/ -v
      env:
        UAP_DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_uap
        UAP_REDIS_URL: redis://localhost:6379/1
    
    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml
        fail_ci_if_error: true

  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Log in to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: ${{ github.event_name != 'pull_request' }}
        tags: |
          ghcr.io/${{ github.repository }}:latest
          ghcr.io/${{ github.repository }}:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to staging
      run: |
        # Deployment script would go here
        echo "Deploying to staging environment"
    
    - name: Run smoke tests
      run: |
        # Smoke tests after deployment
        echo "Running smoke tests"
    
    - name: Deploy to production
      if: success()
      run: |
        # Production deployment script
        echo "Deploying to production environment"
```

---

## Conclusion

This Technical Architecture Design document provides a comprehensive blueprint for implementing the Universal Authentication Platform (UAP) using Python 3.9+ and FastAPI. The architecture emphasizes:

### Key Architectural Strengths

1. **Microservice Design**: Clean separation of concerns with well-defined interfaces
2. **Zero-Migration Integration**: Seamless integration with existing systems without data migration
3. **Extensible Middleware**: Plugin-based architecture for custom authentication workflows
4. **Enterprise Security**: Comprehensive security measures with audit logging and compliance features
5. **High Performance**: Optimized for scalability with caching, connection pooling, and async operations
6. **Production Ready**: Complete deployment, monitoring, and operational procedures

### Next Implementation Steps

1. **Core Development**: Implement authentication engine, session management, and configuration systems
2. **Security Implementation**: Develop middleware stack, encryption, and audit systems
3. **Integration Testing**: Build and test framework-specific integrations
4. **Performance Optimization**: Implement caching, connection pooling, and monitoring
5. **Documentation**: Create comprehensive API documentation and integration guides
6. **Deployment**: Set up CI/CD pipelines and deployment infrastructure

---

This architecture provides a solid foundation for building a production-ready authentication platform that can scale to support millions of authentication requests while maintaining security, reliability, and ease of integration.
