# Security in Distributed Systems

## Overview

Security in distributed systems is a complex challenge that requires protecting multiple components, communication channels, and data stores across different networks and environments. Unlike monolithic applications, distributed systems have a larger attack surface and more potential points of failure. This section covers authentication mechanisms, access control, attack vectors, and security best practices for distributed architectures.

## Authentication Mechanisms

Authentication is the process of verifying the identity of users, services, or systems attempting to access resources.

### Single Sign-On (SSO)

SSO allows users to authenticate once and access multiple applications without re-entering credentials.

#### SSO Protocols:

##### 1. SAML (Security Assertion Markup Language)
```xml
<!-- SAML Assertion Example -->
<saml:Assertion xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
  <saml:Subject>
    <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
      user@example.com
    </saml:NameID>
  </saml:Subject>
  <saml:AttributeStatement>
    <saml:Attribute Name="Role">
      <saml:AttributeValue>Admin</saml:AttributeValue>
    </saml:Attribute>
    <saml:Attribute Name="Department">
      <saml:AttributeValue>Engineering</saml:AttributeValue>
    </saml:Attribute>
  </saml:AttributeStatement>
</saml:Assertion>
```

##### 2. OAuth 2.0 / OpenID Connect
```python
import requests
import jwt
from datetime import datetime, timedelta

class OAuthProvider:
    def __init__(self, client_id, client_secret, auth_server_url):
        self.client_id = client_id
        self.client_secret = client_secret
        self.auth_server_url = auth_server_url
        
    def get_authorization_url(self, redirect_uri, scope, state):
        """Generate authorization URL for OAuth flow"""
        params = {
            'response_type': 'code',
            'client_id': self.client_id,
            'redirect_uri': redirect_uri,
            'scope': scope,
            'state': state
        }
        
        query_string = '&'.join([f"{k}={v}" for k, v in params.items()])
        return f"{self.auth_server_url}/authorize?{query_string}"
        
    def exchange_code_for_token(self, code, redirect_uri):
        """Exchange authorization code for access token"""
        token_data = {
            'grant_type': 'authorization_code',
            'code': code,
            'redirect_uri': redirect_uri,
            'client_id': self.client_id,
            'client_secret': self.client_secret
        }
        
        response = requests.post(f"{self.auth_server_url}/token", data=token_data)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Token exchange failed: {response.text}")
            
    def validate_token(self, access_token):
        """Validate access token with authorization server"""
        headers = {'Authorization': f'Bearer {access_token}'}
        response = requests.get(f"{self.auth_server_url}/userinfo", headers=headers)
        
        if response.status_code == 200:
            return response.json()
        else:
            return None

class OpenIDConnectProvider(OAuthProvider):
    def __init__(self, client_id, client_secret, auth_server_url, jwks_uri):
        super().__init__(client_id, client_secret, auth_server_url)
        self.jwks_uri = jwks_uri
        self.jwks_cache = None
        self.jwks_cache_expiry = None
        
    def validate_id_token(self, id_token):
        """Validate OpenID Connect ID token"""
        try:
            # Get JWKS (JSON Web Key Set) for token validation
            jwks = self._get_jwks()
            
            # Decode and validate JWT
            header = jwt.get_unverified_header(id_token)
            key = self._find_key(jwks, header['kid'])
            
            payload = jwt.decode(
                id_token,
                key,
                algorithms=['RS256'],
                audience=self.client_id,
                issuer=self.auth_server_url
            )
            
            return payload
            
        except jwt.InvalidTokenError as e:
            raise Exception(f"Invalid ID token: {e}")
            
    def _get_jwks(self):
        """Get JSON Web Key Set from provider"""
        if (self.jwks_cache is None or 
            datetime.now() > self.jwks_cache_expiry):
            
            response = requests.get(self.jwks_uri)
            if response.status_code == 200:
                self.jwks_cache = response.json()
                self.jwks_cache_expiry = datetime.now() + timedelta(hours=1)
            else:
                raise Exception("Failed to fetch JWKS")
                
        return self.jwks_cache
        
    def _find_key(self, jwks, kid):
        """Find the correct key from JWKS"""
        for key in jwks['keys']:
            if key['kid'] == kid:
                return jwt.algorithms.RSAAlgorithm.from_jwk(key)
        raise Exception(f"Key {kid} not found in JWKS")

# Usage example
oauth_provider = OpenIDConnectProvider(
    client_id='your_client_id',
    client_secret='your_client_secret',
    auth_server_url='https://auth.example.com',
    jwks_uri='https://auth.example.com/.well-known/jwks.json'
)

# Generate authorization URL
auth_url = oauth_provider.get_authorization_url(
    redirect_uri='https://yourapp.com/callback',
    scope='openid profile email',
    state='random_state_value'
)

# After user authorization, exchange code for tokens
# token_response = oauth_provider.exchange_code_for_token(code, redirect_uri)
# user_info = oauth_provider.validate_id_token(token_response['id_token'])
```

### Token-Based Authentication

Token-based authentication uses cryptographically signed tokens to verify identity and carry user information.

