# Application Programming Interface (API)

## Overview

APIs (Application Programming Interfaces) are the contracts that define how different software components communicate with each other. They serve as the bridge between different systems, enabling data exchange and functionality sharing across applications, services, and platforms. Understanding API design, protocols, and security is essential for building scalable, maintainable distributed systems.

## What is an API?

An API is a set of rules, protocols, and tools that allows different software applications to communicate with each other. It defines the methods of communication between various software components and specifies how requests should be made, what data formats to use, and how responses should be structured.

### Key Components of an API:

#### 1. Endpoints
- **Definition**: Specific URLs where API requests are sent
- **Structure**: Usually follows a pattern like `/api/v1/users/{id}`
- **Purpose**: Each endpoint represents a specific resource or action
- **Examples**: 
  - `GET /api/users` - Retrieve all users
  - `POST /api/users` - Create a new user
  - `GET /api/users/123` - Retrieve user with ID 123

#### 2. Request Methods
- **GET**: Retrieve data from the server
- **POST**: Create new resources
- **PUT**: Update existing resources (complete replacement)
- **PATCH**: Partial update of existing resources
- **DELETE**: Remove resources from the server

#### 3. Request/Response Format
- **Headers**: Metadata about the request/response
- **Body**: The actual data being sent or received
- **Status Codes**: Indicate the result of the request
- **Content Types**: Specify data format (JSON, XML, etc.)

#### 4. Authentication & Authorization
- **Authentication**: Verify the identity of the requester
- **Authorization**: Determine what actions the requester can perform
- **API Keys**: Simple authentication mechanism
- **OAuth**: More sophisticated authorization framework

### API Benefits:

✅ **Modularity**: Separate concerns and responsibilities
✅ **Reusability**: Same API can serve multiple clients
✅ **Scalability**: Scale different components independently
✅ **Flexibility**: Change implementation without affecting clients
✅ **Integration**: Enable third-party integrations
✅ **Testing**: Easier to test individual components

## HTTP Methods and Status Codes

HTTP provides the foundation for most modern APIs, defining standard methods and status codes for communication.

### HTTP Methods (Verbs):

#### GET
```
GET /api/users/123
Purpose: Retrieve user with ID 123
Characteristics:
- Safe: No side effects on server
- Idempotent: Multiple calls have same effect
- Cacheable: Responses can be cached
```

#### POST
```
POST /api/users
Body: {"name": "John Doe", "email": "john@example.com"}
Purpose: Create a new user
Characteristics:
- Not safe: Creates new resources
- Not idempotent: Multiple calls create multiple resources
- Not cacheable: Responses shouldn't be cached
```

#### PUT
```
PUT /api/users/123
Body: {"name": "John Smith", "email": "john.smith@example.com"}
Purpose: Replace entire user resource
Characteristics:
- Not safe: Modifies server state
- Idempotent: Multiple calls have same effect
- Not cacheable: Responses shouldn't be cached
```

#### PATCH
```
PATCH /api/users/123
Body: {"email": "newemail@example.com"}
Purpose: Partially update user resource
Characteristics:
- Not safe: Modifies server state
- Not necessarily idempotent: Depends on implementation
- Not cacheable: Responses shouldn't be cached
```

#### DELETE
```
DELETE /api/users/123
Purpose: Remove user with ID 123
Characteristics:
- Not safe: Removes resources
- Idempotent: Multiple calls have same effect
- Not cacheable: Responses shouldn't be cached
```

### HTTP Status Codes:

#### 2xx Success
- **200 OK**: Request succeeded
- **201 Created**: Resource successfully created
- **202 Accepted**: Request accepted for processing
- **204 No Content**: Request succeeded, no content to return

#### 3xx Redirection
- **301 Moved Permanently**: Resource permanently moved
- **302 Found**: Resource temporarily moved
- **304 Not Modified**: Resource hasn't changed (caching)

#### 4xx Client Errors
- **400 Bad Request**: Invalid request syntax
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource doesn't exist
- **409 Conflict**: Request conflicts with current state
- **422 Unprocessable Entity**: Valid syntax but semantic errors
- **429 Too Many Requests**: Rate limit exceeded

#### 5xx Server Errors
- **500 Internal Server Error**: Generic server error
- **502 Bad Gateway**: Invalid response from upstream server
- **503 Service Unavailable**: Server temporarily unavailable
- **504 Gateway Timeout**: Upstream server timeout

### Best Practices for HTTP APIs:

