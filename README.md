# DEV1004 Assessment 1: Containerise Application

## Multi-Container Docker Application for "The Century Screening Room"

### Overview

This repository contains the Docker configuration files to containerise and run the "The Century Screening Room" MERN stack application.
The application is fully containerised using Docker, with 3 separate containers for the frontend (React/Vite), backend (Express.js), and database (MongoDB).

The application repositories for the frontend and backend can be found here:

- Backend: <https://github.com/lulu-codes/DEV1003-Assessment02>
- Frontend: <https://github.com/lulu-codes/DEV1003-Assessment03>

---

## Repository Structure

```
DEV1004-A1-containerise-app/
├── backend/              # Node.js/Express API server
│   ├── Dockerfile        # Backend container configuration
│   ├── .dockerignore     # Files excluded from Docker build
│   └── src/              # Backend source code
├── frontend/             # React/Vite web application
│   ├── Dockerfile        # Frontend container configuration
│   ├── .dockerignore     # Files excluded from Docker build
│   └── src/              # Frontend source code
├── docs/                 # Documentation
│   ├── PLANNING_AND_APPLICATION_ARCHITECTURE.md
│   ├── CONTAINERISATION_INSTRUCTIONS.md
│   └── images/           # Application architecture diagram
├── docker-compose.yml    # Multi-container orchestration
├── docker-compose.test.yml  # Test environment orchestration
├── .env.example          # Environment variables example
└── README.md             # This file
```

---

## Application Architecture Diagram

![Application Architecture Diagram](docs/images/Application-Architecture-Diagram.drawio(3).png)

All containers communicate through a Docker bridge network (`century-screening-room-network`) using service names for internal communication.

Refer to the planning and application architecture documentation here for a detailed explanation of how the components interact: [Application Architecture Planning](docs/PLANNING_AND_APPLICATION_ARCHITECTURE.md).

## Architecture Overview - The Big Picture

<!-- https://github.com/lulu-codes/ISK1002-A2-Documentation-Website/blob/main/src/content/docs/architecture/overview.md -->

The Century Sreening Room app is a **3-tier architecture** application:

```
┌─────────────────────────────────────────────────────────────┐
│                         FRONTEND                            │
│                    (React Application)                      │
│                                                             │
│  • User Interface (UI Components)                           │
│  • State Management (Context + React Query)                 │
│  • Routing (React Router)                                   │
│  • HTTP Client (Axios)                                      │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  │ HTTP Requests (REST API)
                  │ JSON Data
                  │
┌─────────────────▼───────────────────────────────────────────┐
│                         BACKEND                             │
│                    (Express.js API)                         │
│                                                             │
│  • Routes (Endpoints)                                       │
│  • Controllers (Business Logic)                             │
│  • Middleware (Auth, Validation, Errors)                    │
│  • Models (Data Structure)                                  │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  │ Mongoose ODM
                  │ CRUD Operations
                  │
┌─────────────────▼───────────────────────────────────────────┐
│                        DATABASE                             │
│                      (MongoDB)                              │
│                                                             │
│  • Users Collection                                         │
│  • Movies Collection                                        │
│  • Ratings, Progress, Friends, etc.                         │
└─────────────────────────────────────────────────────────────┘
```

_Diagram of high level overview of the application_

---

## References and Resources

- [Dockerfile Overview](https://docs.docker.com/build/concepts/dockerfile/)

---

<!-- Troubleshooting git: Remove this -->
