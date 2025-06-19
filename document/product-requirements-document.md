# Private Chat Application – Product Requirements Document (PRD)

## Overview
The Private Chat Application is a secure, real-time messaging platform designed for users to create, join, and communicate in private conversations. The application supports user authentication, conversation management, and admin functionalities for user oversight. Key goals include:
- Providing a seamless and secure chat experience.
- Ensuring scalability for multiple concurrent users.
- Supporting role-based access control (Admin vs. User).
- Delivering a modern, responsive UI with real-time messaging capabilities.

**Target Audience**: Individuals or small groups seeking private, secure communication channels.

**Scope**: Web-based application with potential for future mobile expansion.

## Tech Stack

### Backend
- **Language**: Java 21
- **Database**: PostgreSQL
- **Framework**: Spring
- **Library**: Spring Boot, Lombok, Spring Data JPA, Spring Security, Spring WebSocket, Spring Data Reactive Redis, Spring for Apache Kafka
- **Testing**: JUnit, Mockito, Testcontainers
- **Security**: BCrypt for password hashing, JWT for authentication
- **Message Queue**: Apache Kafka
- **Cache Database**: Redis

### Frontend
- **Build Tools**: Vite
- **Language**: TypeScript, HTML, CSS
- **Framework**: ReactJS
- **UI Component**: Shadcn/UI
- **Styling:** Tailwind CSS
- **State Management**: TanStack Query (server state), Zustand (client state)
- **Forms**: React Hook Form, Zod validation
- **API Calls**: Axios
- **Routing**: React Router
- **Real-time Communication**: Socket.IO
- **Testing**: Jest, React Testing Library, Cypress

### Other
- 

### Deployment
- **Deployment Tools**: Docker, AWS ECS
- **CI/CD**: GitHub Actions

## Features
### Login: Users authenticate using a unique username and password to obtain a JWT token
Use Case: A user enters their credentials and is redirected to the conversation list

### Sign Up: Users create an account with a unique username and password
Use Case: A new user registers, and the system creates a user with the 'User' role

### Create Conversation: Users create private conversations with a name and optional password
Use Case: A user creates a conversation and is automatically added as a member

### Join Conversation: Users join existing conversations by providing the conversation ID and password (if required)
Use Case: A user joins a conversation and can start messaging

### Messaging: Users send and receive real-time text messages within a conversation
Use Case: Messages are sent via Socket.IO and displayed in real-time
   
### Delete Conversation: Users can delete conversations
Use Case: An admin deletes a conversation, removing all associated messages
   
### Admin User Management: Admins view a list of all users with their details
Use Case: Admins monitor user activity via a management dashboard

### Future Features:
- Edit/delete messages
- File uploads
- Real-time notifications
- Search functionality

## Data Model Design

### User Table
| Field       | Type       | Notes                                                     |
|-------------|------------|-----------------------------------------------------------|
| id          | Integer    | Primary key, auto-increment                               |
| username    | String     | Unique, max 50 characters, required                       |
| password    | String     | Hashed using BCrypt, min 8 characters, required           |
| role        | String     | Enum: 'Admin' or 'User', default 'User'                   |
| created_at  | Timestamp  | Auto-generated, default current timestamp                 |

**Indexes**: `username` (unique index).

### Conversation Table
| Field       | Type       | Notes                                                     |
|-------------|------------|-----------------------------------------------------------|
| id          | Integer    | Primary key, auto-increment                               |
| name        | String     | Max 100 characters, required                              |
| password    | String     | Hashed using BCrypt, min 8 characters, optional           |
| user_ids    | Integer[]  | Array of User.id, references User table                   |
| created_at  | Timestamp  | Auto-generated, default current timestamp                 |

**Indexes**: `name` (optional, for search).

### ConversationParticipant Table
| Field            | Type     | Notes                                              |
|------------------|----------|----------------------------------------------------|
| id               | Integer  | Primary key, auto-increment                        |
| conversation_id  | Integer  | Foreign key, references Conversation.id, required  |
| user_id          | Integer  | Foreign key, references User.id, required          |
| joined_at        | Timestamp| Auto-generated, default current timestamp          |

**Indexes**: Composite index on `(conversation_id, user_id)`.