#### JWT Implementation:
```python
import jwt
import time
import secrets
from datetime import datetime, timedelta
from typing import Dict, Any, Optional

class JWTManager:
    def __init__(self, secret_key: str, algorithm: str = 'HS256'):
        self.secret_key = secret_key
        self.algorithm = algorithm
        self.token_blacklist = set()  # Simple blacklist (use Redis in production)
        
    def generate_token(self, user_id: str, roles: list, 
                      expires_in: int = 3600, additional_claims: Dict = None) -> str:
        """Generate a JWT token"""
        now = datetime.utcnow()
        
        payload = {
            'sub': user_id,  # Subject (user ID)
            'iat': int(now.timestamp()),  # Issued at
            'exp': int((now + timedelta(seconds=expires_in)).timestamp()),  # Expiration
            'jti': secrets.token_urlsafe(16),  # JWT ID (for blacklisting)
            'roles': roles,
            'token_type': 'access'
        }
        
        if additional_claims:
            payload.update(additional_claims)
            
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)
        
    def generate_refresh_token(self, user_id: str) -> str:
        """Generate a refresh token"""
        now = datetime.utcnow()
        
        payload = {
            'sub': user_id,
            'iat': int(now.timestamp()),
            'exp': int((now + timedelta(days=30)).timestamp()),  # 30 days
            'jti': secrets.token_urlsafe(16),
            'token_type': 'refresh'
        }
        
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)
        
    def validate_token(self, token: str) -> Optional[Dict[str, Any]]:
        """Validate and decode a JWT token"""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=[self.algorithm])
            
            # Check if token is blacklisted
            if payload.get('jti') in self.token_blacklist:
                return None
                
            # Check token type
            if payload.get('token_type') != 'access':
                return None
                
            return payload
            
        except jwt.ExpiredSignatureError:
            return None
        except jwt.InvalidTokenError:
            return None
            
    def refresh_access_token(self, refresh_token: str) -> Optional[str]:
        """Generate new access token using refresh token"""
        try:
            payload = jwt.decode(refresh_token, self.secret_key, algorithms=[self.algorithm])
            
            # Validate refresh token
            if (payload.get('token_type') != 'refresh' or
                payload.get('jti') in self.token_blacklist):
                return None
                
            # Generate new access token
            user_id = payload['sub']
            # In production, fetch current user roles from database
            roles = ['user']  # Placeholder
            
            return self.generate_token(user_id, roles)
            
        except jwt.InvalidTokenError:
            return None
            
    def blacklist_token(self, token: str):
        """Add token to blacklist"""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=[self.algorithm])
            jti = payload.get('jti')
            if jti:
                self.token_blacklist.add(jti)
        except jwt.InvalidTokenError:
            pass  # Invalid token, nothing to blacklist

class TokenAuthenticationMiddleware:
    def __init__(self, jwt_manager: JWTManager):
        self.jwt_manager = jwt_manager
        
    def authenticate_request(self, request):
        """Authenticate HTTP request using JWT token"""
        auth_header = request.headers.get('Authorization')
        
        if not auth_header or not auth_header.startswith('Bearer '):
            return None
            
        token = auth_header[7:]  # Remove 'Bearer ' prefix
        payload = self.jwt_manager.validate_token(token)
        
        if payload:
            return {
                'user_id': payload['sub'],
                'roles': payload.get('roles', []),
                'token_payload': payload
            }
            
        return None
        
    def require_authentication(self, handler_func):
        """Decorator to require authentication for endpoints"""
        def wrapper(request, *args, **kwargs):
            auth_info = self.authenticate_request(request)
            
            if not auth_info:
                return create_error_response(401, 'Authentication required')
                
            # Add auth info to request
            request.auth = auth_info
            return handler_func(request, *args, **kwargs)
            
        return wrapper
        
    def require_role(self, required_role):
        """Decorator to require specific role"""
        def decorator(handler_func):
            def wrapper(request, *args, **kwargs):
                auth_info = self.authenticate_request(request)
                
                if not auth_info:
                    return create_error_response(401, 'Authentication required')
                    
                if required_role not in auth_info['roles']:
                    return create_error_response(403, 'Insufficient permissions')
                    
                request.auth = auth_info
                return handler_func(request, *args, **kwargs)
                
            return wrapper
        return decorator

# Usage example
jwt_manager = JWTManager(secret_key='your-secret-key')
auth_middleware = TokenAuthenticationMiddleware(jwt_manager)

# Generate tokens
access_token = jwt_manager.generate_token('user123', ['user', 'admin'])
refresh_token = jwt_manager.generate_refresh_token('user123')

# Validate token
payload = jwt_manager.validate_token(access_token)
print(f"Token payload: {payload}")

# Use in API endpoints
@auth_middleware.require_authentication
def protected_endpoint(request):
    user_id = request.auth['user_id']
    return f"Hello, user {user_id}!"

@auth_middleware.require_role('admin')
def admin_endpoint(request):
    return "Admin-only content"
```

### Access Control Lists (ACL) for Blob Storage

ACLs provide fine-grained access control for resources like files and objects in blob storage.

