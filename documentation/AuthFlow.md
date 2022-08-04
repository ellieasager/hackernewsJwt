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
sequenceDiagram
    participant middleware as mw
    participant jwt
    participant users
    participant schema.resolvers as resolvers
    middleware->>jwt: ParseToken(tokenStr)
    middleware->>users: GetUserIdByUsername(username)
    middleware->>schema.resolvers: proceed(ctxWithUser)
```

**Note**: Authorization code only confirms that user exists in out DB and adds the username/userId data to the context object. A tokenString is parsed and verified, not user password.


## Creating a user
When a new user input is received `schema.resolvers.go` generates a token for that username.

```mermaid
sequenceDiagram
    participant schema.resolvers
    participant jwt
    schema.resolvers->>jwt: GenerateToken(username)
```


## Login/Authentication
After `middleware.go` extracts username from Authz header (if present), looks up userId by its username, it adds the user Obj to the context and passes it to `schema.resolvers.go`.

```mermaid
sequenceDiagram
    participant schema.resolvers
    participant users
    participant jwt
    schema.resolvers->>users: Authenticate(user)
    schema.resolvers->>jwt: GenerateToken(username)
```

**Note**: Authentication code confirms that user exists in DB, their password is correct and gives them a new tokenString.

