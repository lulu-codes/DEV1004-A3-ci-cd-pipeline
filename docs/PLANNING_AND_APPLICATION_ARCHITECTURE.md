# Containerised Application Architecture

This document provides a diagram and explanations of the containerised application architecture for "The Century Screening Room" app, including justification for each components and explanations of how components interact with each other.

## Application Overview

**The Full MERN Stack (MongoDB, Express.js, React, Node.js)**

- **MongoDB** (Database)
  - The database to store all data (users, movies, ratings, friendships)
  - NoSQL database (documents and flexible schema design)

- **Express.js** (Backend framework)
  - Node.js web application framework
  - Handles API requests, routing, middleware, authentication, and business logic
  - Our API server

- **React** (Frontend framework)
  - JavaScript library for building user interfaces
  - Single page application: component-based architecture used to create dynamic and responsive UI
  - Communicates with backend API to fetch and display data
  - Runs in the user's browser (client-side application)

- **Node.js** (Runtime environment)
  - Runs Javascript on the server
  - Executes backend code (Express.js)
  - Handles incoming requests from frontend and interacts with MongoDB database

---

### Application Architecture Diagram

The diagram below shows how The Century Screening Room app runs when it’s containerised using Docker.
This architecture separates the application into three main containers (Frontend, Backend and MongoDB) that communicate with each other through a Docker network, all running on a Docker Host (your computer or server).

### Application Architecture Diagram

![Application Architecture Diagram](<images/Application-Architecture-Diagram.drawio%20(3).png>)

_Diagram 1: Application Architecture Diagram (Containerised)_

---

### Docker Host

The Docker Host is the physical or virtual machine where Docker is installed and running such as your local development computer, a server or a cloud instance.

**Why it matters:**

- It provides the foundation for all containers to run.
- It manages resources (CPU, memory, storage) and ensures that containers can communicate with each other and to external servers.

### Docker Network

Inside the Docker Host is a Docker network. This is a private virtual network created by Docker that allows containers to talk to each other without exposing everything to the outside world. Containers communicate with each other using service names (like `backend`, `frontend`, `mongodb`) instead of IP addresses.

**Why it matters:**

- Containers can find each other easily by name (eg. `backend`, `frontend`, `mongodb`)
- It provides isolation where containers can only talk to each other through this network
- It makes the application portable and works the same way on any machine

**How it works:**

- When you start containers with Docker Compose, they automatically join the same network and can communicate using their service names. For example, the backend can connect to MongoDB using `mongodb://mongodb:27017/movie_db` instead of needing to know the container's IP address.

The blue dotted rectangle inside the Docker Host represents the Docker Network containing all the three application containers for frontend, backend and database (mongodb).

---

### Frontend Container (React + Vite)

The Frontend Container runs the React application built with Vite. This is what users see and interact with on their web browser.

**Technologies Used:**

- **React 19** - JavaScript library for building user interfaces
- **Vite** - Build tool and development server
- **TanStack Query** - Manages server state, caching and automatic refetching
- **Axios** - HTTP client for making API calls to the backend
- **React Router** - Handles client-side routing (navigation between pages)

**Port:** 5000 (configured in `vite.config.js`)

**Image Name:** `reelcanon-frontend:dev-v1.0` (follows naming scheme: `reelcanon-{service}:{env}-{version}`)

**Why containerise it:**

- **Consistency:** Runs the same way on any machine (development/testing/ production)
- **Isolation:** Frontend dependencies don't conflict with other projects
- **Portability:** Easy to move between environments and lightweight
- **Reproducibility:** Anyone can run the exact same frontend setup

**How it communicates:**

1. Receives HTTP requests from the user's web browser
2. Makes API calls to the Backend Container using Axios
3. Uses the `VITE_API_URL` environment variable to know where the backend is located

**Environment Variables:**

- `VITE_API_URL` - The backend API URL (`http://localhost:3000`)
- `NODE_ENV` - Current environment (development, test, production)

---

### Backend Container (Node.js + Express)

The Backend Container runs the Express.js API server that handles all business logic, authentication and database operations.

