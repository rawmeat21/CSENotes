
### Part 1: Identity, Authentication & Profile Domain

```mermaid
classDiagram
    direction TB

    class User {
        -Long id
        -String username
        -String email
        -String password
        -String displayName
        -String bio
        -String status
        -String avatarUrl
        -List~ProfileLink~ profileLinks
        -List~Hobby~ hobbies
        -List~Post~ posts
        -List~Project~ projects
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        +onCreate() void
        +onUpdate() void
        +getAuthorities() Collection~GrantedAuthority~
        +getPassword() String
        +getUsername() String
        +isAccountNonExpired() boolean
        +isAccountNonLocked() boolean
        +isCredentialsNonExpired() boolean
        +isEnabled() boolean
    }

    class ProfileLink {
        -Long id
        -User user
        -String label
        -String url
    }

    class RegisterRequest {
        -String username
        -String email
        -String password
        -String displayName
    }

    class LoginRequest {
        -String username
        -String password
    }

    class AuthResponse {
        -String token
        -String username
        -String email
        -String displayName
    }

    class ProfileUpdateRequest {
        -String displayName
        -String bio
        -String status
        -String avatarUrl
        -List~ProfileLinkDTO~ links
    }

    class ProfileResponse {
        -Long id
        -String username
        -String email
        -String displayName
        -String bio
        -String status
        -String avatarUrl
        -List~ProfileLinkDTO~ links
        -LocalDateTime createdAt
    }

    class ProfileLinkDTO {
        -Long id
        -String label
        -String url
    }

    class UserRepository {
        <<Interface>>
        +findByUsername(String username) Optional~User~
        +findByEmail(String email) Optional~User~
        +existsByUsername(String username) boolean
        +existsByEmail(String email) boolean
    }

    class ProfileLinkRepository {
        <<Interface>>
        +findByUser(User user) List~ProfileLink~
        +deleteByUser(User user) void
    }

    class CustomUserDetailsService {
        -UserRepository userRepo
        +loadUserByUsername(String username) UserDetails
    }

    class UserService {
        -UserRepository userRepo
        -PasswordEncoder passwordEncoder
        -JwtService jwtService
        -AuthenticationManager authManager
        +register(RegisterRequest request) AuthResponse
        +login(LoginRequest request) AuthResponse
    }

    class ProfileService {
        -UserRepository userRepository
        -ProfileLinkRepository profileLinkRepository
        +getMyProfile(String username) ProfileResponse
        +getProfileByUsername(String username) ProfileResponse
        +updateProfile(String username, ProfileUpdateRequest request) ProfileResponse
        -mapToProfileResponse(User user) ProfileResponse
    }

    User "1" *-- "many" ProfileLink : has
    UserRepository ..> User : queries
    ProfileLinkRepository ..> ProfileLink : queries
    CustomUserDetailsService --> UserRepository : uses
    UserService --> UserRepository : uses
    UserService ..> RegisterRequest : consumes
    UserService ..> LoginRequest : consumes
    UserService ..> AuthResponse : produces
    ProfileService --> UserRepository : uses
    ProfileService --> ProfileLinkRepository : uses
    ProfileService ..> ProfileUpdateRequest : consumes
    ProfileService ..> ProfileResponse : produces
    ProfileResponse "1" *-- "many" ProfileLinkDTO : contains
    ProfileUpdateRequest "1" *-- "many" ProfileLinkDTO : contains

```



### Part 2: Social Feeds, Comments & Media Storage


