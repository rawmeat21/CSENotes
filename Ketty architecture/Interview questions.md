## "Walk me through your tech stack selection. Why did you choose these specific frontend and backend technologies?"

Backend:

I selected **Spring Boot** for the backend primarily to leverage Java’s **static typing**, it's object oriented structure which keeps the code easy to understand and modify, and **production-ready security infrastructure**.

Spring Boot allowed us to cleanly segregate concerns using the **Controller-Service-Repository** structure. By utilizing **Spring’s IoC container**, component instantiation and dependency management are handled at runtime.

For data persistence, I used **Spring Data JPA with Hibernate as the ORM provider**. This helped map objects in my code directly to database tables. By extending JpaRepository, we can also prevent the need to write SQL statements, we just need to write proper function names and during compilation of code, Spring generates a full implmenetation of the function with a boilerplate code.

Finally, rather than building custom security logic, I used **Spring Security’s filter chain architecture**. This gave me a standardized way to handle request filtering, authentication providers, and password encoding before requests ever touch our application services.


PostgreSQL:

In my application, the enitites like User, Posts, Projects, have well defined attributes and strict relationships with each other. Also, using a relational Database model allowed me to enforce different types of constraints at the database level, separate from my application. Also, using relational databases guarantees ACID compliance, so I could trust that the DB would be in a consistent state across transactions. 

Why PostgreSQL: 

- PostgreSQL uses Multi-Version Concurrency Control (MVCC), meaning read queries don't block write queries and vice versa. This makes it performant under concurrent load. 

- If certain features require unstructured data down the line, Postgres supports binary JSON (`jsonb`) indexing, giving you NoSQL-like flexibility inside a relational database without needing to spin up a separate MongoDB instance.
- It can be integrated well with Java SpringBoot

React.js:

On the frontend, I chose **React.js** mainly because it allowed me to build an interactive, single-page web app where page reloads aren't necessary every time a user does something.

A few simple reasons why it worked well for this project:

1. **Component-based design:** I could build individual UI pieces like post cards, navigation bars, or media sections and reuse them across different pages.
    
2. **Fast state updates:** When users interact with the page—like clicking through tabs, typing in form inputs, or opening popups—React updates just those specific components on the screen without re-rendering the whole page.
    
3. **Clean separation from the backend:** React manages the visual interface directly on the browser and simply calls my Spring Boot REST APIs whenever it needs to send or fetch data."

## **"Can you explain your database schema design?"**

See the schema.

## **"What was the most challenging bug or technical hurdle you faced in this project, and how did you resolve it?"**

Talk about CORS.

## **"What was your strategy for testing and deploying this application?"**

**"How does authentication work end-to-end in your app?"**

**"Where are you storing the JWT on the client side, and why?"**

I store JWT tokens in the local storage of the client's browser. When the user logs in, a jwt token is generated and sent back to the client, this token is saved on the local storage of the browser. I understand that it has a flaw as cross site scripting (XSS) can create a vulnerability if malicious JS code is run by the client's browser. But React treats content like post content, bios etc as plaintext so I thought that would not be much of an issue. 

**"How do you handle expired tokens or unauthorized (401) API responses?"**

The backend has a `JwtAuthenticationFilter` that runs once per request. It checks for a Bearer token, and if present, validates it via `JwtService` and sets the Spring Security authentication context so downstream controllers know who the user is. If there's no token, , no authentication gets set, Spring Security's own access-control layer then returns a 401 for any endpoint that requires authentication.

One gap I found is: an _expired_ token throws an unchecked `ExpiredJwtException` during parsing and that's not specifically caught, so  it falls through to my generic exception handler and returns a 500 instead of a 401. So right now, an expired token doesn't actually produce the clean 401; I'd fix that by catching `JwtException` explicitly and mapping it to a 401.

**"What is the N+1 select problem in JPA/Hibernate, and did you run into it?"**

Imagine you're a teacher and you want to know the average marks of every student in your class of 30.

**The dumb way:** You go to the office, ask "give me the list of all 30 students." They hand you the list. Then, for **each student one at a time**, you walk back to the office and ask "what are this student's marks?" That's 1 trip to get the list, plus 30 more trips — one per student. **31 trips total**, when really the office could've just given you everything in one go if you'd asked properly.

**The smart way:** You ask once — "give me all 30 students _along with_ their marks" — and get everything in a single trip.

That "dumb way" — 1 query for the list, then N more queries (one per item in that list) to get related details — is exactly what the N+1 problem is. Each "trip to the office" is one database query. Database queries aren't free — each one takes time (network round-trip, DB processing). Doing 31 of them instead of 1 is much slower, and it gets worse as your class size grows — 1000 students means 1001 trips.

Yes, I did run into this problem. So it was like this: I had a function `getAllPosts()` under `PostService`, which fetched all posts made by the user. Now, this function would find all posts by quering the posts table, then to build each Post object, it would require things like all comments for this post, all likes, whether user liked the post or not, how many comments were in the post - so it would make a query for each of those posts. So first 1 query for all posts, then multiple queries for constructing each post object

