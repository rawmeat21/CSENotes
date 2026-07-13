# Ketty Backend UML Diagram

  

This file contains UML views for the backend API.

  

## 1) UML Class Diagram (Domain + Core Services)

  
# Ketty Backend UML Diagram

  

This file contains UML views for the backend API.

  

## 1) UML Class Diagram (Domain + Core Services)

  

```mermaid

classDiagram

class User {

+Long id

+String username

+String email

+String password

+String displayName

+String bio

+String status

+String avatarUrl

+List<ProfileLink> profileLinks

+List<Hobby> hobbies

+List<Post> posts

+List<Project> projects

+LocalDateTime createdAt

+LocalDateTime updatedAt

+onCreate()

+onUpdate()

}

  

class ProfileLink {

+Long id

+User user

+String label

+String url

}

  

class Hobby {

+Long id

+User user

+HobbyType type

+String name

+String slug

+int displayOrder

+List<HobbyEntry> entries

+onCreate()

}

  

class HobbyEntry {

+Long id

+Hobby hobby

+String externalId

+String title

+String coverImageUrl

+String note

+int displayOrder

}

  

class Post {

+Long id

+User user

+Hobby hobby

+String content

+String imageUrl

+String videoUrl

+LocalDateTime createdAt

+LocalDateTime updatedAt

+List<Like> likes

+List<Comment> comments

+onCreate()

+onUpdate()

}

  

class Comment {

+Long id

+User user

+Post post

+Comment parent

+String content

+LocalDateTime createdAt

+LocalDateTime updatedAt

+List<Comment> replies

+onCreate()

+onUpdate()

}

  

class Like {

+Long id

+User user

+Post post

}

  

class Project {

+Long id

+User user

+String title

+String description

+ProjectStatus status

+String projectUrl

+String imageUrl

+LocalDateTime createdAt

+LocalDateTime updatedAt

+List<ProjectTool> tools

+onCreate()

+onUpdate()

}

  

class ProjectTool {

+Long id

+Project project

+String name

}

  

class UserService {

+register(RegisterRequest) AuthResponse

+login(LoginRequest) AuthResponse

}

  

class ProfileService {

+getMyProfile(String) ProfileResponse

+getProfileByUsername(String) ProfileResponse

+updateProfile(String, ProfileUpdateRequest) ProfileResponse

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

  

class HobbyService {

+addHobby(String, AddHobbyRequest) HobbyResponse

+getHobbies(String) List~HobbyResponse~

+deleteHobby(String, Long)

+addEntry(String, Long, AddHobbyEntryRequest) HobbyEntryResponse

+updateEntry(String, Long, Long, AddHobbyEntryRequest) HobbyEntryResponse

+deleteEntry(String, Long, Long)

}

  

class ProjectService {

+createProject(String, CreateProjectRequest) ProjectResponse

+getProjects(String) List~ProjectResponse~

+getProjectsByStatus(String, ProjectStatus) List~ProjectResponse~

+updateProject(String, Long, UpdateProjectRequest) ProjectResponse

+deleteProject(String, Long)

}

  

class CloudinaryService {

+uploadImage(MultipartFile) String

+uploadVideo(MultipartFile) String

+deleteFile(String)

}

  

class JwtService {

+generateToken(UserDetails) String

+extractUsername(String) String

+isTokenValid(String, UserDetails) boolean

}

  

class JwtAuthenticationFilter {

+doFilterInternal(HttpServletRequest, HttpServletResponse, FilterChain)

}

  

User "1" --> "0..*" ProfileLink : owns

User "1" --> "0..*" Hobby : owns

User "1" --> "0..*" Post : writes

User "1" --> "0..*" Project : owns

User "1" --> "0..*" Comment : writes

User "1" --> "0..*" Like : gives

  

Hobby "1" --> "0..*" HobbyEntry : contains

Hobby "1" --> "0..*" Post : categorizes

  

Post "1" --> "0..*" Like : has

Post "1" --> "0..*" Comment : has

  

Comment "0..1" --> "0..*" Comment : replies

  

Project "1" --> "0..*" ProjectTool : uses

  

UserService --> JwtService : uses

JwtAuthenticationFilter --> JwtService : uses

ProfileService --> User : loads

PostService --> Post : manages

HobbyService --> Hobby : manages

ProjectService --> Project : manages

```

  

## 2) UML Sequence Diagram (Authentication)

  

```mermaid

sequenceDiagram

participant Client

participant AuthController

participant UserService

participant AuthenticationManager

participant CustomUserDetailsService

participant JwtService

participant JwtAuthenticationFilter

participant SecurityContext

  

Client->>AuthController: POST /api/auth/login

AuthController->>UserService: login(request)

UserService->>AuthenticationManager: authenticate(username, password)

AuthenticationManager->>CustomUserDetailsService: loadUserByUsername(username)

UserService->>JwtService: generateToken(user)

UserService-->>AuthController: AuthResponse(token,...)

AuthController-->>Client: 200 OK + JWT

  

Client->>JwtAuthenticationFilter: Request with Bearer token

JwtAuthenticationFilter->>JwtService: extractUsername(token)

JwtAuthenticationFilter->>CustomUserDetailsService: loadUserByUsername(username)

JwtAuthenticationFilter->>JwtService: isTokenValid(token, user)

JwtAuthenticationFilter->>SecurityContext: setAuthentication(user)

```