**Technologies:**

- **Express.js** - Web application framework for Node.js
- **Mongoose** - MongoDB object modeling library (makes database queries easier)
- **JWT (JSON Web Tokens)** - Used for user authentication
- **Helmet** - Security middleware (adds security headers)
- **CORS** - Allows frontend to make requests from different origins
- **bcrypt** - Hashes passwords for secure storage

**Port:** 3000

**Image Name:** `reelcanon-backend:dev-v1.0`

**Why containerise it:**

- **Dependency Management:** All Node.js packages are isolated in the container
- **Environment Consistency:** Same Node.js version and dependencies everywhere
- **Security:** Can run as non-root user, reducing security risks
- **Scalability:** Easy to run multiple instances if needed

**How it communicates:**

- Receives API calls from the Frontend Container
- Makes database queries to the MongoDB Container using Mongoose
- Fetches movie metadata from the external OMDB API
- Can optionally connect to MongoDB Atlas for production
  - MongoDB Atlas is an optional cloud-hosted MongoDB service that can be used in production environments instead of the containerised MongoDB.
  - The DATABASE_URI environment variable can point to either:
    - Local containerised MongoDB: `mongodb://mongodb:27017/movie_db` (development)
    - MongoDB Atlas: `mongodb+srv://user:pass@cluster.mongodb.net/movie_db` (production)

**Environment Variables:**

- `JWT_SECRET_KEY` - Secret key for signing and verifying JWT tokens
- `DATABASE_URI` - MongoDB connection string (for production/MongoDB Atlas)
- `LOCAL_DB_URI` - Local MongoDB connection string (for development)
- `OMDB_API_KEY` - API key for fetching movie metadata from OMDB
- `NODE_ENV` - Current environment (affects which database URI is used)
- `TOKEN_HEADER_KEY` - Header name for JWT tokens (default: "authorization")

Instructions for setting up environment variables can be found here in section 5. [Add Environment Variables](backend/docs/INSTALLATION_AND_DEPLOYMENT.md)

---

### MongoDB Container (Database)

The MongoDB Container runs the MongoDB database server that stores all application data (users, movies, ratings, friendships).

**Technologies:**

- **MongoDB 7.0** - NoSQL document database
- **Database Name:** `movie_db`