```mermaid
classDiagram
    direction TB

    class Post {
        -Long id
        -User user
        -Hobby hobby
        -String content
        -String imageUrl
        -String videoUrl
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        -List~Like~ likes
        -List~Comment~ comments
        +onCreate() void
        +onUpdate() void
    }

    class Like {
        -Long id
        -User user
        -Post post
    }

    class Comment {
        -Long id
        -User user
        -Post post
        -Comment parent
        -List~Comment~ replies
        -String content
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        +onCreate() void
        +onUpdate() void
    }

    class CreatePostRequest {
        -Long hobbyId
        -String content
        -String imageUrl
        -String videoUrl
        +hasConflictingMedia() boolean
    }

    class PostResponse {
        -Long id
        -String username
        -String displayName
        -String avatarUrl
        -Long hobbyId
        -String hobbyName
        -String content
        -String imageUrl
        -String videoUrl
        -int likeCount
        -boolean likedByCurrentUser
        -int commentCount
        -List~CommentResponse~ comments
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
    }

    class CreateCommentRequest {
        -Long parentId
        -String content
    }

    class CommentResponse {
        -Long id
        -String username
        -String displayName
        -String content
        -LocalDateTime createdAt
        -List~CommentResponse~ replies
    }

    class PostRepository {
        <<Interface>>
        +findByUserOrderByCreatedAtDesc(User user) List~Post~
        +findByUserAndHobbyIsNullOrderByCreatedAtDesc(User user) List~Post~
        +findByUserAndHobbyOrderByCreatedAtDesc(User user, Hobby hobby) List~Post~
    }

    class LikeRepository {
        <<Interface>>
        +existsByUserAndPost(User user, Post post) boolean
        +deleteByUserAndPost(User user, Post post) void
        +countByPost(Post post) int
    }

    class CommentRepository {
        <<Interface>>
        +findByPostAndParentIsNullOrderByCreatedAtAsc(Post post) List~Comment~
        +countByPost(Post post) int
    }

    class PostService {
        -UserRepository userRepository
        -PostRepository postRepository
        -HobbyRepository hobbyRepository
        -LikeRepository likeRepository
        -CommentRepository commentRepository
        +createPost(String username, CreatePostRequest request) PostResponse
        +getPersonalPosts(String username) List~PostResponse~
        +getPostsByHobby(String username, Long hobbyId) List~PostResponse~
        +getAllPosts(String username) List~PostResponse~
        +updatePost(String username, Long postId, CreatePostRequest request) PostResponse
        +deletePost(String username, Long postId) void
        +toggleLike(String username, Long postId) int
        +addComment(String username, Long postId, CreateCommentRequest request) CommentResponse
        +deleteComment(String username, Long commentId) void
        -mapToPostResponse(Post post, User currentUser) PostResponse
        -mapToCommentResponse(Comment comment) CommentResponse
    }

    class CloudinaryService {
        -Cloudinary cloudinary
        +uploadImage(MultipartFile file) String
        +uploadVideo(MultipartFile file) String
        +deleteFile(String publicId) void
    }

    Post "1" *-- "many" Like : has
    Post "1" *-- "many" Comment : has
    Comment "1" *-- "many" Comment : replies
    PostRepository ..> Post : queries
    LikeRepository ..> Like : queries
    CommentRepository ..> Comment : queries
    PostService --> PostRepository : uses
    PostService --> LikeRepository : uses
    PostService --> CommentRepository : uses
    PostService ..> CreatePostRequest : consumes
    PostService ..> PostResponse : produces
    PostService ..> CreateCommentRequest : consumes
    PostService ..> CommentResponse : produces
    PostResponse "1" *-- "many" CommentResponse : contains
    CommentResponse "1" *-- "many" CommentResponse : contains
```


### Part 3: Portfolios & Projects Domain