### "If 50,000 users sign up tomorrow, what part of your stack breaks first?"

**External third-party APIs break first — before your own servers even feel pressure.**  
Your `SearchController` hits RAWG, TMDB, Jikan, LastFM, and Google Books directly, per request, with **no caching layer**. Free-tier API keys on services like these typically allow only a handful of requests per second (or a daily cap). If even a small fraction of 50,000 users search for games/movies/anime around the same time, you'd hit **third-party rate limits** almost immediately

**No pagination compounds all of the above.**  
`getAllPosts`, `getProjects`, and `getHobbies` return the user's _entire_ list every time, with no `Pageable`/limit. A user with thousands of posts turns every feed load into an increasingly expensive full-table scan-and-map operation — worse under concurrent load, since more of these expensive queries are competing for the same limited connection pool.

**Cloudinary upload limits, further out but real.**  
`UploadController` has no file size or content-type validation before forwarding to Cloudinary. A free/dev-tier Cloudinary account has bandwidth and storage caps — a burst of large or numerous uploads could hit those limits, and there's no client-side or server-side guard preventing it.


### "Why route third-party API searches (like Spotify, games, or books) through your backend instead of calling them directly from React?"

#### API keys must never live in the frontend

#### Rate limiting and quota protection

If React called RAWG directly, every single browser tab, for every single user, would be making its own independent calls straight to RAWG using your key — you have zero control over how many requests get made in aggregate.

I have five different external APIs with five different response formats, routing through my backend lets me normalize all of them into one consistent `SearchResultItem` object before they hit the UI, so React only ever deals with one schema, and I only have to update one place if a third-party API changes.

### "As users scroll through feeds with lots of images, how do you prevent the page from lagging?"

"Right now, honestly, I'm not doing anything special — my backend returns all of a user's posts in one response, so the frontend renders everything it receives. That's fine at small scale, but it wouldn't hold up with someone who has hundreds of posts with images."

Some solutions could be:

**1. Pagination — don't load what you don't need yet**  
Instead of asking the backend for all 500 posts, ask for 20 at a time (`?page=0&size=20`), and fetch the next batch only when the user scrolls near the bottom ("infinite scroll") or clicks "load more." This is the single highest-impact fix, and it's a backend change first — your `PostController`/`PostService` would need to accept a `Pageable` and return a `Page<PostResponse>` instead of a full `List`.

**2. Lazy-load images — don't download what's off-screen**  
Even with pagination, 20 posts might still mean 20 images. The browser's native `<img loading="lazy">` attribute tells it to skip downloading an image until it's about to scroll into view, instead of downloading all 20 the moment the page loads. This is a one-line change per `<img>` tag and costs nothing to add.

**3. Serve appropriately-sized images, not full-resolution originals**  
Since you're using Cloudinary, this one's basically free — Cloudinary can resize/compress images on the fly via URL parameters (e.g. appending transformation params to the `secure_url` your `CloudinaryService` returns), so a thumbnail in a feed doesn't force the browser to download a multi-megabyte original photo just to shrink it visually with CSS.

### "How did you handle environment variables and secrets during development versus production?"

I store the external API keys in application.properties which can be read by my Spring backend when required. For environment variables like `VITE_API_URL` I stored it in .env. Though, I plan to put everything in the .env file n future, so if I ever accidentally push the application.properties file, I don't risk leaking my API keys.

## Why did you build this? What problem were you solving, and for whom?

I wanted a place where I could track my hobbies, skills as well as ongoing projects. I wanted a place where people could express their creative lives more freely. Apps like Instagram or Facebook don't really have this feature, people do post about their interests but its not as organised. I took inspiration from several sites like Letterboxd, Spotify, MAL to come up with this idea where we can track all of our interests at once. My web app is meant to be a platform for people who want to grow and express their skills and interests. I would say the goal of this app is to serve as a life tracker, where people can share their profiles to instantly let the other person know more about them.

## What was the single hardest part of building this, and why?



## If you had another month, what would you add or change? 

Well, there are several categories (Hobby Type) under which my users can post on. Users can also have their own categories like wood cutting, surfing, etc. The problem is there. Users can name categories whatever they want, so I can have categories like "Wood cutting" or "wood cutting", which are the same thing, but treated differently. I would like to add a unified method, where if a hobby type already exists in system, I can give the user a suggestion to use that hobby type name.

## What would you do differently if you started over today?



## What's the one part of this project you're proudest of, and why?

One part that I'm proud of is that the project works both on my PC and my mobile phone. Also, I think the app has turned out to largely what I had in mind. External APIs allowed me to easily add my favourite anime, so I had a fun time with my own app

## What did you learn building this that you didn't already know?