**Port:** 27017 (MongoDB's default port)

**Image Name:** `mongo:7.0` (official MongoDB Docker image)

**Why containerise it:**

- **Easy Setup:** No need to install MongoDB on your computer
- **Data Persistence:** Uses Docker volumes to store data
- **Isolation:** Database is separate from other services
- **Version Control:** Can use specific MongoDB versions
- **Portability:** Database can be moved or backed up easily

**How it communicates:**

- Receives database queries from the Backend Container
- Stores and retrieves data using MongoDB's document-based storage
- Data persists in a Docker volume (not shown in diagram but configured in docker-compose.yml)

**Data Storage:**

- Uses Docker volumes to persist data
- Data survives container restarts and updates
- Can be backed up by copying the volume

---

### Environment Variables

Environment variables are configuration values that change how the application behaves without modifying the code, like settings that can be different for development, testing and production.

**Why they matter:**

- **Security:** Sensitive data (like API keys, passwords) are not hardcoded
- **Flexibility:** Same code works in different environments
- **Configuration:** Easy to change settings without rebuilding containers

**How they work:**

- Usually stored in `.env` files (for local development), but for security will store separately
- Passed to containers via Docker Compose `environment:` section
- Accessed in code using `process.env.VARIABLE_NAME` (backend) or `import.meta.env.VITE_VARIABLE_NAME` (frontend)

**Key Environment Variables:**

**Backend:**

- `JWT_SECRET_KEY` - Secret key for JWT token signing (must be strong and unique)
- `DATABASE_URI` - Production MongoDB connection string (MongoDB Atlas)
- `LOCAL_DB_URI` - Local MongoDB connection string (containerised MongoDB)
- `OMDB_API_KEY` - API key for OMDB movie metadata service
- `NODE_ENV` - Environment mode (development, test, production)
- `TOKEN_HEADER_KEY` - HTTP header name for JWT tokens

**Frontend:**

- `VITE_API_URL` - Backend API URL (e.g., `http://localhost:3000`)
- `NODE_ENV` - Environment mode

In the diagram, it shows environment variables with arrows pointing to Frontend and Backend containers, indicating they configure these services.

---

### Secrets Management and Storage

**Security Best Practice:** Sensitive data (API keys, passwords, secret keys) are never stored in the code repository, instead they are managed separately for security.

#### Local Development Storage

**Location:** `.env` files in the project root (on your local computer)

**Security:**

- These files are listed in `.gitignore` (already configured)
- **Never commit** these files to Git
- Each developer creates their own `.env` files locally
- Contains real secrets: `JWT_SECRET_KEY`, `DATABASE_URI`, `OMDB_API_KEY`

> Refer to Backend repo INSTALLATION_INSTRUCTIONS on how to set up

#### GitHub Secrets (for CI/CD)

**Location:** GitHub repository settings

**Purpose:** Used by GitHub Actions workflows for automated builds and tests

**Security:**

- Secrets are encrypted by GitHub
- Only accessible to GitHub Actions workflows
- Never visible in code, commits or pull requests
- Values are masked in workflow logs to not expose

#### Docker Compose Integration

**How secrets flow to containers:**

- Local Development: Docker Compose reads from .env.development file
- Passes values to containers via environment: section
- Containers receive secrets at runtime

**GitHub Actions (CI/CD):**

- Workflow reads from GitHub Secrets
- Passes to Docker build/run commands
- Same containers, different secret source

### GitHub Actions (Future Improvement)

- GitHub Actions workflows will be added to automate Docker builds and tests.
- Secrets will be stored in GitHub repository settings for CI/CD purposes.
- This is planned for implementation after core containerisation is complete.

---

### External Services

#### OMDB API (Movie Metadata)

OMDB (Open Movie Database) API is an external HTTP service that provides movie information (title, director, poster image, etc).

**Why it's external:** This service is not part of the application as it's a third-party API that is called to get movie data when seeding the database.

**How it's used:**

- Backend Container makes HTTP requests to OMDB API using `node-fetch`
- Requires `OMDB_API_KEY` environment variable for authentication
- Used during database seeding to populate movie information
- Called only when needed (not constantly running)

In the diagram it is shown as an external service outside the Docker Host, with a red dashed arrow from Backend Container labeled "Fetch movie metadata".

---

#### MongoDB Atlas (Optional Production Database)

MongoDB Atlas is a cloud-hosted MongoDB service and is an alternative to running MongoDB in a container. For this application architecture diagram, I have included it to reference it as an optional service used for production env.

**Why it's optional:**

- **Development:** Use containerised MongoDB (easier, no cloud account needed)
- **Production:** Can use MongoDB Atlas (managed service, automatic backups, scaling)

**How it works:**

- The `DATABASE_URI` environment variable points to either:
  - Containerised MongoDB: `mongodb://mongodb:27017/movie_db` (development)
  - MongoDB Atlas: `mongodb+srv://user:pass@cluster.mongodb.net/movie_db` (production)
- Same code works with both, just change the environment variable

---

### Data Flow (How Requests Work)

**Complete Request Cycle Example: User Views Movies**

1. **User Action:** User opens web browser and navigates to `localhost:5000`
   - Browser sends HTTP request to Frontend Container

2. **Frontend Processing:** Frontend Container receives request
   - React Router determines which page to show
   - Component uses TanStack Query to fetch movie data
   - TanStack Query calls Axios to make API request

3. **API Call:** Frontend Container makes API call to Backend Container
   - Axios sends HTTP request to `http://backend:3000/movies` (using service name `backend`)
   - Request includes JWT token in Authorization header (if user is logged in)

4. **Backend Processing:** Backend Container receives API request
   - Express.js routes request to appropriate handler
   - JWT middleware verifies authentication token
   - Controller function executes business logic

5. **Database Query:** Backend Container queries MongoDB Container
   - Mongoose sends query to `mongodb://mongodb:27017/movie_db`
   - MongoDB returns movie documents

6. **Response Flow:** Data flows back through the system
   - MongoDB → Backend (returns JSON data)
   - Backend → Frontend (returns JSON response)
   - Frontend → Browser (renders movies in React components)
   - Browser → User (displays movies on screen)

**Fetching Movie Metadata (Seeding)**

When seeding the database with movies:

1. Backend Container reads movie list from JSON file
2. For each movie, makes HTTP request to OMDB API
3. OMDB API returns movie metadata (title, poster etc.)
4. Backend Container saves complete movie data to MongoDB Container

---

### Component Interactions Summary

| Component             | Communicates With  | Method                      | Purpose                  |
| --------------------- | ------------------ | --------------------------- | ------------------------ |
| Web Browser           | Frontend Container | HTTP Request                | User views application   |
| Frontend Container    | Backend Container  | HTTP/REST (Axios)           | Fetch data, submit forms |
| Backend Container     | MongoDB Container  | Database Queries (Mongoose) | Store/retrieve data      |
| Backend Container     | OMDB API           | HTTP Request (node-fetch)   | Get movie metadata       |
| Backend Container     | MongoDB Atlas      | Database Queries (Optional) | Production database      |
| Environment Variables | All Containers     | Configuration               | Set up app behavior      |

---

### Justification for Containerisation

**Why containerise each component:**

1. **Frontend Container:**

- Ensures consistent Node.js and npm versions across all environments
- Isolates frontend dependencies from other projects
- Makes it easy to test different frontend versions
- Simplifies deployment as can just run the container

1. **Backend Container:**

- Ensures the same Node.js version and dependencies everywhere
- Security: Can run as non-root user
- Easy to scale: Run multiple backend instances if needed
- Consistent API behavior across environments

1. **MongoDB Container:**

- No need to install MongoDB on host machine
- Easy to reset database (just restart container)
- Data persistence through Docker volumes
- Can use different MongoDB versions for different projects

**Benefits of Containerised Architecture:**

- **Portability:** Works the same on Windows, Mac, Linux or cloud servers
- **Isolation:** Each service is independent, so if one fails, others keep running
- **Reproducibility:** Anyone can run the exact same setup
- **Scalability:** Easy to add more containers if needed
- **Development:** Quick setup by just running `docker-compose up`
- **Testing:** Can test with different configurations easily
- **Deployment:** Same containers work in development and production

---

### Network Architecture

**Internal Communication (Docker Network):**

- Containers communicate using service names: `frontend`, `backend`, `mongodb`
- All communication happens within the Docker Network (isolated from host)
- Ports are mapped to host for external access:
  - Frontend: `localhost:5000` = Container port 5000
  - Backend: `localhost:3000` = Container port 3000
  - MongoDB: `localhost:27017` = Container port 27017

**External Access:**

- Users access Frontend via `localhost:5000` (exposed to host)
- Backend API accessible at `localhost:3000` (exposed to host)
- MongoDB only accessible internally (not exposed, for security)

**Security Considerations:**

- MongoDB port not exposed to host (only accessible from other containers)
- Environment variables keep secrets out of code
- Containers run as non-root users when possible
- Network isolation prevents unauthorised access

---

## Development Workflow

### Starting the Application

**Backend:**

```bash
cd backend
npm install          # Install dependencies
npm run dev         # Start development server (port 3000)
```

**Frontend:**

```bash
cd frontend
npm install          # Install dependencies
npm run dev         # Start development server (port 5000)
```

### Making Changes

1. **Backend changes:**
   - Edit controller/model/route files
   - Server auto-restarts (if using `tsx watch`)
   - Test with API client (Bruno/Insomnia/Postman)

2. **Frontend changes:**
   - Edit component files
   - Vite hot-reloads automatically
   - See changes in browser immediately

### Testing

**Backend:**

```bash
npm test            # Run all tests
npm run test:watch # Watch mode
```

**Frontend:**

```bash
npm test            # Run all tests
npm run test:watch # Watch mode
```
