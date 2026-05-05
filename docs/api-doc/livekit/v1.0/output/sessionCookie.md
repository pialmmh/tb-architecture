name: sessionCookie
type: HTTP `Set-Cookie` header (HttpOnly, SameSite=Lax) + side-effect: server-side `HttpSession`
trigger: [session](../trigger/session.md)

Issued on successful login; cleared on logout. Cookie carries the Spring session id; the session itself lives in process memory and references the authenticated `User`. No payload is exposed in the cookie value beyond the opaque session id.

Lifetime is governed by Spring Boot's session timeout (default 30 min idle). `Secure` flag is set in non-dev environments.