There are several things that I ended up learning about through this project. First, I understood the application of object oriented concepts, how important they are for writing code that can be modified or changed without affecting the whole application. I learned about REST APIs, how to structure a backend application into controller, service and repository structure. I understood how servers handle authentication and authorization, how JWT tokens can be used to track identity of users and prevent attackers from faking their identity. 

## Did you write any tests? Why or why not?

I didn't write automated tests — I tested manually through Postman for the API and by using the app myself on desktop and mobile browsers. 

## How do you debug something you don't understand? Walk me through your actual process, not the theory


## Did you use any AI tools while building this? Be honest, and be ready to explain what you understood vs. what you copied.


**"What steps did you take to protect your application against common security vulnerabilities?"**



**"Can you explain the data flow when a user submits a form or triggers an action?"**

**"Describe a complex bug you encountered in this project and how you fixed it."**

**"What is the difference between REST APIs and GraphQL, and why did you use REST for this project?"**

**"How does your frontend communicate with your backend? Explain CORS."**

**"Explain how you structured your database transactions. What happens if your backend updates the user's balance but crashes right before creating the order entry?"**

**"You used Java/Spring Boot for your backend. Is your server single-threaded or multi-threaded? How does it handle 5,000 concurrent requests?"**

**"Walk me through the exact network lifecycle when your frontend sends an authenticated `POST /checkout` request to your backend over HTTPS."**


**"If an attacker intercepts your JWT token from the browser, they can impersonate the user. Where did you store your tokens, and how did you prevent XSS (Cross-Site Scripting) and CSRF (Cross-Site Request Forgery)?"**


**"Show me where you applied SOLID principles or a specific Design Pattern (like Singleton, Factory, or Strategy) inside your project code."**


**"You mentioned you used JSON Web Tokens (JWT) for authentication. If a user changes their password or an administrator bans a malicious user, how do you instantly revoke that user's active JWT before it naturally expires?"**


**"You used an ORM/ODM (like Prisma, Mongoose, Hibernate). When your backend server fires 50 concurrent database queries, how many TCP connections are opened? Explain connection pooling."**


**"Analyze the asymptotic Time and Space Complexity of your entire API data pipeline."**



**"Spring Boot uses an embedded Tomcat server. By default, how does Tomcat handle multiple concurrent API requests? What happens if 500 requests hit your React endpoints simultaneously?"**



**"How does Spring manage the lifecycle of your classes? If you inject a Repository into a Controller, is a new instance created for every API request?"**



**"How does your Spring Boot backend verify that a incoming JWT from React hasn't been altered by the user? Walk me through the mathematical signature validation."**



**"What is the exact internal startup lifecycle when you invoke `SpringApplication.run()`? How does Spring Boot dynamically choose which Auto-Configurations to apply?"**



**"If you use `@Autowired` on a field layer versus a constructor layer, how does that impact the testability and lifecycle of your Spring Beans?"**



**"Explain the structural difference between Symmetric (HS256) and Asymmetric (RS256) signing for JWT. Why do top enterprise firms mandate RS256?"**


- What's the difference between `@Component`, `@Service`, `@Repository`, and `@Bean`? Are they functionally different, or just semantic?
- What is the Spring IoC container, and what does "bean" actually mean?
- What's the default scope of a Spring bean, and when would you use a different one?
- What does `@Valid` on `@RequestBody` actually do under the hood — what happens if you remove it from `RegisterRequest`? _(Hint: validation annotations like `@NotBlank` become no-ops without `@Valid` triggering them — bad data would reach your service layer untouched.)_
- Your entities use `@Enumerated(EnumType.STRING)` for `ProjectStatus` and `HobbyType`. Why `STRING` instead of the default `ORDINAL`? What breaks if you reorder your enum constants under `ORDINAL`?
- What's the difference between checked and unchecked exceptions in Java, and why does almost all of your business logic throw plain `RuntimeException` instead of a checked exception?
- What is `Optional<T>` used for (you use `.orElseThrow()` constantly) — why is it preferable to returning `null`?

Why relational (JPA/MySQL-or-Postgres) instead of a NoSQL document store for this schema? What would change if you'd picked MongoDB instead?

Is your app protected against SQL injection? Why or why not?

Right now, is there anything stopping someone from scripting thousands of login attempts against `/api/auth/login`? _(No rate limiting/brute-force lockout exists — know this gap and how you'd add it, e.g. Bucket4j or a simple attempt-counter.)_

How would you implement "logout" cleanly, given your system is fully stateless and has no server-side session to destroy?

How would you add a "forgot password" flow to this app? Walk through it end to end.

Where is this actually deployed, and how? Walk me through your deployment process.

What's the difference between horizontal and vertical scaling, and which one does your stateless-JWT design make easier?


If this needed a CI/CD pipeline, what stages would you include?

If you were told "we're giving you two more weeks on this project before it ships to real users," what's your prioritized list of fixes?

What's one decision in this project you'd defend even if someone challenged it?





