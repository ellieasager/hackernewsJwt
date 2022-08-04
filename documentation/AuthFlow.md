# Key Components

```mermaid
classDiagram
class jwt {
  jwt : +string GenerateToken(username)
  jwt : +string ParseToken(tokenStr)
}

class users {
  users : +string HashPassword(password)
  users : +string CheckPasswordHash(password, hash)
  users: +int GetUserIdByUsername(username)
  users: +bool Authenticate(User)
}
```
```mermaid
classDiagram
class middleware {
  middleware : +User ForContext(ctx)
}

class resolvers {
  resolvers : +tokenStr CreateUser(ctx, input)
  resolvers : +string RefreshToken(ctx, tokenStr)
  }
```

# Authorization in `middleware.go`

This code in `middleware.go` is executed every time before the request reaches the resolver. If Authz header is missing in request, the request is forwarded to resolver w/o any checks.


```mermaid
flowchart TD
    middleware.go --> B{ParseToken}
    B -->|absent| C[forwardAsIs]
    B -->|invalid| D[proceed]
    B -->|valid| E[getUserFromDb]
    E --> |notFound| F[proceed]
    E --> |found| G[addToRequest,proceed]
```

```mermaid
sequenceDiagram
    participant middleware as mw
    participant jwt
    participant users
    participant schema.resolvers as resolvers
    middleware->>jwt: ParseToken(tokenStr)
    middleware->>users: GetUserIdByUsername(username)
    middleware->>schema.resolvers: proceed(ctxWithUser)
```

**Note**: Authorization code only confirms that authToken is valid. If corresponding user exists in DB, it is added to the request's context. User password is not part of authorization.


# Creating an auth token

An auth token is created when an existing user logs in or when a new user is created.

## Creating a new user

When a new user input is received, `schema.resolvers.go` generates a token for that username.

```mermaid
sequenceDiagram
    participant schema.resolvers
    participant jwt
    schema.resolvers->>jwt: GenerateToken(username)
```

There is no code in this project that does anything with that token. The token is simply returned to user and the user is expected to add this auth header to subsequent requests:
```
{
  "Authorization": "" // the auth token you have received
}
```
See more about it in `GraphQLTests.md`


## Login/Authentication

After `middleware.go` extracts username from Authz header, it looks up userId by its username, adds the `User` object to the request's context and passes it to `schema.resolvers.go`.

`schema.resolvers.go` calls `users.Authenticate(user)` method that, in turn, checks the provided password with the hashed password from db. 

- If the check succeeds, `schema.resolvers.go` calls `jwt.GenerateToken(username)` to return a new auth token. (See diagram below).

- If the check fails, `WrongUsernameOrPasswordError` is returned.

```mermaid
sequenceDiagram
    participant outside
    participant schema.resolvers.go
    participant users
    participant jwt
    outside->>schema.resolvers.go: Login(username, pwd)
    schema.resolvers.go->>users: Authenticate(user)
    users->>users: CheckPasswordHash(pwd, hashedPwd)
    schema.resolvers.go->>jwt: GenerateToken(username)
    schema.resolvers.go->>outside: authToken
```

**Note**: Authentication code confirms that user exists in DB, their password is correct and gives them a new authToken.

