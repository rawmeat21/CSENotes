# Ketty API Layer UML (Split View)

The original single diagram was too wide because every controller, service, repository, and DTO sat on one canvas at once. Splitting it by feature domain keeps each diagram narrow and vertical — better for Obsidian's pane width.

## 1. Overview — Controllers to Services

```mermaid
classDiagram
    direction TB

    class AuthController
    class ProfileController
    class PostController
    class HobbyController
    class ProjectController
    class SearchController
    class UploadController

    class UserService
    class ProfileService
    class PostService
    class HobbyService
    class ProjectService
    class CloudinaryService
    class RawgSearchService
    class LastFmSearchService
    class GoogleBooksSearchService
    class TmdbSearchService
    class JikanSearchService

    AuthController --> UserService : uses
    ProfileController --> ProfileService : uses
    PostController --> PostService : uses
    HobbyController --> HobbyService : uses
    ProjectController --> ProjectService : uses
    SearchController --> RawgSearchService : uses
    SearchController --> LastFmSearchService : uses
    SearchController --> GoogleBooksSearchService : uses
    SearchController --> TmdbSearchService : uses
    SearchController --> JikanSearchService : uses
    UploadController --> CloudinaryService : uses
```

## 2. Auth Domain

```mermaid
classDiagram
    direction TB

    class AuthController {
        +register(request)
        +login(request)
    }

    class UserService {
        +register(RegisterRequest) AuthResponse
        +login(LoginRequest) AuthResponse
    }

    class UserRepository {
        +findByUsername(String)
        +findByEmail(String)
        +existsByUsername(String)
        +existsByEmail(String)
    }

    class AuthResponse
    class LoginRequest
    class RegisterRequest

    AuthController --> UserService : uses
    UserService --> UserRepository : reads/writes

    AuthController ..> AuthResponse : returns
    AuthController ..> LoginRequest : accepts
    AuthController ..> RegisterRequest : accepts
```

## 3. Profile Domain

```mermaid
classDiagram
    direction TB

    class ProfileController {
        +getMyProfile()
        +updateMyProfile(request)
        +getProfileByUsername(username)
    }

    class ProfileService {
        +getMyProfile(String) ProfileResponse
        +getProfileByUsername(String) ProfileResponse
        +updateProfile(String, ProfileUpdateRequest) ProfileResponse
    }

    class UserRepository {
        +findByUsername(String)
        +findByEmail(String)
    }

    class ProfileLinkRepository {
        +findByUser(User)
        +deleteByUser(User)
    }

    class ProfileResponse
    class ProfileUpdateRequest

    ProfileController --> ProfileService : uses
    ProfileService --> UserRepository : reads/writes
    ProfileService --> ProfileLinkRepository : reads/writes

    ProfileController ..> ProfileResponse : returns
    ProfileController ..> ProfileUpdateRequest : accepts
```

## 4. Post Domain

```mermaid
classDiagram
    direction TB

    class PostController {
        +createPost(request)
        +getAllPosts(username)
        +getPersonalPosts(username)
        +getPostsByHobby(username, hobbyId)
        +updatePost(postId, request)
        +deletePost(postId)
        +toggleLike(postId)
        +addComment(postId, request)
        +deleteComment(commentId)
    }

    class PostService {
        +createPost(String, CreatePostRequest) PostResponse
        +getAllPosts(String) List~PostResponse~
        +getPersonalPosts(String) List~PostResponse~
        +getPostsByHobby(String, Long) List~PostResponse~
        +updatePost(String, Long, CreatePostRequest) PostResponse
        +deletePost(String, Long)
        +toggleLike(String, Long) int
        +addComment(String, Long, CreateCommentRequest) CommentResponse
        +deleteComment(String, Long)
    }

    class UserRepository {
        +findByUsername(String)
    }

    class PostRepository {
        +findByUserOrderByCreatedAtDesc(User)
        +findByUserAndHobbyIsNullOrderByCreatedAtDesc(User)
        +findByUserAndHobbyOrderByCreatedAtDesc(User, Hobby)
    }

    class HobbyRepository {
        +findByUserOrderByDisplayOrderAsc(User)
    }

    class LikeRepository {
        +existsByUserAndPost(User, Post)
        +deleteByUserAndPost(User, Post)
        +countByPost(Post)
    }

    class CommentRepository {
        +findByPostAndParentIsNullOrderByCreatedAtAsc(Post)
        +countByPost(Post)
    }

    class PostResponse
    class CreatePostRequest
    class CommentResponse
    class CreateCommentRequest

    PostController --> PostService : uses
    PostService --> UserRepository : reads/writes
    PostService --> PostRepository : reads/writes
    PostService --> HobbyRepository : reads/writes
    PostService --> LikeRepository : reads/writes
    PostService --> CommentRepository : reads/writes

    PostController ..> PostResponse : returns
    PostController ..> CreatePostRequest : accepts
    PostController ..> CommentResponse : returns
    PostController ..> CreateCommentRequest : accepts
```

## 5. Hobby Domain

