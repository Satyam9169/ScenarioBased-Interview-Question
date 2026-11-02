# How Will You Secure Your React App?

*Senior React.js Developer — Interview Preparation*

## Interview Answer

I would secure my React application using multiple layers of security. React handles the user interface, but the backend must enforce authentication, authorization, validation, and access control.

1. **Prevent XSS attacks:** Avoid rendering untrusted HTML with dangerouslySetInnerHTML. Sanitize user-generated content when necessary.
2. **Secure authentication:** Use a secure authentication mechanism. Prefer HttpOnly, Secure, and SameSite cookies for session management when appropriate.
3. **Implement authorization:** Use protected routes and role-based access control (RBAC) to control the UI, but always enforce authorization on the backend too.
4. **Protect API calls:** Use HTTPS, handle authentication failures, and ensure sensitive operations are authorized by the server.
5. **Prevent CSRF attacks:** When authentication relies on cookies, implement suitable CSRF protection on the backend.
6. **Secure sensitive information:** Never store passwords or secret API keys in frontend code. Environment variables bundled into the client are not secret.
7. **Validate user input:** Validate input on both frontend and backend. Frontend validation helps usability; backend validation is essential for security.
8. **Configure security headers:** Configure Content Security Policy (CSP), HSTS, and other appropriate security headers on the server.
9. **Manage dependencies:** Audit npm packages regularly, update vulnerable dependencies, and remove unnecessary packages.
10. **Handle errors securely:** Avoid exposing stack traces, internal system details, or sensitive information in production error messages.

## Follow-up Questions

- How do you prevent XSS attacks in React?
- Why is storing JWTs in localStorage potentially risky?
- How do you implement role-based access control in React?
- What is the difference between authentication and authorization?
- How would you protect a React application against CSRF attacks?

## Senior-Level Point to Remember

Hiding a button or protecting a React route does not secure the underlying data or API. A user can call an API directly, so the backend must independently verify permissions on every protected operation.
