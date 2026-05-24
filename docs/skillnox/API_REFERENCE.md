# Skillnox API Design Reference & Endpoint Specifications

This document outlines the RESTful API endpoints and WebSocket architecture of the Skillnox platform. It serves as a comprehensive reference of our contract-first API design, authentication flows, inputs validation, and response envelopes.

---

## 1. Global API Architecture & Envelopes

### Base URL
All API requests are prefixed with `/api` (or `/api/v1` in production proxies).

### Authentication & Sessions
- **Authentication Strategy**: Cookie-based sessions backed by Redis store (clustered mode) or MemoryStore.
- **Session ID Cookie**: `sid` (HttpOnly, Secure, SameSite: Strict/Lax).
- **Default Expiry**: 2 Hours.
- **CORS Rules**: Credentials: `true`, dynamic origin validation.

### Standard Response Schemas

#### Success JSON Envelope
```json
{
  "success": true,
  "data": { ... }
}
```

#### Validation Error (400 Bad Request)
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid fields provided",
    "details": [
      {
        "field": "email",
        "issue": "Invalid email address format"
      }
    ]
  }
}
```

#### Server Error (500 Internal Server Error)
```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "message": "An unexpected error occurred on the server"
  }
}
```

---

## 2. Authentication APIs

### Register Student
- **Method**: `POST`
- **Path**: `/api/auth/register`
- **Request Body**:
  ```json
  {
    "username": "23K61A0501",
    "email": "student501@kitaghire.in",
    "firstName": "Siddharth",
    "lastName": "Kumar",
    "password": "SecurePassword123!",
    "studentId": "KITS-2023-0501",
    "year": 3,
    "branch": "CSE",
    "section": "A"
  }
  ```
- **Response (201 Created)**: Returns the user object, omitting password hashes, and initiates the session cookie.

---

### Login Session
- **Method**: `POST`
- **Path**: `/api/auth/login`
- **Request Body**:
  ```json
  {
    "username": "23K61A0501",
    "password": "SecurePassword123!"
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "id": "abc-123-xyz",
    "username": "23K61A0501",
    "email": "student501@kitaghire.in",
    "firstName": "Siddharth",
    "lastName": "Kumar",
    "role": "student",
    "studentId": "KITS-2023-0501",
    "year": 3,
    "branch": "CSE",
    "section": "A"
  }
  ```
- **Notes**: Login is case-insensitive for student usernames (e.g., uppercase roll numbers). If legacy Bcrypt hashes are detected, they are automatically re-hashed using Argon2 upon successful login.

---

### Logout Session
- **Method**: `POST`
- **Path**: `/api/auth/logout`
- **Response (200 OK)**: Clears cookie and invalidates session in Redis cache.

---

### Get Authenticated User
- **Method**: `GET`
- **Path**: `/api/auth/user`
- **Response (200 OK)**: Returns the current user profile.

---

### Bulk Import Students (Admin Only)
- **Method**: `POST`
- **Path**: `/api/admin/bulk-import-students`
- **Request Body**:
  ```json
  {
    "students": [
      {
        "username": "23K61A0502",
        "email": "student502@kitaghire.in",
        "firstName": "Varun",
        "lastName": "Tej",
        "password": "TempPassword99!",
        "role": "student",
        "studentId": "KITS-2023-0502",
        "year": 3,
        "branch": "CSE",
        "section": "A"
      }
    ]
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "importedCount": 1,
    "totalCount": 1,
    "errors": []
  }
  ```
- **Optimizations**: Uses parallel password hashing and bulk DB transactional insertions. Automatically invalidates the student list cache.

---

## 3. Contest Management APIs

### List All Contests
- **Method**: `GET`
- **Path**: `/api/contests`
- **Query Params**: `page`, `limit`
- **Response (200 OK)**: Array of contests targeted at or accessible to the requesting user.

---

### Create New Contest (Admin Only)
- **Method**: `POST`
- **Path**: `/api/contests`
- **Request Body**:
  ```json
  {
    "title": "Data Structures Midterm Exam",
    "description": "Midterm contest covering arrays, trees, and hash maps.",
    "startTime": "2026-05-25T09:00:00.000Z",
    "endTime": "2026-05-25T11:00:00.000Z",
    "duration": 120,
    "passPercentage": 40,
    "targetYear": 3,
    "targetBranch": "CSE",
    "targetSections": ["A", "B"],
    "isPublic": false,
    "randomizeQuestions": true,
    "randomizeOptions": true
  }
  ```
- **Response (201 Created)**: Returns the new contest object.

---

### Join Contest
- **Method**: `POST`
- **Path**: `/api/contests/:id/join`
- **Response (200 OK)**: Enrolls the student in the contest, creating their participant record. If question randomization is active, it generates and persists their unique `studentQuestionOrders` record.

---

### Log Tab Switch (Anti-Cheat Event)
- **Method**: `POST`
- **Path**: `/api/contests/:contestId/tab-switch`
- **Response (200 OK)**: Increments the `tabSwitches` violation counter in the database. Returns the total count. If the threshold is exceeded, triggers disqualification.

---

## 4. Problems & Submissions APIs

### List Contest Problems
- **Method**: `GET`
- **Path**: `/api/contests/:contestId/problems`
- **Response (200 OK)**: Returns the problems in their default or randomized order for the student.

---

### Submit Code Solution
- **Method**: `POST`
- **Path**: `/api/submissions`
- **Request Body**:
  ```json
  {
    "contestId": "contest-uuid-here",
    "problemId": "problem-uuid-here",
    "code": "def solve(n):\n    return n * n\n\nif __name__ == '__main__':\n    import sys\n    n = int(sys.stdin.read().strip())\n    print(solve(n))",
    "language": "python"
  }
  ```
- **Response (200 OK)**:
  ```json
  {
    "id": "submission-uuid-here",
    "status": "accepted",
    "score": 100,
    "executionTime": 25,
    "memoryUsage": 8,
    "testResults": [
      {
        "input": "5",
        "expectedOutput": "25",
        "actualOutput": "25",
        "passed": true,
        "executionTime": 8
      }
    ]
  }
  ```

---

### Execute Code (Test Run)
- **Method**: `POST`
- **Path**: `/api/execute-code`
- **Request Body**:
  ```json
  {
    "code": "public class Solution {\n    public static void main(String[] args) {\n        System.out.println(\"Hello World\");\n    }\n}",
    "language": "java",
    "problemId": "problem-uuid-here"
  }
  ```
- **Response (200 OK)**: Executes code against sample visible test cases and returns stdout/stderr immediately. Does not record a score in the DB.

---

### Fetch Contest Leaderboard
- **Method**: `GET`
- **Path**: `/api/contests/:contestId/leaderboard`
- **Response (200 OK)**: Returns sorted ranking array of participants. Coalesced via Singleflight to protect PostgreSQL from concurrent hits during exam closeout.

---

## 5. Socket.IO Real-time Events

### Connection Setup
Clients connect to `/socket.io/` namespace using sticky sessions and WebSocket protocol.

### Server-to-Client Broadcasts
1.  `contest:timer:update` (Object: `{ contestId, remainingSeconds }`): Broadcasts countdown ticks.
2.  `leaderboard:refresh` (Object: `{ contestId }`): Signals clients to pull fresh leaderboard data.
3.  `submission:processed` (Object: `{ submissionId, status, score }`): Updates candidates as executions finish in the sandbox.

---

## 6. Production Middleware Proof: Request Schema Validation

This Zod validation middleware is used in Express route handlers to intercept invalid payloads before they hit application logic, keeping logic clean and types tight:

```typescript
import { Request, Response, NextFunction } from "express";
import { AnyZodObject, ZodError } from "zod";

export const validateRequest = (schema: AnyZodObject) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      // Validate req.body, req.query, or req.params depending on schema setup
      const validated = await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      });
      
      // Inject sanitized values back to request
      req.body = validated.body;
      req.query = validated.query;
      req.params = validated.params;
      
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        return res.status(400).json({
          success: false,
          error: {
            code: "VALIDATION_ERROR",
            message: "Request payload failed schema validation",
            details: error.errors.map((err) => ({
              field: err.path.join("."),
              issue: err.message,
            })),
          },
        });
      }
      next(error);
    }
  };
};
```
