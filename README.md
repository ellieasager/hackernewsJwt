# hackernews
A project to learn integration of go with gqlgen

A Hackernews clone with Go, GraphQL and JWT. 

The API should be able to handle registration, authentication, submitting links and getting list of links.

## Instructions

1. In MySQL db
```
CREATE DATABASE hackernews;
```

2. Cli
```
git clone https://github.com/ellieasager/hackernewsJwt
cd hackernewsJwt
printf '// +build tools\npackage tools\nimport _ "github.com/99designs/gqlgen"' | gofmt > tools.go
go mod tidy
```

3. In code:
In the file `internal/pkg/db/mysql/mysql.go` set username and password for the db connection in method `InitDB()`.

4. Cli: make sure to use your username and password when running the command below
```
migrate -database mysql://root:dbpassword@/hackernews -path internal/pkg/db/migrations/mysql up
```

5. Cli: `go run server.go`

6. In your browser go to http://localhost:8080/

- Try creating a user:
```
mutation {
  createUser(input: {username: "new user", password: "password"}){
    id,
    name
  }
}
```

- List existing users:
```
query {
  users {
    id
    name
  }
}
```

7. If you need to re-generate files, run 
```
go run github.com/99designs/gqlgen generate
```