#### URL Design:
```
Good:
GET /api/v1/users          # Get all users
GET /api/v1/users/123      # Get specific user
POST /api/v1/users         # Create user
PUT /api/v1/users/123      # Update user
DELETE /api/v1/users/123   # Delete user

Bad:
GET /api/getUsers          # Verb in URL
POST /api/user/delete/123  # Wrong method for action
GET /api/users?action=create # Action in query parameter
```

#### Response Format:
```json
{
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "1.0"
  }
}
```

## API Protocols & Interaction Models

Different protocols and interaction models serve different use cases and requirements.

### REST (Representational State Transfer)

REST is an architectural style for designing networked applications, particularly web services.

#### REST Principles:

#### 1. Stateless
```
Each request contains all information needed to process it
Client → [Request with all context] → Server
Client ← [Complete response] ← Server
```

#### 2. Client-Server Architecture
```
Client (UI/Mobile App) ←→ Server (API/Business Logic)
```

#### 3. Cacheable
```
Client → Server → [Cache-Control: max-age=3600] → Client
Client → [Cache Hit] → Response (no server call)
```

#### 4. Uniform Interface
```
Standard HTTP methods (GET, POST, PUT, DELETE)
Standard status codes (200, 404, 500, etc.)
Standard headers (Content-Type, Authorization, etc.)
```

#### 5. Layered System
```
Client → Load Balancer → API Gateway → Microservice → Database
```

#### 6. Code on Demand (Optional)
```
Server can send executable code to client (JavaScript, etc.)
```

#### REST API Example:
```
# User Management API
GET    /api/v1/users           # List all users
GET    /api/v1/users/123       # Get user by ID
POST   /api/v1/users           # Create new user
PUT    /api/v1/users/123       # Update user
DELETE /api/v1/users/123       # Delete user

# Nested Resources
GET    /api/v1/users/123/orders    # Get user's orders
POST   /api/v1/users/123/orders    # Create order for user
```

#### REST Advantages:
✅ **Simple**: Easy to understand and implement
✅ **Scalable**: Stateless nature enables scaling
✅ **Cacheable**: Built-in caching support
✅ **Flexible**: Works with any data format
✅ **Widely Supported**: Extensive tooling and libraries

#### REST Disadvantages:
❌ **Over-fetching**: May return more data than needed
❌ **Under-fetching**: May require multiple requests
❌ **Rigid**: Fixed endpoints and data structures
❌ **Versioning**: API versioning can be complex

### SOAP (Simple Object Access Protocol)

SOAP is a protocol for exchanging structured information in web services.

#### SOAP Characteristics:
```xml
<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header>
    <authentication>
      <username>user123</username>
      <password>pass456</password>
    </authentication>
  </soap:Header>
  <soap:Body>
    <getUserRequest>
      <userId>123</userId>
    </getUserRequest>
  </soap:Body>
</soap:Envelope>
```

#### SOAP Features:
- **Protocol**: Strict protocol with defined standards
- **XML-based**: All messages in XML format
- **WSDL**: Web Services Description Language for API definition
- **Built-in Security**: WS-Security standards
- **Transaction Support**: Built-in transaction handling
- **Error Handling**: Standardized fault handling

#### SOAP Advantages:
✅ **Standardized**: Strict standards and specifications
✅ **Security**: Built-in security features
✅ **Reliability**: Built-in error handling and transactions
✅ **Language Agnostic**: Works with any programming language
✅ **Enterprise Ready**: Mature enterprise features

#### SOAP Disadvantages:
❌ **Complex**: More complex than REST
❌ **Verbose**: XML overhead increases message size
❌ **Performance**: Slower due to XML parsing
❌ **Limited Browser Support**: Not natively supported in browsers

### GraphQL

GraphQL is a query language and runtime for APIs that allows clients to request exactly the data they need.

#### GraphQL Query Example:
```graphql
# Query
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    posts {
      title
      content
      createdAt
    }
  }
}

# Variables
{
  "id": "123"
}

# Response
{
  "data": {
    "user": {
      "id": "123",
      "name": "John Doe",
      "email": "john@example.com",
      "posts": [
        {
          "title": "My First Post",
          "content": "Hello World!",
          "createdAt": "2024-01-01T12:00:00Z"
        }
      ]
    }
  }
}
```

#### GraphQL Features:

#### 1. Single Endpoint
```
All requests go to: POST /graphql
Different operations specified in query body
```

#### 2. Flexible Queries
```graphql
# Client specifies exactly what data it needs
query {
  user(id: "123") {
    name        # Only fetch name
    email       # and email
  }
}
```

#### 3. Type System
```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}
```

