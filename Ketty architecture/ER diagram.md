# Ketty ER Diagram

  

This ER diagram represents the backend database model used by the Ketty API.

  

```mermaid

erDiagram

KETTY_USERS {

BIGINT id PK

VARCHAR username UK

VARCHAR email UK

VARCHAR password

VARCHAR display_name

VARCHAR bio

VARCHAR status

VARCHAR avatar_url

TIMESTAMP created_at

TIMESTAMP updated_at

}

  

PROFILE_LINKS {

BIGINT id PK

BIGINT user_id FK

VARCHAR label

VARCHAR url

}

  

HOBBIES {

BIGINT id PK

BIGINT user_id FK

VARCHAR type

VARCHAR name

VARCHAR slug

INT display_order

}

  

HOBBY_ENTRIES {

BIGINT id PK

BIGINT hobby_id FK

VARCHAR external_id

VARCHAR title

VARCHAR cover_image_url

VARCHAR note

INT display_order

}

  

POSTS {

BIGINT id PK

BIGINT user_id FK

BIGINT hobby_id FK

TEXT content

VARCHAR image_url

VARCHAR video_url

TIMESTAMP created_at

TIMESTAMP updated_at

}

  

LIKES {

BIGINT id PK

BIGINT user_id FK

BIGINT post_id FK

}

  

COMMENTS {

BIGINT id PK

BIGINT user_id FK

BIGINT post_id FK

BIGINT parent_id FK

TEXT content

TIMESTAMP created_at

TIMESTAMP updated_at

}

  

PROJECTS {

BIGINT id PK

BIGINT user_id FK

VARCHAR title

TEXT description

VARCHAR status

VARCHAR project_url

VARCHAR image_url

TIMESTAMP created_at

TIMESTAMP updated_at

}

  

PROJECT_TOOLS {

BIGINT id PK

BIGINT project_id FK

VARCHAR name

}

  

KETTY_USERS ||--o{ PROFILE_LINKS : owns

KETTY_USERS ||--o{ HOBBIES : owns

KETTY_USERS ||--o{ POSTS : writes

KETTY_USERS ||--o{ PROJECTS : owns

  

HOBBIES ||--o{ HOBBY_ENTRIES : contains

HOBBIES o|--o{ POSTS : categorizes

  

POSTS ||--o{ LIKES : receives

POSTS ||--o{ COMMENTS : has

  

KETTY_USERS ||--o{ LIKES : gives

KETTY_USERS ||--o{ COMMENTS : writes

  

COMMENTS ||--o{ COMMENTS : replies_to

  

PROJECTS ||--o{ PROJECT_TOOLS : uses

```

  

## Notes

  

- `PK` means primary key.

- `FK` means foreign key.

- `UK` means unique constraint.

- Unique constraints in schema:

- `HOBBIES`: `(user_id, type, name)`

- `HOBBY_ENTRIES`: `(hobby_id, external_id)`

- `LIKES`: `(user_id, post_id)`

- `COMMENTS.parent_id` creates recursive comment threads (a comment can have replies).

- `POSTS.hobby_id` is optional, so a post can be personal (no hobby) or tied to one hobby.


Credit: Github Copilot