```mermaid
classDiagram
    direction TB

    class ProjectStatus {
        <<Enumeration>>
        IN_PROGRESS
        COMPLETED
        ARCHIVED
    }

    class Project {
        -Long id
        -User user
        -String title
        -String description
        -ProjectStatus status
        -String projectUrl
        -String imageUrl
        -List~ProjectTool~ tools
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        +onCreate() void
        +onUpdate() void
    }

    class ProjectTool {
        -Long id
        -Project project
        -String name
    }

    class CreateProjectRequest {
        -String title
        -String description
        -ProjectStatus status
        -String projectUrl
        -String imageUrl
        -List~String~ tools
    }

    class UpdateProjectRequest {
        -String title
        -String description
        -ProjectStatus status
        -String projectUrl
        -String imageUrl
        -List~String~ tools
    }

    class ProjectResponse {
        -Long id
        -String title
        -String description
        -ProjectStatus status
        -String projectUrl
        -String imageUrl
        -List~String~ tools
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
    }

    class ProjectRepository {
        <<Interface>>
        +findByUserOrderByCreatedAtDesc(User user) List~Project~
        +findByUserAndStatusOrderByCreatedAtDesc(User user, ProjectStatus status) List~Project~
        +findByUserAndId(User user, Long id) Optional~Project~
    }

    class ProjectToolRepository {
        <<Interface>>
        +findByProject(Project project) List~ProjectTool~
        +deleteByProject(Project project) void
    }

    class ProjectService {
        -UserRepository userRepository
        -ProjectRepository projectRepository
        -ProjectToolRepository projectToolRepository
        +createProject(String username, CreateProjectRequest request) ProjectResponse
        +getProjects(String username) List~ProjectResponse~
        +getProjectsByStatus(String username, ProjectStatus status) List~ProjectResponse~
        +updateProject(String username, Long projectId, UpdateProjectRequest request) ProjectResponse
        +deleteProject(String username, Long projectId) void
        -saveTools(Project project, List~String~ toolNames) void
        -mapToProjectResponse(Project project) ProjectResponse
    }

    Project "1" *-- "many" ProjectTool : has
    Project --> ProjectStatus : uses
    ProjectRepository ..> Project : queries
    ProjectToolRepository ..> ProjectTool : queries
    ProjectService --> ProjectRepository : uses
    ProjectService --> ProjectToolRepository : uses
    ProjectService ..> CreateProjectRequest : consumes
    ProjectService ..> UpdateProjectRequest : consumes
    ProjectService ..> ProjectResponse : produces
```


### Part 4: Hobby Tracking & Third-Party API Catalogs

```mermaid
classDiagram
    direction TB

    class HobbyType {
        <<Enumeration>>
        MUSIC
        GAMES
        BOOKS
        MOVIES
        ANIME
        CUSTOM
    }

    class Hobby {
        -Long id
        -User user
        -HobbyType type
        -String name
        -String slug
        -int displayOrder
        -List~HobbyEntry~ entries
        +onCreate() void
        -generateSlug(String name) String
    }

    class HobbyEntry {
        -Long id
        -Hobby hobby
        -String externalId
        -String title
        -String coverImageUrl
        -String note
        -int displayOrder
    }

    class AddHobbyRequest {
        -HobbyType type
        -String name
    }

    class AddHobbyEntryRequest {
        -String externalId
        -String title
        -String coverImageUrl
        -String note
    }

    class HobbyResponse {
        -Long id
        -HobbyType type
        -String name
        -String slug
        -int displayOrder
        -List~HobbyEntryResponse~ entries
    }

    class HobbyEntryResponse {
        -Long id
        -String externalId
        -String title
        -String coverImageUrl
        -String note
        -int displayOrder
    }

    class SearchResultItem {
        -String externalId
        -String title
        -String coverImageUrl
        -String extraInfo
    }

    class HobbyRepository {
        <<Interface>>
        +findByUserOrderByDisplayOrderAsc(User user) List~Hobby~
        +findByUserAndId(User user, Long id) Optional~Hobby~
        +findByUserAndTypeAndName(User user, HobbyType type, String name) Optional~Hobby~
        +existsByUserAndTypeAndName(User user, HobbyType type, String name) boolean
        +countByUser(User user) int
    }

    class HobbyEntryRepository {
        <<Interface>>
        +findByHobbyOrderByDisplayOrderAsc(Hobby hobby) List~HobbyEntry~
        +findByHobbyAndId(Hobby hobby, Long id) Optional~HobbyEntry~
        +existsByHobbyAndExternalId(Hobby hobby, String externalId) boolean
        +countByHobby(Hobby hobby) int
    }

    class HobbyService {
        -UserRepository userRepository
        -HobbyRepository hobbyRepository
        -HobbyEntryRepository hobbyEntryRepository
        -Map~HobbyType, String~ CATALOGUED_NAMES$
        +addHobby(String username, AddHobbyRequest request) HobbyResponse
        +getHobbies(String username) List~HobbyResponse~
        +deleteHobby(String username, Long hobbyId) void
        +addEntry(String username, Long hobbyId, AddHobbyEntryRequest request) HobbyEntryResponse
        +updateEntry(String username, Long hobbyId, Long entryId, AddHobbyEntryRequest request) HobbyEntryResponse
        +deleteEntry(String username, Long hobbyId, Long entryId) void
        -mapToHobbyResponse(Hobby hobby) HobbyResponse
        -mapToHobbyEntryResponse(HobbyEntry entry) HobbyEntryResponse
    }

    class JikanSearchService {
        -WebClient webClient
        +search(String query) List~SearchResultItem~
    }

    class TmdbSearchService {
        -String apiKey
        -WebClient webClient
        +searchMovies(String query) List~SearchResultItem~
        +searchTv(String query) List~SearchResultItem~
        -search(String query, String path, String dateField, String titleField) List~SearchResultItem~
    }

    class GoogleBooksSearchService {
        -String apiKey
        -WebClient webClient
        +search(String query) List~SearchResultItem~
    }

    class LastFmSearchService {
        -String apiKey
        -WebClient webClient
        +search(String query) List~SearchResultItem~
        -extractLastFmImage(Map track) String
    }

    class RawgSearchService {
        -String apiKey
        -WebClient webClient
        +search(String query) List~SearchResultItem~
    }

    Hobby --> HobbyType : uses
    Hobby "1" *-- "many" HobbyEntry : has
    HobbyRepository ..> Hobby : queries
    HobbyEntryRepository ..> HobbyEntry : queries
    HobbyService --> HobbyRepository : uses
    HobbyService --> HobbyEntryRepository : uses
    HobbyService ..> AddHobbyRequest : consumes
    HobbyService ..> AddHobbyEntryRequest : consumes
    HobbyService ..> HobbyResponse : produces
    HobbyService ..> HobbyEntryResponse : produces
    HobbyResponse "1" *-- "many" HobbyEntryResponse : contains
    JikanSearchService ..> SearchResultItem : creates
    TmdbSearchService ..> SearchResultItem : creates
    GoogleBooksSearchService ..> SearchResultItem : creates
    LastFmSearchService ..> SearchResultItem : creates
    RawgSearchService ..> SearchResultItem : creates
```