```python
from enum import Enum
from typing import Dict, List, Set
import json

class Permission(Enum):
    READ = 'read'
    WRITE = 'write'
    DELETE = 'delete'
    LIST = 'list'
    ADMIN = 'admin'

class Principal:
    def __init__(self, principal_type: str, identifier: str):
        self.type = principal_type  # 'user', 'group', 'service'
        self.identifier = identifier
        
    def __str__(self):
        return f"{self.type}:{self.identifier}"
        
    def __hash__(self):
        return hash((self.type, self.identifier))
        
    def __eq__(self, other):
        return self.type == other.type and self.identifier == other.identifier

class ACLEntry:
    def __init__(self, principal: Principal, permissions: Set[Permission], 
                 allow: bool = True):
        self.principal = principal
        self.permissions = permissions
        self.allow = allow  # True for allow, False for deny
        
    def to_dict(self):
        return {
            'principal': {
                'type': self.principal.type,
                'identifier': self.principal.identifier
            },
            'permissions': [p.value for p in self.permissions],
            'allow': self.allow
        }
        
    @classmethod
    def from_dict(cls, data):
        principal = Principal(
            data['principal']['type'],
            data['principal']['identifier']
        )
        permissions = {Permission(p) for p in data['permissions']}
        return cls(principal, permissions, data.get('allow', True))

class BlobStorageACL:
    def __init__(self, resource_path: str):
        self.resource_path = resource_path
        self.entries: List[ACLEntry] = []
        
    def add_entry(self, principal: Principal, permissions: Set[Permission], 
                  allow: bool = True):
        """Add an ACL entry"""
        entry = ACLEntry(principal, permissions, allow)
        self.entries.append(entry)
        
    def remove_entry(self, principal: Principal):
        """Remove all entries for a principal"""
        self.entries = [e for e in self.entries if e.principal != principal]
        
    def check_permission(self, principal: Principal, permission: Permission) -> bool:
        """Check if principal has permission"""
        # Check for explicit deny first
        for entry in self.entries:
            if (entry.principal == principal and 
                permission in entry.permissions and 
                not entry.allow):
                return False
                
        # Check for explicit allow
        for entry in self.entries:
            if (entry.principal == principal and 
                permission in entry.permissions and 
                entry.allow):
                return True
                
        # Check for admin permission (grants all permissions)
        for entry in self.entries:
            if (entry.principal == principal and 
                Permission.ADMIN in entry.permissions and 
                entry.allow):
                return True
                
        return False
        
    def get_permissions(self, principal: Principal) -> Set[Permission]:
        """Get all permissions for a principal"""
        permissions = set()
        denied_permissions = set()
        
        for entry in self.entries:
            if entry.principal == principal:
                if entry.allow:
                    permissions.update(entry.permissions)
                else:
                    denied_permissions.update(entry.permissions)
                    
        # Remove denied permissions
        permissions -= denied_permissions
        
        # Admin permission grants all permissions
        if Permission.ADMIN in permissions:
            return {Permission.READ, Permission.WRITE, Permission.DELETE, 
                   Permission.LIST, Permission.ADMIN}
            
        return permissions
        
    def to_json(self) -> str:
        """Serialize ACL to JSON"""
        data = {
            'resource_path': self.resource_path,
            'entries': [entry.to_dict() for entry in self.entries]
        }
        return json.dumps(data, indent=2)
        
    @classmethod
    def from_json(cls, json_str: str):
        """Deserialize ACL from JSON"""
        data = json.loads(json_str)
        acl = cls(data['resource_path'])
        
        for entry_data in data['entries']:
            entry = ACLEntry.from_dict(entry_data)
            acl.entries.append(entry)
            
        return acl

class BlobStorageACLManager:
    def __init__(self):
        self.acls: Dict[str, BlobStorageACL] = {}
        
    def create_acl(self, resource_path: str) -> BlobStorageACL:
        """Create ACL for a resource"""
        acl = BlobStorageACL(resource_path)
        self.acls[resource_path] = acl
        return acl
        
    def get_acl(self, resource_path: str) -> BlobStorageACL:
        """Get ACL for a resource"""
        return self.acls.get(resource_path)
        
    def check_access(self, resource_path: str, principal: Principal, 
                    permission: Permission) -> bool:
        """Check if principal has permission on resource"""
        acl = self.get_acl(resource_path)
        
        if not acl:
            return False  # No ACL means no access
            
        return acl.check_permission(principal, permission)
        
    def grant_permission(self, resource_path: str, principal: Principal, 
                        permissions: Set[Permission]):
        """Grant permissions to principal"""
        acl = self.get_acl(resource_path)
        if not acl:
            acl = self.create_acl(resource_path)
            
        acl.add_entry(principal, permissions, allow=True)
        
    def deny_permission(self, resource_path: str, principal: Principal, 
                       permissions: Set[Permission]):
        """Deny permissions to principal"""
        acl = self.get_acl(resource_path)
        if not acl:
            acl = self.create_acl(resource_path)
            
        acl.add_entry(principal, permissions, allow=False)

# Usage example
acl_manager = BlobStorageACLManager()

# Create principals
user1 = Principal('user', 'alice@example.com')
user2 = Principal('user', 'bob@example.com')
admin_group = Principal('group', 'administrators')

# Create ACL for a file
file_path = '/bucket/documents/secret.pdf'
acl = acl_manager.create_acl(file_path)

# Grant permissions
acl_manager.grant_permission(file_path, user1, {Permission.READ, Permission.WRITE})
acl_manager.grant_permission(file_path, user2, {Permission.READ})
acl_manager.grant_permission(file_path, admin_group, {Permission.ADMIN})

# Check permissions
can_read = acl_manager.check_access(file_path, user1, Permission.READ)
can_delete = acl_manager.check_access(file_path, user2, Permission.DELETE)

print(f"User1 can read: {can_read}")
print(f"User2 can delete: {can_delete}")

# Serialize ACL
acl_json = acl.to_json()
print(f"ACL JSON: {acl_json}")
```

