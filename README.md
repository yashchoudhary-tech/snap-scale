# Snap-Scale 📸

Snap-Scale is a high-performance, full-stack media sharing application. It allows users to upload images and videos, add captions, and view a global feed with dynamic image transformations.

## 🚀 Features

- **User Authentication**: Secure JWT-based authentication using `fastapi-users`.
- **Media Uploads**: Support for images and videos with automatic cloud storage.
- **Dynamic Transformations**: Real-time image processing and caption overlays via **ImageKit.io**.
- **Interactive Feed**: A responsive feed built with Streamlit, featuring owner-only deletion rights.
- **Async Architecture**: Fully asynchronous backend using FastAPI and SQLAlchemy.

## 🛠️ Tech Stack

- **Frontend**: Streamlit
- **Backend**: FastAPI
- **Database**: SQLAlchemy (Async) with PostgreSQL
- **Image Management**: ImageKit.io
- **Authentication**: FastAPI Users (JWT Strategy)

## 📋 Prerequisites

Before you begin, ensure you have the following:

- Python 3.9+
- An ImageKit.io account (for API keys)
- A configured `.env` file or environment variables for ImageKit credentials and Database URLs.

## 🔧 Setup & Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/yashchoudhary-tech/snap-scale.git
   cd snap-scale
   ```

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Configuration**:
   Ensure your `app/images.py` is configured with your ImageKit private and public keys.

4. **Run the Backend**:
   ```bash
   uvicorn app.app:app --reload
   ```
5. **Run the Frontend**:
   ```bash
   streamlit run frontend.py
   ```

## 📂 Project Structure

```text
├── app/
│   ├── app.py        # FastAPI server & route definitions
│   ├── db.py         # Database models & session management
│   ├── images.py     # ImageKit SDK initialization
│   ├── schemas.py    # Pydantic models for validation
│   └── users.py      # FastAPI Users configuration
├── frontend.py       # Streamlit UI logic
└── README.md
```

## 📝 Product Requirements Document (PRD)

### 1. Introduction
Snap-Scale is a user-friendly media sharing platform designed to allow users to easily upload, share, and view images and videos with accompanying captions. The application focuses on a seamless user experience for content creation and consumption.

### 2. User Stories / Features
-   **User Authentication**:
    -   As a new user, I want to be able to register for an account using my email and password.
    -   As an existing user, I want to be able to log in securely with my credentials.
    -   As a logged-in user, I want to be able to log out.
-   **Media Upload**:
    -   As a logged-in user, I want to upload image (PNG, JPG, JPEG) or video (MP4, AVI, MOV, MKV, WEBM) files.
    -   As a logged-in user, I want to add a text caption to my uploaded media.
    -   As a logged-in user, I want to see a confirmation when my media is successfully posted.
-   **Content Feed**:
    -   As a logged-in user, I want to view a feed of all posts, ordered by most recent.
    -   Each post in the feed should display the media, its caption, the uploader's email, and the creation date.
    -   As the owner of a post, I want to see an option to delete my post from the feed.
-   **Media Display**:
    -   Images should display with the caption overlaid directly on them.
    -   Videos should display with a consistent size and a blurred background, with the caption displayed below.

### 3. Non-Functional Requirements
-   **Performance**: Fast media uploads and feed loading.
-   **Security**: User authentication should be secure (JWT-based).
-   **Scalability**: Utilize cloud services (ImageKit) for media storage and processing to handle growing content.
-   **Usability**: Intuitive and easy-to-navigate interface.

## 📐 Technical Requirements Document (TRD)

### 1. Architecture Overview
Snap-Scale employs a client-server architecture with a Streamlit-based frontend and a FastAPI-based asynchronous backend. ImageKit.io is used for efficient cloud media storage and on-the-fly transformations.

### 2. Technology Choices
-   **Frontend**: Streamlit (Python web framework for rapid UI development).
-   **Backend**: FastAPI (Python web framework for building high-performance APIs).
-   **Database**: SQLAlchemy (ORM) with the `asyncpg` driver for PostgreSQL.
-   **Authentication**: `fastapi-users` library with a JWT (JSON Web Token) strategy.
-   **Media Management**: ImageKit.io SDK for uploading, storing, and transforming images/videos.
-   **Environment Management**: `python-dotenv` for managing environment variables.

### 3. Data Flow
-   **User Registration/Login**: Frontend sends credentials to FastAPI `/auth` endpoints. Backend authenticates and returns a JWT.
-   **Media Upload**: Frontend sends multipart form data (file + caption) and JWT to FastAPI `/upload` endpoint. Backend uploads the file to ImageKit, stores ImageKit URL and metadata in the database, and returns post details.
-   **Feed Retrieval**: Frontend requests posts from FastAPI `/feed` endpoint with JWT. Backend queries the database for posts and associated user emails, then returns a list of post data.
-   **Post Deletion**: Frontend sends a DELETE request to FastAPI `/posts/{post_id}` with JWT. Backend verifies ownership, deletes the post from the database, and returns a success message.

## 🗺️ Application Flow

1.  **Start Application**:
    -   User runs `uvicorn` for backend and `streamlit` for frontend.
2.  **Initial Load (Frontend)**:
    -   Streamlit checks `st.session_state` for existing `token` or `user` data.
    -   If no session, displays the **Login/Sign Up Page**.
3.  **Login/Sign Up**:
    -   User enters email and password.
    -   **Login**: Frontend sends `POST` request to `/auth/jwt/login`. If successful, stores `access_token` and fetches user details from `/users/me`, then transitions to **Feed Page**. If failed, displays error.
    -   **Sign Up**: Frontend sends `POST` request to `/auth/register`. If successful, prompts user to log in. If failed, displays error.
4.  **Main Application (Logged In)**:
    -   **Sidebar**: Displays user's email and a "Logout" button.
    -   **Navigation**: User can switch between "🏠 Feed" and "📸 Upload" pages.
5.  **Feed Page**:
    -   Frontend sends `GET` request to `/feed`.
    -   Displays all posts, each showing:
        -   Uploader's email and creation date.
        -   Media (image with overlaid caption, or video with caption below).
        -   A "🗑️ Delete" button if the current user is the post owner.
    -   **Delete Post**: If "Delete" is clicked, frontend sends `DELETE` request to `/posts/{post_id}`. If successful, refreshes the feed.
6.  **Upload Page**:
    -   User selects a media file and enters a caption.
    -   On "Share" button click, frontend sends `POST` request (multipart/form-data) to `/upload`.
    -   If successful, displays "Posted!" and redirects to the **Feed Page**. If failed, displays error.
7.  **Logout**:
    -   User clicks "Logout" in the sidebar.
    -   `st.session_state` is cleared, and the application returns to the **Login/Sign Up Page**.

## 🗄️ Backend Schema & Implementation

### 1. Database Schema (`app/db.py`)

#### User Model
Extends `SQLAlchemyBaseUserTableUUID` from `fastapi-users` for standard user fields:
-   `id`: UUID (Primary Key)
-   `email`: String (Unique, Indexed)
-   `hashed_password`: String
-   `is_active`: Boolean (Default: True)
-   `is_superuser`: Boolean (Default: False)
-   `is_verified`: Boolean (Default: False)
-   `posts`: Relationship to `Post` model (one-to-many).

#### Post Model
-   `id`: UUID (Primary Key, default generated by `uuid.uuid4`)
-   `user_id`: UUID (Foreign Key to `User.id`, not nullable)
-   `caption`: Text (Optional text for the media)
-   `url`: String (URL of the media on ImageKit.io, not nullable)
-   `file_type`: String ("image" or "video", not nullable)
-   `file_name`: String (Original file name on ImageKit.io, not nullable)
-   `created_at`: DateTime (Timestamp of creation, default `datetime.utcnow`)
-   `user`: Relationship to `User` model (many-to-one).

### 2. API Schemas (`app/schemas.py`)

Pydantic models for request/response validation and serialization:
-   `UserRead`: Schema for reading user data (e.g., `/users/me` response).
-   `UserCreate`: Schema for user registration (email, password).
-   `UserUpdate`: Schema for updating user data.
-   `PostCreate`: (Currently defined but not directly used as a request body for upload; the `/upload` endpoint uses `Form` and `File` directly).
-   `PostResponse`: (Currently defined but not directly used as a response model for individual posts; the `/feed` endpoint constructs a dictionary for each post).

### 3. Backend Implementation Details (`app/app.py`)

-   **FastAPI Application**: Initialized with a `lifespan` event to ensure database tables are created on application startup.
-   **Authentication Routers**: Integrates `fastapi-users` routers for JWT authentication, user registration, password reset, email verification, and user management.
-   **`/upload` Endpoint**:
    -   Handles media file (`UploadFile`) and caption (`Form`) submission.
    -   Authenticates the user using the `current_active_user` dependency.
    -   Temporarily saves the incoming file to the local filesystem.
    -   Uploads the temporary file to ImageKit.io, ensuring a unique file name and applying a tag.
    -   Creates a new `Post` record in the database, storing the ImageKit URL, file type, and other metadata.
    -   Includes robust error handling and ensures the temporary local file is always cleaned up in a `finally` block.
-   **`/feed` Endpoint**:
    -   Authenticates the user.
    -   Retrieves all `Post` records from the database, ordered by their creation date (most recent first).
    -   To display the uploader's email, it currently fetches all `User` records and creates a dictionary mapping user IDs to emails.
        -   *Note on Optimization*: For larger datasets, this approach can lead to an N+1 query problem (fetching all users separately from posts). A more efficient approach would involve using SQLAlchemy's `joinedload` or `selectinload` to eager-load the `User` relationship when querying `Post` objects.
    -   Constructs a list of dictionaries for each post, including a boolean `is_owner` flag to indicate if the current user owns the post.
-   **`/posts/{post_id}` Endpoint**:
    -   Handles `DELETE` requests for a specific post identified by its UUID `post_id`.
    -   Authenticates the user.
    -   Fetches the post from the database. If not found, raises `HTTPException(404)`.
    -   Verifies that the authenticated user is indeed the owner of the post. If not, raises `HTTPException(403)`.
    -   Deletes the post record from the database.
        -   *Note*: This endpoint currently only deletes the database record. The actual media file on ImageKit.io is *not* deleted.

### 4. ImageKit Integration (`app/images.py`)
-   Initializes the `ImageKit` SDK by loading API credentials from environment variables (typically defined in a `.env` file).
-   The initialized `imagekit` instance is then imported and used by the FastAPI application for media uploads.

### 5. Frontend Logic (`frontend.py`)
-   **Streamlit UI**: Provides the interactive user interface, managing application state (user `token` and `user` object) using `st.session_state`.
-   **Authentication Flow**: Implements the UI for user login and signup, making `requests` to the FastAPI backend's authentication endpoints.
-   **Media Upload UI**: Offers a file uploader widget and a text area for captions. It sends the selected media and caption as `multipart/form-data` to the backend's `/upload` endpoint.
-   **Feed Display**: Fetches posts from the backend's `/feed` endpoint and dynamically renders them.
    -   `create_transformed_url`: A utility function that constructs ImageKit URLs, incorporating transformations for dynamic content display.
    -   `encode_text_for_overlay`: Prepares captions for safe embedding into ImageKit transformation URLs (base64 encoding followed by URL encoding).
    -   Applies ImageKit transformations to images (e.g., caption overlay) and videos (e.g., resizing, blurred background) for a consistent and enhanced user experience.
-   **Post Deletion UI**: Displays a delete button for posts that belong to the currently logged-in user, sending `DELETE` requests to the backend's `/posts/{post_id}` endpoint.

```text
├── app/
│   ├── app.py        # FastAPI server & route definitions
│   ├── db.py         # Database models & session management
│   ├── images.py     # ImageKit SDK initialization
│   ├── schemas.py    # Pydantic models for validation
│   └── users.py      # FastAPI Users configuration
├── frontend.py       # Streamlit UI logic
└── README.md
```