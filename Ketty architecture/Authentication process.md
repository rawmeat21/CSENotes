### The full walkthrough

**1. Registration — `POST /api/auth/register`**

`AuthController.register()` receives a `RegisterRequest` (username, email, password), passed to `UserService.register()`:

- Checks `userRepo.existsByUsername(...)` and `existsByEmail(...)` — rejects duplicates.
- Hashes the password with `BCryptPasswordEncoder` before saving — **the plain password is never stored**, only the bcrypt hash.
- Saves the new `User` entity.
- Immediately generates a JWT via `JwtService.generateToken(user)` and returns it in `AuthResponse` — so registration logs the user in automatically, no separate login step required.

**2. Login — `POST /api/auth/login`**

`AuthController.login()` receives a `LoginRequest`, passed to `UserService.login()`:

- Calls `authManager.authenticate(new UsernamePasswordAuthenticationToken(username, password))`. This delegates to the `AuthenticationProvider` bean configured in `SecurityConfig` — a `DaoAuthenticationProvider` wired with `CustomUserDetailsService` and the `BCryptPasswordEncoder`.
- Under the hood, Spring Security loads the user via `CustomUserDetailsService.loadUserByUsername()`, then compares the submitted password against the stored bcrypt hash. If they don't match, it throws `BadCredentialsException` — caught by `GlobalExceptionHandler` and returned as a clean `401`.
- If it matches, `UserService.login()` fetches the `User`, generates a fresh JWT, and returns it.

**3. What's actually inside the token**

`JwtService.generateToken()` builds the JWT with the username as the subject, an issued-at timestamp, an expiration timestamp (`app.jwt.expiration`), and signs it using HMAC-SHA with a secret key (`app.jwt.secret`) via `Keys.hmacShaKeyFor(...)`. The result is the standard `header.payload.signature` structure — anyone can _read_ the payload (it's just base64, not encrypted), but only someone holding the secret can produce a valid _signature_ for it, which is what actually makes it trustworthy.

**4. Client stores it**

The frontend (`Login.jsx` → `AuthContext.login()`) saves the token in `localStorage`. From then on, `axios.js`'s request interceptor automatically attaches it as `Authorization: Bearer <token>` on every outgoing API call.

**5. Every subsequent request — `JwtAuthenticationFilter`**

This filter runs once per request, before the request reaches any controller:

- No `Bearer` header → does nothing, request proceeds unauthenticated.
- Header present → extracts the token, calls `jwtService.extractUsername(token)` to get the subject out of the payload, then loads that user via `CustomUserDetailsService`.
- Calls `jwtService.isTokenValid(token, userDetails)` — checks the username matches _and_ the token isn't expired.
- If valid, manually builds a `UsernamePasswordAuthenticationToken` and places it in `SecurityContextHolder` — this is what makes the current user available in controllers via `@AuthenticationPrincipal User currentUser`.

**6. Authorization decision — `SecurityConfig`**

After the filter runs, Spring Security's own `SecurityFilterChain` checks the request against the rules defined in `securityFilterChain()`: certain paths (`/api/auth/**`, `/api/search/**`, GET on posts/profile/projects) are `permitAll()`; everything else requires `.anyRequest().authenticated()`. If no authentication was set in step 5 (missing or invalid token) and the endpoint requires auth, Spring Security itself returns a **401** — this happens _after_ the filter, in the framework's own access-control layer, not inside your filter code.

**7. Statelessness**

`SessionCreationPolicy.STATELESS` means the server keeps zero session memory between requests — every single request must carry a valid token and gets independently re-authenticated from scratch. There's no server-side "logged in" state at all; the token _is_ the entire proof of identity, every time.





