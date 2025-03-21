# Clerk Cake Example

A demonstration of using Clerk JWT authentication with CakePHP 5. This example shows how to integrate Clerk's user authentication with a CakePHP backend API.

## Installation

After cloning this repository:

```bash
$ composer update
```

## Configuration

Set the required environment variables:

```bash
$ export CLERK_API_SECRET_KEY=your_secret_key

# Set authorized parties (comma-separated list of allowed origins)
$ export CLERK_AUTHORIZED_PARTIES=http://localhost:5173
```

Configure Clerk in `config/app_local.php`:

```php
'Clerk' => [
    'secret_key' => env("CLERK_API_SECRET_KEY"),
    'authorized_parties' => explode(',', env("CLERK_AUTHORIZED_PARTIES"))
]
```

## Running

Start the CakePHP development server:

```bash
$ bin/cake server
```

The API will be available at http://localhost:8765

## How It Works

Authentication flow:

```mermaid
sequenceDiagram
    autonumber
    participant Client as Client Browser
    participant CORS as CorsMiddleware
    participant Auth as AuthenticationMiddleware
    participant Clerk as ClerkAuthenticator
    participant Cont as ProtectedController

    Client->>+CORS: HTTP Request with JWT token
    CORS->>CORS: Set CORS headers
    CORS->>+Auth: Pass request to authentication middleware
    
    Auth->>Auth: Create AuthenticationService
    Auth->>+Clerk: authenticate(request)
    
    Clerk->>Clerk: Skip if OPTIONS request
    Clerk->>Clerk: Extract JWT from Authorization header
    Clerk->>Clerk: Verify JWT using Clerk SDK
    
    alt JWT Valid
        Clerk-->>Auth: Return SUCCESS with identity
        Auth->>Auth: Set identity in request
        Auth->>+Cont: Pass authenticated request
        
        Cont->>Cont: Access identity data
        Cont-->>Client: Return protected data
        Cont-->>Auth: Return response to Auth Middleware
        Auth-->>CORS: Return successful response to CORS Middleware
        CORS-->>Client: Return final response to Client
    else JWT Invalid or Missing
        Clerk-->>Auth: Return FAILURE result
        Auth-->>CORS: Authentication error
        CORS-->>Client: Return 401 Unauthorized
    end
```

## Frontend Integration

From a Clerk React frontend:

```javascript
import { useAuth } from '@clerk/clerk-react';

function ApiExample() {
  const { getToken } = useAuth();
  
  const fetchData = async () => {
    if (getToken) {
      // Get the userId or null if the token is invalid
      let res = await fetch("http://localhost:8765/clerk-jwt", {
          headers: {
              "Authorization": `Bearer ${await getToken()}`
          }
      });
      console.log(await res.json()); // {userId: 'the_user_id_or_null'}

      // Get gated data or a 401 Unauthorized if the token is not valid
      res = await fetch("http://localhost:8765/get-gated", {
          headers: {
              "Authorization": `Bearer ${await getToken()}`
          }
      });
      if (res.ok) {
          console.log(await res.json()); // {foo: "bar"}
      } else {
          // Token was invalid
      }
    }
  };
  
  return <button onClick={fetchData}>Fetch Data</button>;
}
```

## API Reference

Available endpoints:

- `GET /clerk-jwt` - Returns the authenticated user ID
- `GET /get-gated` - Returns protected data (requires authentication)

## Implementation Details

Key files:

- `src/Auth/ClerkAuthenticator.php` - Handles JWT validation
- `src/Controller/ProtectedController.php` - Contains protected API endpoints
- `src/Middleware/CorsMiddleware.php` - Manages CORS headers

## ⚠️ Production Warning

This project is not optimized for production and does not address all best practices that should be configured in a production app. It serves as a design template and should be given appropriate consideration before being used in production.

Issues to address for production use:
- CORS configuration is specific to development environments
- No HTTPS enforcement
- Minimal error handling (especially 401 errors)
- Using development server settings

For production deployment:
1. Configure proper CORS settings for your specific domains
2. Enforce HTTPS for all API communication
3. Implement comprehensive error handling
4. Use a production-grade web server instead of the built-in development server