#### 4. Real-time Subscriptions
```graphql
subscription {
  messageAdded(channelId: "123") {
    id
    content
    user {
      name
    }
  }
}
```

#### GraphQL Advantages:
✅ **Efficient**: Fetch exactly what you need
✅ **Flexible**: Clients control data requirements
✅ **Strongly Typed**: Built-in type system
✅ **Introspective**: Self-documenting APIs
✅ **Real-time**: Built-in subscription support

#### GraphQL Disadvantages:
❌ **Complexity**: More complex than REST
❌ **Caching**: More difficult to cache responses
❌ **Learning Curve**: Requires learning new concepts
❌ **Over-fetching**: Possible with poorly designed queries

### Polling + WebSockets

These technologies enable real-time communication between clients and servers.

#### Polling

##### Short Polling:
```
Client → [Request] → Server
Client ← [Response] ← Server
... wait interval ...
Client → [Request] → Server (repeat)
```

**Characteristics:**
- **Simple**: Easy to implement
- **Inefficient**: Many unnecessary requests
- **Latency**: Delay between updates and detection
- **Server Load**: Constant requests even when no updates

##### Long Polling:
```
Client → [Request] → Server (holds connection)
                     Server ← [Data available]
Client ← [Response] ← Server
Client → [New Request] → Server (immediately)
```

**Characteristics:**
- **More Efficient**: Fewer unnecessary requests
- **Lower Latency**: Immediate response when data available
- **Complex**: More complex error handling
- **Resource Usage**: Holds server connections

#### WebSockets

WebSockets provide full-duplex communication over a single TCP connection.

```
Client → [HTTP Upgrade Request] → Server
Client ← [HTTP 101 Switching Protocols] ← Server
Client ↔ [Bidirectional Messages] ↔ Server
```

**WebSocket Features:**
- **Full Duplex**: Both client and server can send messages
- **Low Latency**: No HTTP overhead after connection
- **Persistent**: Connection stays open
- **Binary Support**: Can send binary data

**Use Cases:**
- **Real-time Chat**: Instant messaging applications
- **Live Updates**: Stock prices, sports scores
- **Gaming**: Real-time multiplayer games
- **Collaboration**: Shared documents, whiteboards

#### Comparison:

| Feature | Short Polling | Long Polling | WebSockets |
|---------|---------------|--------------|------------|
| **Latency** | High | Medium | Low |
| **Server Load** | High | Medium | Low |
| **Complexity** | Low | Medium | High |
| **Real-time** | No | Partial | Yes |
| **Bidirectional** | No | No | Yes |
| **Browser Support** | Universal | Universal | Modern |

### gRPC (Google Remote Procedure Call)

gRPC is a high-performance, open-source RPC framework that uses HTTP/2 and Protocol Buffers.

#### gRPC Characteristics:

#### 1. Protocol Buffers
```protobuf
// user.proto
syntax = "proto3";

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
}
```

#### 2. HTTP/2 Based
```
Features:
- Multiplexing: Multiple requests over single connection
- Server Push: Server can push data to client
- Binary Protocol: More efficient than text-based protocols
- Header Compression: Reduced overhead
```

#### 3. Code Generation
```bash
# Generate client and server code from .proto files
protoc --go_out=. --go-grpc_out=. user.proto
```

#### gRPC Advantages:
✅ **Performance**: Binary protocol, HTTP/2 multiplexing
✅ **Type Safety**: Strong typing with Protocol Buffers
✅ **Code Generation**: Automatic client/server code generation
✅ **Streaming**: Built-in streaming support
✅ **Cross-platform**: Works across different languages

#### gRPC Disadvantages:
❌ **Browser Support**: Limited browser support
❌ **Debugging**: Binary protocol harder to debug
❌ **Learning Curve**: More complex than REST
❌ **Tooling**: Less mature ecosystem than REST

## API Security

API security is crucial for protecting data and preventing unauthorized access.

### Authentication vs Authorization

#### Authentication
**Definition**: Verifying the identity of the user or system
**Question**: "Who are you?"

**Methods:**
- **Username/Password**: Traditional credentials
- **API Keys**: Simple token-based authentication
- **Certificates**: X.509 certificates for mutual TLS
- **Biometrics**: Fingerprint, facial recognition

#### Authorization
**Definition**: Determining what actions the authenticated user can perform
**Question**: "What are you allowed to do?"

**Methods:**
- **Role-Based Access Control (RBAC)**: Permissions based on roles
- **Attribute-Based Access Control (ABAC)**: Permissions based on attributes
- **Access Control Lists (ACL)**: Explicit permissions for resources