## Introduction to Attack Vectors

Understanding common attack vectors helps in designing secure distributed systems.

### Common Attack Vectors in Distributed Systems

#### 1. Distributed Denial of Service (DDoS)
```python
import time
from collections import defaultdict, deque
from typing import Dict, List

class DDoSProtection:
    def __init__(self, rate_limit: int = 100, time_window: int = 60):
        self.rate_limit = rate_limit
        self.time_window = time_window
        self.request_counts = defaultdict(deque)  # IP -> deque of timestamps
        self.blocked_ips = {}  # IP -> block_until_timestamp
        
    def is_request_allowed(self, client_ip: str) -> bool:
        """Check if request from IP is allowed"""
        current_time = time.time()
        
        # Check if IP is currently blocked
        if client_ip in self.blocked_ips:
            if current_time < self.blocked_ips[client_ip]:
                return False
            else:
                # Unblock IP
                del self.blocked_ips[client_ip]
                
        # Clean old requests
        cutoff_time = current_time - self.time_window
        while (self.request_counts[client_ip] and 
               self.request_counts[client_ip][0] < cutoff_time):
            self.request_counts[client_ip].popleft()
            
        # Check rate limit
        if len(self.request_counts[client_ip]) >= self.rate_limit:
            # Block IP for increasing duration based on violations
            block_duration = min(3600, 60 * (2 ** len(self.blocked_ips)))  # Max 1 hour
            self.blocked_ips[client_ip] = current_time + block_duration
            return False
            
        # Allow request and record it
        self.request_counts[client_ip].append(current_time)
        return True
        
    def get_blocked_ips(self) -> List[str]:
        """Get list of currently blocked IPs"""
        current_time = time.time()
        return [ip for ip, block_until in self.blocked_ips.items() 
                if current_time < block_until]

class GeographicFiltering:
    def __init__(self, allowed_countries: List[str] = None, 
                 blocked_countries: List[str] = None):
        self.allowed_countries = set(allowed_countries or [])
        self.blocked_countries = set(blocked_countries or [])
        
    def is_country_allowed(self, country_code: str) -> bool:
        """Check if requests from country are allowed"""
        if self.blocked_countries and country_code in self.blocked_countries:
            return False
            
        if self.allowed_countries and country_code not in self.allowed_countries:
            return False
            
        return True
        
    def get_country_from_ip(self, ip_address: str) -> str:
        """Get country code from IP address (placeholder implementation)"""
        # In production, use a GeoIP service like MaxMind
        # This is a simplified example
        ip_to_country = {
            '192.168.1.1': 'US',
            '10.0.0.1': 'CA',
            '172.16.0.1': 'GB'
        }
        return ip_to_country.get(ip_address, 'UNKNOWN')

# Usage example
ddos_protection = DDoSProtection(rate_limit=10, time_window=60)
geo_filter = GeographicFiltering(blocked_countries=['CN', 'RU'])

def handle_request(client_ip: str):
    # Check DDoS protection
    if not ddos_protection.is_request_allowed(client_ip):
        return create_error_response(429, 'Rate limit exceeded')
        
    # Check geographic filtering
    country = geo_filter.get_country_from_ip(client_ip)
    if not geo_filter.is_country_allowed(country):
        return create_error_response(403, 'Access denied from your location')
        
    # Process request
    return process_normal_request()
```