### Part 5: API Controllers & Routing Layer

```mermaid
classDiagram
    direction TB

    class AuthController {
        -UserService userService
        +register(RegisterRequest request) ResponseEntity~AuthResponse~
        +login(LoginRequest request) ResponseEntity~AuthResponse~
    }

    class UploadController {
        -CloudinaryService cloudinaryService
        +uploadImage(MultipartFile file) ResponseEntity~Map~String, String~~
        +uploadVideo(MultipartFile file) ResponseEntity~Map~String, String~~
    }

    class ProfileController {
        -ProfileService profileService
        +getMyProfile(User currentUser) ResponseEntity~ProfileResponse~
        +updateMyProfile(User currentUser, ProfileUpdateRequest request) ResponseEntity~ProfileResponse~
        +getProfileByUsername(String username) ResponseEntity~ProfileResponse~
    }

    class ProjectController {
        -ProjectService projectService
        +createProject(User currentUser, CreateProjectRequest request) ResponseEntity~ProjectResponse~
        +getProjects(String username) ResponseEntity~List~ProjectResponse~~
        +getProjectsByStatus(String username, ProjectStatus status) ResponseEntity~List~ProjectResponse~~
        +updateProject(User currentUser, Long projectId, UpdateProjectRequest request) ResponseEntity~ProjectResponse~
        +deleteProject(User currentUser, Long projectId) ResponseEntity~Void~
    }

    class PostController {
        -PostService postService
        +createPost(User currentUser, CreatePostRequest request) ResponseEntity~PostResponse~
        +getAllPosts(String username) ResponseEntity~List~PostResponse~~
        +getPersonalPosts(String username) ResponseEntity~List~PostResponse~~
        +getPostsByHobby(String username, Long hobbyId) ResponseEntity~List~PostResponse~~
        +updatePost(User currentUser, Long postId, CreatePostRequest request) ResponseEntity~PostResponse~
        +deletePost(User currentUser, Long postId) ResponseEntity~Void~
        +toggleLike(User currentUser, Long postId) ResponseEntity~Integer~
        +addComment(User currentUser, Long postId, CreateCommentRequest request) ResponseEntity~CommentResponse~
        +deleteComment(User currentUser, Long commentId) ResponseEntity~Void~
    }

    class HobbyController {
        -HobbyService hobbyService
        +addHobby(User currentUser, AddHobbyRequest request) ResponseEntity~HobbyResponse~
        +getMyHobbies(User currentUser) ResponseEntity~List~HobbyResponse~~
        +getHobbiesByUsername(String username) ResponseEntity~List~HobbyResponse~~
        +deleteHobby(User currentUser, Long hobbyId) ResponseEntity~Void~
        +addEntry(User currentUser, Long hobbyId, AddHobbyEntryRequest request) ResponseEntity~HobbyEntryResponse~
        +updateEntry(User currentUser, Long hobbyId, Long entryId, AddHobbyEntryRequest request) ResponseEntity~HobbyEntryResponse~
        +deleteEntry(User currentUser, Long hobbyId, Long entryId) ResponseEntity~Void~
    }

    class SearchController {
        -RawgSearchService rawgSearchService
        -LastFmSearchService lastFmSearchService
        -GoogleBooksSearchService googleBooksSearchService
        -TmdbSearchService tmdbSearchService
        -JikanSearchService jikanSearchService
        +searchGames(String q) ResponseEntity~List~SearchResultItem~~
        +searchMusic(String q) ResponseEntity~List~SearchResultItem~~
        +searchBooks(String q) ResponseEntity~List~SearchResultItem~~
        +searchMovies(String q) ResponseEntity~List~SearchResultItem~~
        +searchTv(String q) ResponseEntity~List~SearchResultItem~~
        +searchAnime(String q) ResponseEntity~List~SearchResultItem~~
    }

    AuthController --> UserService : uses
    UploadController --> CloudinaryService : uses
    ProfileController --> ProfileService : uses
    ProjectController --> ProjectService : uses
    PostController --> PostService : uses
    HobbyController --> HobbyService : uses
    
    SearchController --> RawgSearchService : uses
    SearchController --> LastFmSearchService : uses
    SearchController --> GoogleBooksSearchService : uses
    SearchController --> TmdbSearchService : uses
    SearchController --> JikanSearchService : uses
```