### JWT (JSON Web Token) Authorization Mechanism

JWT is a compact, URL-safe means of representing claims between two parties.

#### JWT Structure:
```
Header.Payload.Signature

# Example:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### JWT Components:

#### 1. Header
```json
{
  "alg": "HS256",    # Algorithm used for signing
  "typ": "JWT"       # Token type
}
```

#### 2. Payload
```json
{
  "sub": "1234567890",           # Subject (user ID)
  "name": "John Doe",            # Custom claim
  "iat": 1516239022,             # Issued at
  "exp": 1516242622,             # Expiration time
  "aud": "api.example.com",      # Audience
  "iss": "auth.example.com"      # Issuer
}
```

#### 3. Signature
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

#### JWT Workflow:
```
1. User → [Login] → Auth Server
2. Auth Server → [Validate] → [Generate JWT] → User
3. User → [API Request + JWT] → API Server
4. API Server → [Validate JWT] → [Process Request] → Response
```

#### JWT Advantages:
✅ **Stateless**: No server-side session storage
✅ **Scalable**: Easy to scale across multiple servers
✅ **Cross-domain**: Works across different domains
✅ **Self-contained**: Contains all necessary information
✅ **Standard**: Well-defined standard (RFC 7519)

#### JWT Disadvantages:
❌ **Size**: Larger than simple session tokens
❌ **Revocation**: Difficult to revoke before expiration
❌ **Security**: Sensitive data exposed if not encrypted
❌ **Debugging**: Base64 encoded, harder to debug

### SSO (Single Sign-On) Authorization Mechanism

SSO allows users to authenticate once and access multiple applications.

#### SSO Protocols:

#### 1. SAML (Security Assertion Markup Language)
```xml
<saml:Assertion>
  <saml:Subject>
    <saml:NameID>user@example.com</saml:NameID>
  </saml:Subject>
  <saml:AttributeStatement>
    <saml:Attribute Name="Role">
      <saml:AttributeValue>Admin</saml:AttributeValue>
    </saml:Attribute>
  </saml:AttributeStatement>
</saml:Assertion>
```

#### 2. OAuth 2.0 / OpenID Connect
```
1. User → [Login] → Identity Provider
2. Identity Provider → [Authorization Code] → Application
3. Application → [Exchange Code for Token] → Identity Provider
4. Identity Provider → [Access Token + ID Token] → Application
```

#### SSO Benefits:
✅ **User Experience**: Single login for multiple apps
✅ **Security**: Centralized authentication
✅ **Administration**: Easier user management
✅ **Compliance**: Better audit trails

### Multi-Factor Authentication (MFA)

MFA requires multiple forms of verification to authenticate users.

#### Authentication Factors:

#### 1. Something You Know
- **Passwords**: Traditional text passwords
- **PINs**: Numeric personal identification numbers
- **Security Questions**: Personal information questions

#### 2. Something You Have
- **SMS Codes**: Text message verification codes
- **Hardware Tokens**: Physical security devices
- **Mobile Apps**: Authenticator applications (Google Authenticator, Authy)

#### 3. Something You Are
- **Biometrics**: Fingerprint, facial recognition, iris scan
- **Voice Recognition**: Voice pattern analysis
- **Behavioral**: Typing patterns, mouse movements

#### MFA Implementation:
```
1. User → [Username/Password] → System
2. System → [Request Second Factor] → User
3. User → [Provide Second Factor] → System
4. System → [Validate Both Factors] → [Grant Access]
```

#### MFA Best Practices:
- **Risk-Based**: Require MFA for high-risk operations
- **User-Friendly**: Provide multiple MFA options
- **Backup Methods**: Alternative authentication methods
- **Recovery Process**: Account recovery procedures

## Conclusion

APIs are the backbone of modern distributed systems, enabling communication between different components and services. Key takeaways:

1. **Choose the Right Protocol**: REST for simplicity, GraphQL for flexibility, gRPC for performance
2. **Design for Security**: Implement proper authentication and authorization
3. **Follow Standards**: Use HTTP methods and status codes correctly
4. **Plan for Scale**: Design APIs that can handle growth
5. **Document Thoroughly**: Provide clear API documentation
6. **Version Carefully**: Plan for API evolution and backward compatibility

Understanding these concepts enables you to design robust, secure, and scalable APIs that serve as reliable interfaces between different parts of your system.

## Next Steps

In the following sections, we'll explore:
- **Networking**: Communication patterns and protocols
- **Caching**: Performance optimization strategies
- **Security**: Advanced security patterns and practices
- **Monitoring**: API observability and analytics