### Message Table
| Field             | Type       | Notes                                                     |
|-------------------|------------|-----------------------------------------------------------|
| id                | Integer    | Primary key, auto-increment                               |
| conversation_id   | Integer    | Foreign key, references Conversation.id, required         |
| user_id           | Integer    | Foreign key, references User.id, required                 |
| content           | String     | Max 1000 characters, required                             |
| created_at        | Timestamp  | Auto-generated, default current timestamp                 |

**Indexes**: `conversation_id` (for fast message retrieval).

## API Design

### Error Response
- 400 Bad Request
```json
{
    "error_message": "Request is invalid"
}
```

- 401 Unauthorized
```json
{
    "error_message": "Token is missing or invalid"
}
```

- 403 Forbidden
```json
{
  "error_message": "You are not allowed"
}
```

- 404 Not Found
```json
{
  "error_message": "Entity not found"
}
```

- 500 Internal Server Error
```json
{
  "error_message": "Something went wrong, please try again later"
}
```

### LOGIN
#### POST /login

#### Request Body:
```json
{
    "username",
    "password"
}
```

#### Response:
- 200 OK:
```json
{
    "token"
}
```
- 400 Bad Request: Invalid username or password format
- 400 Bad Request: Incorrect credentials

### SIGN_UP
#### POST /sign-up

#### Request Body: 
```json
{
    "username",
    "password"
}
```

#### Response:
- 200 OK
- 400 Bad Request: Username already exists or invalid input

### GET_USER_PROFILE
#### GET /profile

#### Headers:
- Authorization: Bearer <JWT_token>

#### Response:
- 200 OK:
```json
{
    "id",
    "username",
    "role"
}
```

### GET_CONVERSATION_LIST
#### GET /conversations

#### Headers:
- Authorization: Bearer <JWT_token>

#### Query Parameters:
- offset: Integer (default 0)
- limit: Integer (default 20)

#### Response:
- 200 OK:
```json
[{
    "id",
    "name",
    "total_users",
    "created_at"
}]
```

### GET_CONVERSATION_DETAIL
#### GET /conversations/{id}

#### Headers:
- Authorization: Bearer <JWT_token>

#### Response:
- 200 OK:
```json
{
    "id",
    "name",
    "users": [{
			  "id",
			  "username"
		}],
    "created_at"
}
```
- 404 Not Found: Conversation not found

### CREATE_CONVERSATION
#### POST /conversations

#### Headers:
- Authorization: Bearer <JWT_token>

#### Request Body:
```json
{
    "name",
    "password"
}
```

#### Response:
- 200 OK:
```json
{
    "id"
}
```
- 400 Bad Request: Invalid name or password

### JOIN_CONVERSATION
#### POST /conversations/{id}/join

#### Headers:
- Authorization: Bearer <JWT_token>

#### Request Body:
```json
{
    "password"
}
```

#### Response:
- 200 OK
- 400 Bad Request: Invalid conversation ID or password
- 400 Bad Request: User already in conversation

### GET_CONVERSATION_MESSAGE_LIST
#### GET /conversations/{id}/messages

#### Headers:
- Authorization: Bearer <JWT_token>

#### Query Parameters:
- offset: Integer (default 0)
- limit: Integer (default 20)

#### Response Body:
- 200 OK:
```json
[{
    "user": {
        "id",
        "username"
    },
    "content"
}]
```
- 404 Not Found: Conversation not found

### CREATE_CONVERSATION_MESSAGE
#### POST /conversations/{id}/messages

#### Headers:
- Authorization: Bearer <JWT_token>

#### Request Body:
```json
{
    "content"
}
```

#### Response:
- 200 OK:
```json
{
    "id"
}
```
- 400 Bad Request: Invalid content
- 404 Not Found: Conversation not found

### ADMIN_GET_USER_LIST
#### GET /admin/users

#### Headers:
- Authorization: Bearer <JWT_token>

#### Query Parameters:
- offset: Integer (default 0)
- limit: Integer (default 20)

#### Response:
- 200 OK:
```json
{
    "id",
    "username",
    "role",
    "created_at"
}
```
- 403 Forbidden: User is not an admin