# Python Protocol Example

## Build Plan
Module B (JWT manager), Module D (auth API) depends on B

## Generated Interface

```python
# interfaces/token_manager.py

"""
Token Manager Interface

Module B (auth/jwt.py) implements this protocol.
Module D (api/auth.py) depends on this protocol.
"""

from typing import Protocol, Dict, Optional, TypedDict
from datetime import timedelta

class TokenPayload(TypedDict):
    """Token payload structure."""
    user_id: str
    email: str
    exp: int  # Expiration timestamp

class TokenManager(Protocol):
    """Interface for JWT token management."""

    def generate_token(
        self,
        user_id: str,
        email: str,
        expires_in: timedelta = timedelta(hours=24)
    ) -> str:
        """
        Generate JWT token for user.

        Args:
            user_id: Unique user identifier
            email: User email address
            expires_in: Token expiration duration

        Returns:
            Signed JWT token string

        Example:
            token = manager.generate_token('user123', 'user@example.com')
        """
        ...

    def verify_token(self, token: str) -> TokenPayload:
        """
        Verify and decode JWT token.

        Args:
            token: JWT token string

        Returns:
            Decoded token payload

        Raises:
            InvalidTokenError: If token is invalid or expired
        """
        ...

    def refresh_token(self, token: str) -> str:
        """
        Generate new token from existing valid token.

        Args:
            token: Current valid token

        Returns:
            New JWT token with extended expiration

        Raises:
            InvalidTokenError: If token is invalid or expired
        """
        ...

class InvalidTokenError(Exception):
    """Raised when token is invalid or expired."""
    pass
```

## Usage by Module D Builder

```python
# Module D uses the protocol
from interfaces.token_manager import TokenManager, InvalidTokenError

class AuthAPI:
    def __init__(self, token_manager: TokenManager):
        self.token_manager = token_manager

    def login(self, user_id: str, email: str) -> dict:
        # Use interface - implementation comes from Module B
        token = self.token_manager.generate_token(user_id, email)
        return {"access_token": token, "token_type": "bearer"}

    def validate_request(self, token: str) -> dict:
        try:
            payload = self.token_manager.verify_token(token)
            return {"user_id": payload["user_id"]}
        except InvalidTokenError:
            raise HTTPException(401, "Invalid token")
```

## Implementation by Module B Builder

```python
# Module B implements the protocol
from interfaces.token_manager import TokenManager, TokenPayload, InvalidTokenError
import jwt
from datetime import datetime, timedelta

class JWTTokenManager:
    """Concrete implementation of TokenManager protocol."""

    def __init__(self, secret_key: str):
        self.secret_key = secret_key

    def generate_token(
        self,
        user_id: str,
        email: str,
        expires_in: timedelta = timedelta(hours=24)
    ) -> str:
        """Generate JWT token."""
        payload = {
            "user_id": user_id,
            "email": email,
            "exp": datetime.utcnow() + expires_in
        }
        return jwt.encode(payload, self.secret_key, algorithm="HS256")

    def verify_token(self, token: str) -> TokenPayload:
        """Verify and decode token."""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=["HS256"])
            return TokenPayload(
                user_id=payload["user_id"],
                email=payload["email"],
                exp=payload["exp"]
            )
        except jwt.ExpiredSignatureError:
            raise InvalidTokenError("Token has expired")
        except jwt.InvalidTokenError as e:
            raise InvalidTokenError(f"Invalid token: {str(e)}")

    def refresh_token(self, token: str) -> str:
        """Generate new token from existing token."""
        payload = self.verify_token(token)
        # Generate new token with same user info
        return self.generate_token(payload["user_id"], payload["email"])
```

## Benefits of Protocol Pattern

1. **Type checking**: MyPy can verify Module D uses TokenManager correctly
2. **No runtime overhead**: Protocols are purely for type checking
3. **Duck typing**: Any class matching the protocol works
4. **Clear contracts**: Module builders know exactly what to implement
5. **Parallel development**: Module D and B can develop simultaneously