#### 2. Man-in-the-Middle (MITM) Attacks
```python
import ssl
import socket
import hashlib
import hmac
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64

class SecureCommunication:
    def __init__(self, password: str):
        # Derive key from password
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=b'stable_salt',  # Use random salt in production
            iterations=100000,
        )
        key = base64.urlsafe_b64encode(kdf.derive(password.encode()))
        self.cipher = Fernet(key)
        
    def encrypt_message(self, message: str) -> bytes:
        """Encrypt message for secure transmission"""
        return self.cipher.encrypt(message.encode())
        
    def decrypt_message(self, encrypted_message: bytes) -> str:
        """Decrypt received message"""
        return self.cipher.decrypt(encrypted_message).decode()
        
    def create_message_signature(self, message: str, secret_key: str) -> str:
        """Create HMAC signature for message integrity"""
        signature = hmac.new(
            secret_key.encode(),
            message.encode(),
            hashlib.sha256
        ).hexdigest()
        return signature
        
    def verify_message_signature(self, message: str, signature: str, 
                                secret_key: str) -> bool:
        """Verify message signature"""
        expected_signature = self.create_message_signature(message, secret_key)
        return hmac.compare_digest(signature, expected_signature)

class TLSClient:
    def __init__(self, ca_cert_path: str = None):
        self.ca_cert_path = ca_cert_path
        
    def create_secure_connection(self, hostname: str, port: int):
        """Create TLS connection with certificate verification"""
        context = ssl.create_default_context()
        
        if self.ca_cert_path:
            context.load_verify_locations(self.ca_cert_path)
            
        # Enable certificate verification
        context.check_hostname = True
        context.verify_mode = ssl.CERT_REQUIRED
        
        # Create secure socket
        sock = socket.create_connection((hostname, port))
        secure_sock = context.wrap_socket(sock, server_hostname=hostname)
        
        return secure_sock
        
    def verify_certificate_pinning(self, hostname: str, port: int, 
                                  expected_fingerprint: str) -> bool:
        """Verify certificate pinning to prevent MITM"""
        try:
            sock = self.create_secure_connection(hostname, port)
            cert_der = sock.getpeercert(binary_form=True)
            
            # Calculate certificate fingerprint
            fingerprint = hashlib.sha256(cert_der).hexdigest()
            
            sock.close()
            return fingerprint == expected_fingerprint
            
        except Exception:
            return False

# Usage example
secure_comm = SecureCommunication('shared_secret_password')
tls_client = TLSClient()

# Encrypt and sign message
message = "Sensitive data"
encrypted_message = secure_comm.encrypt_message(message)
signature = secure_comm.create_message_signature(message, 'hmac_secret')

# Verify and decrypt on receiving end
is_valid = secure_comm.verify_message_signature(message, signature, 'hmac_secret')
if is_valid:
    decrypted_message = secure_comm.decrypt_message(encrypted_message)
    print(f"Decrypted: {decrypted_message}")
```

#### 3. SQL Injection and NoSQL Injection
```python
import re
from typing import Any, Dict, List
import pymongo
import psycopg2

class SQLInjectionProtection:
    def __init__(self):
        # Common SQL injection patterns
        self.sql_injection_patterns = [
            r"(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER)\b)",
            r"(--|#|/\*|\*/)",
            r"(\b(UNION|OR|AND)\b.*\b(SELECT|INSERT|UPDATE|DELETE)\b)",
            r"(\b(EXEC|EXECUTE)\b)",
            r"(;|\||&)",
            r"(\b(SCRIPT|JAVASCRIPT|VBSCRIPT)\b)"
        ]
        
    def is_sql_injection_attempt(self, input_string: str) -> bool:
        """Detect potential SQL injection attempts"""
        if not input_string:
            return False
            
        input_upper = input_string.upper()
        
        for pattern in self.sql_injection_patterns:
            if re.search(pattern, input_upper, re.IGNORECASE):
                return True
                
        return False
        
    def sanitize_input(self, input_string: str) -> str:
        """Basic input sanitization"""
        if not input_string:
            return ""
            
        # Remove potentially dangerous characters
        sanitized = re.sub(r"[';\"\\]", "", input_string)
        
        # Limit length
        return sanitized[:1000]
        
    def create_parameterized_query(self, base_query: str, params: Dict[str, Any]) -> tuple:
        """Create parameterized query to prevent SQL injection"""
        # Replace named parameters with placeholders
        query = base_query
        values = []
        
        for param_name, param_value in params.items():
            placeholder = f":{param_name}"
            if placeholder in query:
                query = query.replace(placeholder, "%s")
                values.append(param_value)
                
        return query, tuple(values)

class NoSQLInjectionProtection:
    def __init__(self):
        self.dangerous_operators = {
            '$where', '$regex', '$ne', '$gt', '$lt', '$gte', '$lte',
            '$in', '$nin', '$exists', '$type', '$mod', '$all', '$size'
        }
        
    def sanitize_mongodb_query(self, query: Dict[str, Any]) -> Dict[str, Any]:
        """Sanitize MongoDB query to prevent NoSQL injection"""
        return self._sanitize_dict(query)
        
    def _sanitize_dict(self, obj: Any) -> Any:
        """Recursively sanitize dictionary objects"""
        if isinstance(obj, dict):
            sanitized = {}
            for key, value in obj.items():
                # Check for dangerous operators
                if key.startswith('$') and key in self.dangerous_operators:
                    # Log potential injection attempt
                    print(f"Blocked dangerous operator: {key}")
                    continue
                    
                sanitized[key] = self._sanitize_dict(value)
            return sanitized
            
        elif isinstance(obj, list):
            return [self._sanitize_dict(item) for item in obj]
            
        elif isinstance(obj, str):
            # Prevent JavaScript injection in $where clauses
            if 'function' in obj.lower() or 'eval' in obj.lower():
                return ""
            return obj
            
        else:
            return obj

# Safe database access patterns
class SafeDatabaseAccess:
    def __init__(self, db_connection):
        self.db_connection = db_connection
        self.sql_protection = SQLInjectionProtection()
        self.nosql_protection = NoSQLInjectionProtection()
        
    def safe_sql_query(self, query: str, params: Dict[str, Any]) -> List[Dict]:
        """Execute SQL query safely with parameterization"""
        # Check for injection attempts
        for param_value in params.values():
            if isinstance(param_value, str):
                if self.sql_protection.is_sql_injection_attempt(param_value):
                    raise ValueError("Potential SQL injection detected")
                    
        # Create parameterized query
        safe_query, safe_params = self.sql_protection.create_parameterized_query(
            query, params
        )
        
        # Execute query
        cursor = self.db_connection.cursor()
        cursor.execute(safe_query, safe_params)
        return cursor.fetchall()
        
    def safe_mongodb_find(self, collection, query: Dict[str, Any]) -> List[Dict]:
        """Execute MongoDB find operation safely"""
        # Sanitize query
        safe_query = self.nosql_protection.sanitize_mongodb_query(query)
        
        # Execute query
        return list(collection.find(safe_query))

# Usage example
# sql_protection = SQLInjectionProtection()

# Detect injection attempt
# malicious_input = "'; DROP TABLE users; --"
# is_malicious = sql_protection.is_sql_injection_attempt(malicious_input)
# print(f"Is malicious: {is_malicious}")

# Safe parameterized query
# safe_query = "SELECT * FROM users WHERE username = :username AND age > :min_age"
# params = {"username": "john_doe", "min_age": 18}
# query, values = sql_protection.create_parameterized_query(safe_query, params)
```

