
## UserDetails

In [Spring Security](https://docs.spring.io/spring-security/reference/index.html), **`UserDetails` is a core interface that represents a user principal containing the necessary information (such as username, password, and authorities) to perform authentication and authorization**. [](https://www.geeksforgeeks.org/advance-java/spring-security-userdetailsservice-and-userdetails-with-example/)

It acts as a data wrapper that translates your application's user entity into a format that Spring Security's internal engines can easily read and evaluate. [](https://www.geeksforgeeks.org/advance-java/spring-security-userdetailsservice-and-userdetails-with-example/)

---

Core Methods of the Interface

The [UserDetails interface API](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/userdetails/UserDetails.html) requires implementing the following 7 core methods: [](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/userdetails/UserDetails.html)

|Method|Return Type|Purpose|
|---|---|---|
|`getUsername()`|`String`|Returns the unique identifier/username of the user.|
|`getPassword()`|`String`|Returns the hashed password used for validation.|
|`getAuthorities()`|`Collection<? extends GrantedAuthority>`|Returns the user's granted permissions or roles (e.g., `ROLE_USER`, `ROLE_ADMIN`).|
|`isAccountNonExpired()`|`boolean`|Checks if the user's account has expired.|
|`isAccountNonLocked()`|`boolean`|Checks if the user is locked (e.g., due to too many bad login attempts).|
|`isCredentialsNonExpired()`|`boolean`|Checks if the user's password/credentials have expired.|
|`isEnabled()`|`boolean`|Checks if the user account is active or disabled.|

---

How It Fits Into the Authentication Flow

- **The Request**: A user submits their login credentials (username and password).

- **The Fetch**: Spring Security delegates the lookup to a [UserDetailsService](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details-service.html) via its `loadUserByUsername(username)` method.

- **The Wrapper**: Your data store fetches the user, wraps it into a `UserDetails` object, and returns it to the framework.

- **The Validation**: The framework's `DaoAuthenticationProvider` compares the submitted password against the password inside the `UserDetails` object, while also checking the account status booleans


	