### Part 6: Security, JWT & Configuration

```mermaid
classDiagram
    direction TB

    class JwtService {
        -String secret
        -long expiration
        -getSigningKey() SecretKey
        +generateToken(UserDetails userDetails) String
        +extractUsername(String token) String
        +isTokenValid(String token, UserDetails userDetails) boolean
        -isTokenExpired(String token) boolean
        -extractExpiration(String token) Date
        -extractClaim~T~(String token, Function~Claims, T~ claimsResolver) T
        -extractAllClaims(String token) Claims
    }

    class JwtAuthenticationFilter {
        -JwtService jwtService
        -CustomUserDetailsService userDetailsService
        #doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) void
    }

    class SecurityConfig {
        -String frontendUrl
        -CustomUserDetailsService userDetailsService
        -JwtAuthenticationFilter jwtAuthenticationFilter
        +securityFilterChain(HttpSecurity http) SecurityFilterChain
        +authenticationProvider() AuthenticationProvider
        +passwordEncoder() PasswordEncoder
        +authenticationManager(AuthenticationConfiguration config) AuthenticationManager
        +corsConfigurationSource() CorsConfigurationSource
    }

    class OncePerRequestFilter {
        <<Abstract>>
    }

    JwtAuthenticationFilter --|> OncePerRequestFilter : extends
    JwtAuthenticationFilter --> JwtService : uses
    JwtAuthenticationFilter --> CustomUserDetailsService : uses
    
    SecurityConfig --> JwtAuthenticationFilter : registers
    SecurityConfig --> CustomUserDetailsService : configures
    SecurityConfig ..> AuthenticationManager : produces
    SecurityConfig ..> PasswordEncoder : produces
```