## Video Protection inside CDNs

Protecting video content in CDNs involves multiple layers of security to prevent unauthorized access and piracy.

### Video Content Protection Strategies

#### 1. Token-Based Authentication for Video Access
```python
import jwt
import time
import hashlib
from urllib.parse import urlparse, parse_qs
from typing import Optional, Dict, Any

class VideoTokenGenerator:
    def __init__(self, secret_key: str):
        self.secret_key = secret_key
        
    def generate_video_token(self, video_id: str, user_id: str, 
                           expires_in: int = 3600, 
                           allowed_ips: List[str] = None,
                           max_views: int = None) -> str:
        """Generate secure token for video access"""
        now = int(time.time())
        
        payload = {
            'video_id': video_id,
            'user_id': user_id,
            'iat': now,
            'exp': now + expires_in,
            'jti': hashlib.md5(f"{video_id}{user_id}{now}".encode()).hexdigest()
        }
        
        if allowed_ips:
            payload['allowed_ips'] = allowed_ips
            
        if max_views:
            payload['max_views'] = max_views
            
        return jwt.encode(payload, self.secret_key, algorithm='HS256')
        
    def validate_video_token(self, token: str, client_ip: str = None) -> Optional[Dict[str, Any]]:
        """Validate video access token"""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=['HS256'])
            
            # Check IP restrictions
            if client_ip and 'allowed_ips' in payload:
                if client_ip not in payload['allowed_ips']:
                    return None
                    
            return payload
            
        except jwt.ExpiredSignatureError:
            return None
        except jwt.InvalidTokenError:
            return None

class CDNVideoProtection:
    def __init__(self, token_generator: VideoTokenGenerator):
        self.token_generator = token_generator
        self.view_counts = {}  # token_jti -> view_count
        
    def generate_signed_url(self, video_id: str, user_id: str, 
                          base_url: str, expires_in: int = 3600) -> str:
        """Generate signed URL for video access"""
        token = self.token_generator.generate_video_token(
            video_id, user_id, expires_in
        )
        
        return f"{base_url}?token={token}"
        
    def validate_video_request(self, request_url: str, client_ip: str) -> Dict[str, Any]:
        """Validate video request and return access decision"""
        parsed_url = urlparse(request_url)
        query_params = parse_qs(parsed_url.query)
        
        token = query_params.get('token', [None])[0]
        if not token:
            return {'allowed': False, 'reason': 'Missing token'}
            
        payload = self.token_generator.validate_video_token(token, client_ip)
        if not payload:
            return {'allowed': False, 'reason': 'Invalid or expired token'}
            
        # Check view count limits
        jti = payload.get('jti')
        max_views = payload.get('max_views')
        
        if max_views and jti:
            current_views = self.view_counts.get(jti, 0)
            if current_views >= max_views:
                return {'allowed': False, 'reason': 'View limit exceeded'}
                
            # Increment view count
            self.view_counts[jti] = current_views + 1
            
        return {
            'allowed': True,
            'video_id': payload['video_id'],
            'user_id': payload['user_id'],
            'remaining_views': max_views - self.view_counts.get(jti, 0) if max_views else None
        }

# Usage example
token_generator = VideoTokenGenerator('video_secret_key')
cdn_protection = CDNVideoProtection(token_generator)

# Generate signed URL for video access
signed_url = cdn_protection.generate_signed_url(
    video_id='video123',
    user_id='user456',
    base_url='https://cdn.example.com/videos/video123.mp4',
    expires_in=1800  # 30 minutes
)

print(f"Signed URL: {signed_url}")

# Validate video request
validation_result = cdn_protection.validate_video_request(
    signed_url, 
    client_ip='192.168.1.100'
)

print(f"Access allowed: {validation_result['allowed']}")
```