```mermaid
classDiagram
    direction TB

    class HobbyController {
        +addHobby(request)
        +getMyHobbies()
        +getHobbiesByUsername(username)
        +deleteHobby(hobbyId)
        +addEntry(hobbyId, request)
        +updateEntry(hobbyId, entryId, request)
        +deleteEntry(hobbyId, entryId)
    }

    class HobbyService {
        +addHobby(String, AddHobbyRequest) HobbyResponse
        +getHobbies(String) List~HobbyResponse~
        +deleteHobby(String, Long)
        +addEntry(String, Long, AddHobbyEntryRequest) HobbyEntryResponse
        +updateEntry(String, Long, Long, AddHobbyEntryRequest) HobbyEntryResponse
        +deleteEntry(String, Long, Long)
    }

    class UserRepository {
        +findByUsername(String)
    }

    class HobbyRepository {
        +findByUserOrderByDisplayOrderAsc(User)
        +findByUserAndId(User, Long)
        +findByUserAndTypeAndName(User, HobbyType, String)
        +existsByUserAndTypeAndName(User, HobbyType, String)
        +countByUser(User)
    }

    class HobbyEntryRepository {
        +findByHobbyOrderByDisplayOrderAsc(Hobby)
        +findByHobbyAndId(Hobby, Long)
        +existsByHobbyAndExternalId(Hobby, String)
        +countByHobby(Hobby)
    }

    class HobbyResponse
    class AddHobbyRequest
    class HobbyEntryResponse
    class AddHobbyEntryRequest

    HobbyController --> HobbyService : uses
    HobbyService --> UserRepository : reads/writes
    HobbyService --> HobbyRepository : reads/writes
    HobbyService --> HobbyEntryRepository : reads/writes

    HobbyController ..> HobbyResponse : returns
    HobbyController ..> AddHobbyRequest : accepts
    HobbyController ..> HobbyEntryResponse : returns
    HobbyController ..> AddHobbyEntryRequest : accepts
```

## 6. Project Domain

```mermaid
classDiagram
    direction TB

    class ProjectController {
        +createProject(request)
        +getProjects(username)
        +getProjectsByStatus(username, status)
        +updateProject(projectId, request)
        +deleteProject(projectId)
    }

    class ProjectService {
        +createProject(String, CreateProjectRequest) ProjectResponse
        +getProjects(String) List~ProjectResponse~
        +getProjectsByStatus(String, ProjectStatus) List~ProjectResponse~
        +updateProject(String, Long, UpdateProjectRequest) ProjectResponse
        +deleteProject(String, Long)
    }

    class UserRepository {
        +findByUsername(String)
    }

    class ProjectRepository {
        +findByUserOrderByCreatedAtDesc(User)
        +findByUserAndStatusOrderByCreatedAtDesc(User, ProjectStatus)
        +findByUserAndId(User, Long)
    }

    class ProjectToolRepository {
        +findByProject(Project)
        +deleteByProject(Project)
    }

    class ProjectResponse
    class CreateProjectRequest
    class UpdateProjectRequest

    ProjectController --> ProjectService : uses
    ProjectService --> UserRepository : reads/writes
    ProjectService --> ProjectRepository : reads/writes
    ProjectService --> ProjectToolRepository : reads/writes

    ProjectController ..> ProjectResponse : returns
    ProjectController ..> CreateProjectRequest : accepts
    ProjectController ..> UpdateProjectRequest : accepts
```

## 7. Search & Upload Domain

```mermaid
classDiagram
    direction TB

    class SearchController {
        +searchGames(q)
        +searchMusic(q)
        +searchBooks(q)
        +searchMovies(q)
        +searchTv(q)
        +searchAnime(q)
    }

    class UploadController {
        +uploadImage(file)
        +uploadVideo(file)
    }

    class RawgSearchService {
        +search(String) List~SearchResultItem~
    }

    class LastFmSearchService {
        +search(String) List~SearchResultItem~
    }

    class GoogleBooksSearchService {
        +search(String) List~SearchResultItem~
    }

    class TmdbSearchService {
        +searchMovies(String) List~SearchResultItem~
        +searchTv(String) List~SearchResultItem~
    }

    class JikanSearchService {
        +search(String) List~SearchResultItem~
    }

    class CloudinaryService {
        +uploadImage(MultipartFile) String
        +uploadVideo(MultipartFile) String
    }

    class SearchResultItem

    SearchController --> RawgSearchService : uses
    SearchController --> LastFmSearchService : uses
    SearchController --> GoogleBooksSearchService : uses
    SearchController --> TmdbSearchService : uses
    SearchController --> JikanSearchService : uses
    UploadController --> CloudinaryService : uses

    SearchController ..> SearchResultItem : returns
```

## Notes

- Controllers sit at the API boundary and expose routes.
- DTOs define the request and response shapes.
- Services hold business rules and orchestrate repository calls.
- Repositories are the database access layer.
- Search controllers call external search services rather than repositories.
- Some repositories/entities (e.g. `UserRepository`) show up in multiple domain diagrams — that's expected, since multiple features depend on the same core entity.