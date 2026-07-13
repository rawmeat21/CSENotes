### Part 1: Authentication & Security Filter Chain

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Filter as JwtAuthenticationFilter
    participant AuthCtrl as AuthController
    participant UserSvc as UserService
    participant JwtSvc as JwtService
    participant DB as UserRepository

    Client->>Filter: POST /api/auth/login (LoginRequest request)
    Note over Filter: Bypasses token verification for /api/auth/** paths
    Filter->>AuthCtrl: doFilter(HttpServletRequest request, HttpServletResponse response)
    AuthCtrl->>UserSvc: login(LoginRequest request)
    UserSvc->>DB: findByUsername(String username)
    DB-->>UserSvc: Optional~User~
    Note over UserSvc: AuthenticationManager verifies credentials internally
    UserSvc->>JwtSvc: generateToken(UserDetails userDetails)
    JwtSvc-->>UserSvc: String token
    UserSvc-->>AuthCtrl: AuthResponse (contains token, username, email, displayName)
    AuthCtrl-->>Client: ResponseEntity~AuthResponse~ (Status 200)
```


### Part 2: Authenticated Content Lifecycle (Posts & Media)

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Filter as JwtAuthenticationFilter
    participant JwtSvc as JwtService
    participant PostCtrl as PostController
    participant PostSvc as PostService
    participant PostRepo as PostRepository

    Client->>Filter: POST /api/posts (Bearer Token, CreatePostRequest request)
    Filter->>JwtSvc: extractUsername(String token)
    JwtSvc-->>Filter: String username
    Note over Filter: Validates token & sets SecurityContextHolder
    Filter->>PostCtrl: createPost(User currentUser, CreatePostRequest request)
    PostCtrl->>PostSvc: createPost(String username, CreatePostRequest request)
    Note over PostSvc: Fetches User and Hobby entities via respective repositories
    PostSvc->>PostRepo: save(Post post)
    PostRepo-->>PostSvc: Post savedPost
    PostSvc->>PostSvc: mapToPostResponse(Post post, User currentUser)
    PostSvc-->>PostCtrl: PostResponse
    PostCtrl-->>Client: ResponseEntity~PostResponse~ (Status 201)
```

### Part 3: Composite Entity Persistence (Projects & Tools)

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant ProjCtrl as ProjectController
    participant ProjSvc as ProjectService
    participant ProjRepo as ProjectRepository
    participant ToolRepo as ProjectToolRepository

    Client->>ProjCtrl: POST /api/projects (CreateProjectRequest request)
    ProjCtrl->>ProjSvc: createProject(String username, CreateProjectRequest request)
    Note over ProjSvc: Instantiates Project and sets ProjectStatus
    ProjSvc->>ProjRepo: save(Project project)
    ProjRepo-->>ProjSvc: Project savedProject
    
    rect rgb(30, 30, 30)
        Note over ProjSvc, ToolRepo: Loop execution for request.getTools()
        loop For each String toolName
            ProjSvc->>ToolRepo: save(ProjectTool tool)
        end
    end
    
    ProjSvc->>ProjSvc: mapToProjectResponse(Project project)
    ProjSvc-->>ProjCtrl: ProjectResponse
    ProjCtrl-->>Client: ResponseEntity~ProjectResponse~ (Status 201)
```


### Part 4: Third-Party Search & Hobby Aggregation

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant SearchCtrl as SearchController
    participant TmdbSvc as TmdbSearchService
    participant WebClient as External API
    participant HobbyCtrl as HobbyController
    participant HobbySvc as HobbyService
    participant HobbyRepo as HobbyEntryRepository

    %% Step 1: External Search Execution
    Client->>SearchCtrl: GET /api/search/movies?q=Interstellar
    SearchCtrl->>TmdbSvc: searchMovies(String q)
    TmdbSvc->>WebClient: get().uri(path, query).retrieve()
    WebClient-->>TmdbSvc: JSON Response
    Note over TmdbSvc: Parses JSON node arrays
    TmdbSvc-->>SearchCtrl: List~SearchResultItem~
    SearchCtrl-->>Client: ResponseEntity~List~SearchResultItem~~ (Status 200)

    %% Step 2: Saving the External Data Locally
    Client->>HobbyCtrl: POST /api/hobbies/{hobbyId}/entries (AddHobbyEntryRequest request)
    HobbyCtrl->>HobbySvc: addEntry(String username, Long hobbyId, AddHobbyEntryRequest request)
    HobbySvc->>HobbyRepo: save(HobbyEntry entry)
    HobbyRepo-->>HobbySvc: HobbyEntry savedEntry
    HobbySvc->>HobbySvc: mapToHobbyEntryResponse(HobbyEntry entry)
    HobbySvc-->>HobbyCtrl: HobbyEntryResponse
    HobbyCtrl-->>Client: ResponseEntity~HobbyEntryResponse~ (Status 201)
```