#### 2. Digital Rights Management (DRM)
```python
import base64
import json
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC

class DRMProtection:
    def __init__(self, master_key: str):
        # Derive encryption key from master key
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=32,
            salt=b'drm_salt_12345',  # Use random salt in production
            iterations=100000,
        )
        key = base64.urlsafe_b64encode(kdf.derive(master_key.encode()))
        self.cipher = Fernet(key)
        
    def encrypt_content_key(self, content_key: str, user_license: Dict[str, Any]) -> str:
        """Encrypt content decryption key with user license"""
        license_data = {
            'content_key': content_key,
            'license': user_license,
            'timestamp': int(time.time())
        }
        
        license_json = json.dumps(license_data)
        encrypted_license = self.cipher.encrypt(license_json.encode())
        
        return base64.b64encode(encrypted_license).decode()
        
    def decrypt_content_key(self, encrypted_license: str) -> Optional[Dict[str, Any]]:
        """Decrypt content key from encrypted license"""
        try:
            encrypted_data = base64.b64decode(encrypted_license.encode())
            decrypted_data = self.cipher.decrypt(encrypted_data)
            license_data = json.loads(decrypted_data.decode())
            
            # Validate license expiration
            license_info = license_data['license']
            if 'expires_at' in license_info:
                if time.time() > license_info['expires_at']:
                    return None
                    
            return license_data
            
        except Exception:
            return None
            
    def generate_user_license(self, user_id: str, video_id: str, 
                            permissions: List[str], expires_in: int = 86400) -> Dict[str, Any]:
        """Generate user license for content access"""
        return {
            'user_id': user_id,
            'video_id': video_id,
            'permissions': permissions,  # ['play', 'download', 'offline']
            'issued_at': int(time.time()),
            'expires_at': int(time.time()) + expires_in,
            'device_limit': 3,
            'quality_limit': '1080p'
        }

class VideoWatermarking:
    def __init__(self):
        self.watermark_positions = ['top-left', 'top-right', 'bottom-left', 'bottom-right', 'center']
        
    def generate_watermark_config(self, user_id: str, video_id: str) -> Dict[str, Any]:
        """Generate watermark configuration for video"""
        # Create unique watermark based on user and video
        watermark_text = f"User: {user_id[-6:]} | Video: {video_id[-6:]}"
        
        # Vary position based on user ID to make watermarks unique
        position_index = hash(user_id) % len(self.watermark_positions)
        
        return {
            'text': watermark_text,
            'position': self.watermark_positions[position_index],
            'opacity': 0.3,
            'font_size': 12,
            'color': 'white',
            'rotation': hash(user_id) % 15 - 7  # -7 to +7 degrees
        }
        
    def apply_forensic_watermark(self, user_id: str, video_id: str, 
                                session_id: str) -> Dict[str, Any]:
        """Apply forensic watermark for piracy tracking"""
        # Invisible watermark data
        forensic_data = {
            'user_id': user_id,
            'video_id': video_id,
            'session_id': session_id,
            'timestamp': int(time.time()),
            'client_info': 'encoded_client_fingerprint'
        }
        
        # Encode forensic data
        forensic_payload = base64.b64encode(
            json.dumps(forensic_data).encode()
        ).decode()
        
        return {
            'type': 'forensic',
            'payload': forensic_payload,
            'embedding_strength': 0.1,  # Very subtle
            'frequency_domain': True
        }

# Usage example
drm = DRMProtection('drm_master_key_12345')
watermarking = VideoWatermarking()

# Generate user license
user_license = drm.generate_user_license(
    user_id='user123',
    video_id='video456',
    permissions=['play', 'offline'],
    expires_in=86400  # 24 hours
)

# Encrypt content key with license
content_key = 'content_encryption_key_789'
encrypted_license = drm.encrypt_content_key(content_key, user_license)

# Generate watermark configuration
watermark_config = watermarking.generate_watermark_config('user123', 'video456')
forensic_watermark = watermarking.apply_forensic_watermark(
    'user123', 'video456', 'session789'
)

print(f"Encrypted license: {encrypted_license[:50]}...")
print(f"Watermark config: {watermark_config}")
print(f"Forensic watermark: {forensic_watermark}")
```

