# Authentication & Authorization in ReactJS
## Beginner-friendly step-by-step guide

This guide follows the topics in your original document and explains them in simple English and Hinglish, with text diagrams.

## Step 1: Authentication vs. Authorization

### Authentication — Who are you?
Authentication means verifying a user's identity. For example, a user enters an email and password, and the application checks whether the credentials are valid.

**Hindi:** Authentication ka matlab hai check karna ki aap kaun ho.

### Authorization — What are you allowed to do?
Authorization checks what a user is permitted to access or do. A normal user and an admin might both log in, but only the admin can open the admin dashboard.

**Hindi:** Authorization ka matlab hai check karna ki aapko kya karne ki permission hai.

### Diagram: Login and access flow
```text
              USER OPENS APP
                    |
                    v
             Enter Credentials
                    |
                    v
          +---------------------+
          |   AUTHENTICATION    |
          |    Who are you?     |
          +---------------------+
                    |
             Login successful?
               /         \\
             No           Yes
             |             |
             v             v
          Login page    Check user role
                            |
                 +----------+----------+
                 |                     |
                 v                     v
            Normal User               Admin
                 |                     |
                 v                     v
            /dashboard              /admin
                 |                     |
                 v                     v
               ALLOW                 ALLOW
```

**Interview tip:** Authentication verifies identity; authorization verifies permissions. React protected routes help control navigation, but the backend must also enforce permissions.

## Step 2: The Confused Deputy Problem

A confused deputy problem happens when a trusted service uses its own privileges for a request from a user who is not allowed to perform that action.

### Example
A hacker requests a refund for someone else's order. If the order service doesn't check ownership before asking the payment service to issue a refund, the refund might be processed incorrectly.

### Incorrect flow
```text
Hacker requests refund for Order #999
                 |
                 v
           Order Service
      (does not check ownership)
                 |
                 v
           Payment Service
                 |
                 v
          Refund processed! X
```

### Correct flow
```text
User requests refund
        |
        v
  Order Service
        |
        v
Does this user own the order?
        |
     +--+--+
     |     |
    Yes    No
     |     |
     v     v
  Refund  BLOCK
```

**Remember:** Being logged in is not enough. The service must check whether the user is allowed to perform the specific action.

## Step 3: Sessions vs. Tokens

After login, the application needs a way to recognize the user on later requests.

### 3.1 Session-based authentication
The server stores a session record and gives the browser a cookie that refers to it. On later requests, the browser sends the cookie and the server looks up the session.

```text
User logs in
    |
    v
Server creates session record
    |
    v
Browser receives session cookie
    |
    v
Browser sends cookie with request
    |
    v
Server looks up session
    |
    v
Access granted if session is valid
```

### 3.2 Token-based authentication (JWT)
The server issues a signed token to the client. On later requests, the client sends the token and the server verifies its signature and relevant claims.

```text
User logs in
    |
    v
Server issues signed JWT
    |
    v
Client stores/holds token safely
    |
    v
Client sends token with API request
    |
    v
Server verifies token and claims
    |
    v
Access granted if valid
```

### Quick comparison

| Feature | Session | JWT |
|---|---|---|
| State | Session record stored on server | Claims carried in token |
| Request check | Server looks up session | Server verifies token |
| Revocation | Can invalidate/delete session | Early revocation needs extra mechanisms |
| Main concern | Protect session and cookie | Protect token, expiry and validation |

Neither approach is automatically best for every application. Choose based on the system's needs and security design.

## Step 4: Access Token vs. Refresh Token

Some systems issue two tokens at login.

- **Access token:** Sent with protected API requests. It is short-lived on purpose.
- **Refresh token:** Used to obtain a new access token when the old one expires. It should be sent only to the refresh endpoint, not ordinary APIs.

### Diagram
```text
             USER LOGS IN
                   |
                   v
            Server issues tokens
                   |
             +-----+------+
             |            |
             v            v
        Access token  Refresh token
             |            |
             v            |
       Call protected     |
            API            |
             |             |
             v             |
      Access token expires |
             |             |
             +------|------+
                    v
       Send refresh token to refresh endpoint
                    |
                    v
          Server validates refresh token
                    |
                    v
           New access token is issued
                    |
                    v
             Continue using app
```

**Analogy:** The access token is like a hotel key card that expires after a short time. The refresh token is like a separate voucher used to get a new key card without checking in again.

## Step 5: Refresh Token Rotation

If a refresh token is stolen and reused repeatedly, an attacker may keep obtaining access tokens. Rotation reduces this risk by issuing a new refresh token and invalidating the old one every time a refresh succeeds.

### Diagram
```text
Initial tokens: Access A1 + Refresh R1
                      |
                      v
                Submit R1
                      |
                      v
             Server validates R1
                      |
                      v
       New tokens issued: Access A2 + Refresh R2
                      |
                      v
                 R1 invalidated
                      |
                      v
          Next refresh must use R2
```

If an already-used refresh token appears again, the server can treat it as suspicious reuse and revoke the related token family, depending on the implementation.

**Hindi:** Har successful refresh par naya refresh token milta hai aur purana invalid ho jata hai. Purana token dobara use hona suspicious activity ka signal ho sakta hai.

## Step 6: JWT — What Is Inside a Token?

JWT stands for JSON Web Token. A JWT has three segments separated by dots:

```text
xxxxx.yyyyy.zzzzz
   |      |     |
   v      v     v
 Header  Payload Signature
```

### Header
Contains metadata such as the token type and signing algorithm.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload
Contains claims about the subject or token, such as an identifier, role, or expiry.

```json
{
  "sub": "user_42",
  "role": "admin",
  "exp": 1799999999
}
```

The server must validate relevant claims and must not grant permissions merely because a client presents a role value.

### Signature
Helps the server verify that the token has not been changed and that it was signed by a trusted issuer.

**Important security note:** A typical signed JWT is not encrypted. Anyone who obtains it may be able to decode and read its payload. The signature helps protect integrity; it does not hide the contents. Never put passwords or secrets in a JWT payload.

## Final revision: 6 concepts

| No. | Concept | Remember this |
|---|---|---|
| 1 | Authentication | Who are you? |
| 2 | Authorization | What can you access or do? |
| 3 | Confused Deputy | A trusted service must still check the caller's permissions. |
| 4 | Sessions vs. JWT | Server-side session state vs. a signed token carrying claims. |
| 5 | Access vs. Refresh Token | Access APIs vs. obtain new access tokens. |
| 6 | JWT Structure | Header + Payload + Signature. |

## Overall flow in a React application

```text
        React Login Page
               |
               v
       Send credentials to backend
               |
               v
      Backend verifies user
               |
               v
       Login successful?
               |
               v
       Authentication done
               |
               v
      React maintains login state
               |
               v
        User opens a route
               |
               v
      Check authentication status
               |
               v
       Check required permission
               |
          +----+----+
          |         |
          v         v
       Allowed    Forbidden
          |         |
          v         v
       Show page  Deny access
```

**Important:** React protected routes improve navigation and user experience, but they are not a security boundary. The backend must independently verify authentication and authorize each protected API operation.

## Suggested study order

1. Authentication vs. authorization
2. Protected routes in React Router
3. Sessions vs. JWT
4. Access tokens and refresh tokens
5. Refresh token rotation
6. JWT structure and security