#### 3. Geo-blocking and Access Control
```python
import requests
from typing import List, Dict, Any, Optional

class GeoBlockingService:
    def __init__(self, geoip_api_key: str = None):
        self.geoip_api_key = geoip_api_key
        self.country_cache = {}  # IP -> country code cache
        
    def get_country_from_ip(self, ip_address: str) -> Optional[str]:
        """Get country code from IP address using GeoIP service"""
        if ip_address in self.country_cache:
            return self.country_cache[ip_address]
            
        try:
            # Example using a GeoIP service (replace with actual service)
            if self.geoip_api_key:
                response = requests.get(
                    f"https://api.geoip.com/v1/{ip_address}",
                    headers={'Authorization': f'Bearer {self.geoip_api_key}'},
                    timeout=5
                )
                
                if response.status_code == 200:
                    data = response.json()
                    country_code = data.get('country_code')
                    self.country_cache[ip_address] = country_code
                    return country_code
                    
        except Exception as e:
            print(f"GeoIP lookup failed: {e}")
            
        return None
        
    def is_access_allowed(self, ip_address: str, content_geo_rules: Dict[str, Any]) -> Dict[str, Any]:
        """Check if access is allowed based on geographic rules"""
        country_code = self.get_country_from_ip(ip_address)
        
        if not country_code:
            return {
                'allowed': False,
                'reason': 'Unable to determine location',
                'country': None
            }
            
        allowed_countries = content_geo_rules.get('allowed_countries', [])
        blocked_countries = content_geo_rules.get('blocked_countries', [])
        
        # Check blocked countries first
        if blocked_countries and country_code in blocked_countries:
            return {
                'allowed': False,
                'reason': f'Content blocked in {country_code}',
                'country': country_code
            }
            
        # Check allowed countries
        if allowed_countries and country_code not in allowed_countries:
            return {
                'allowed': False,
                'reason': f'Content not available in {country_code}',
                'country': country_code
            }
            
        return {
            'allowed': True,
            'reason': 'Access granted',
            'country': country_code
        }

class ContentAccessControl:
    def __init__(self, geo_blocking: GeoBlockingService):
        self.geo_blocking = geo_blocking
        self.content_rules = {}  # content_id -> access rules
        
    def set_content_rules(self, content_id: str, rules: Dict[str, Any]):
        """Set access rules for content"""
        self.content_rules[content_id] = rules
        
    def check_content_access(self, content_id: str, user_id: str, 
                           client_ip: str, user_subscription: Dict[str, Any]) -> Dict[str, Any]:
        """Comprehensive content access check"""
        rules = self.content_rules.get(content_id, {})
        
        # Check geographic restrictions
        geo_rules = rules.get('geo_restrictions', {})
        if geo_rules:
            geo_check = self.geo_blocking.is_access_allowed(client_ip, geo_rules)
            if not geo_check['allowed']:
                return geo_check
                
        # Check subscription requirements
        required_tier = rules.get('required_subscription_tier')
        if required_tier:
            user_tier = user_subscription.get('tier')
            if user_tier != required_tier:
                return {
                    'allowed': False,
                    'reason': f'Requires {required_tier} subscription',
                    'current_tier': user_tier
                }
                
        # Check age restrictions
        age_rating = rules.get('age_rating')
        if age_rating:
            user_age = user_subscription.get('age')
            if not user_age or user_age < age_rating:
                return {
                    'allowed': False,
                    'reason': f'Content rated for ages {age_rating}+',
                    'user_age': user_age
                }
                
        # Check time-based restrictions
        time_restrictions = rules.get('time_restrictions')
        if time_restrictions:
            current_time = int(time.time())
            available_from = time_restrictions.get('available_from')
            available_until = time_restrictions.get('available_until')
            
            if available_from and current_time < available_from:
                return {
                    'allowed': False,
                    'reason': 'Content not yet available',
                    'available_from': available_from
                }
                
            if available_until and current_time > available_until:
                return {
                    'allowed': False,
                    'reason': 'Content no longer available',
                    'available_until': available_until
                }
                
        return {
            'allowed': True,
            'reason': 'All access checks passed'
        }

# Usage example
geo_blocking = GeoBlockingService('your_geoip_api_key')
access_control = ContentAccessControl(geo_blocking)

# Set content rules
content_rules = {
    'geo_restrictions': {
        'allowed_countries': ['US', 'CA', 'GB'],
        'blocked_countries': []
    },
    'required_subscription_tier': 'premium',
    'age_rating': 18,
    'time_restrictions': {
        'available_from': int(time.time()) - 86400,  # Available since yesterday
        'available_until': int(time.time()) + 86400 * 30  # Available for 30 days
    }
}

access_control.set_content_rules('movie123', content_rules)

# Check access
user_subscription = {
    'tier': 'premium',
    'age': 25
}

access_result = access_control.check_content_access(
    content_id='movie123',
    user_id='user456',
    client_ip='192.168.1.100',
    user_subscription=user_subscription
)

print(f"Access result: {access_result}")
```

## Conclusion

Security in distributed systems requires a multi-layered approach addressing authentication, authorization, attack prevention, and content protection. Key takeaways:

1. **Implement Strong Authentication**: Use modern protocols like OAuth 2.0/OIDC and JWT
2. **Apply Fine-Grained Authorization**: Use ACLs and RBAC for precise access control
3. **Protect Against Common Attacks**: Implement DDoS protection, input validation, and secure communication
4. **Secure Content Delivery**: Use DRM, watermarking, and geo-blocking for valuable content
5. **Monitor and Audit**: Continuously monitor for security threats and maintain audit logs

The key is to design security into the system from the beginning rather than adding it as an afterthought.

## Next Steps

In the following sections, we'll explore:
- **Observability in Distributed Systems**: Monitoring and alerting for security events
- **HLD Interview Problems**: Applying security concepts in system design interviews