# hackernews
A project to learn integration of go with gqlgen

A Hackernews clone with Go and gqlgen. 

The API should be able to handle registration, authentication, submitting links and getting list of links.

## Instructions

1. In MySQL db
```
CREATE DATABASE hackernews;
```
2. Cli
```
git clone https://github.com/ellieasager/hackernews
cd hackernews
go mod init github.com/ellieasager/hackernews
go mod tidy
go get github.com/99designs/gqlgen
printf '// +build tools\npackage tools\nimport _ "github.com/99designs/gqlgen"' | gofmt > tools.go
go get -u github.com/go-sql-driver/mysql
go get github.com/golang-migrate/migrate/v4/cmd/migrate/
```

3. In code:
In the file `internal/pkg/db/mysql/mysql.go` set username and password for the db connection in method `InitDB()`.

4. Cli: make sure to use your username and password when running `migrate -database` below
```
go build -tags 'mysql' -ldflags="-X main.Version=1.0.0" -o $GOPATH/bin/migrate github.com/golang-migrate/migrate/v4/cmd/migrate/
migrate -database mysql://root:dbpass@/hackernews -path internal/pkg/db/migrations/mysql up
go mod tidy
go run github.com/99designs/gqlgen generate
go run server.go
```

5. In your browser go to http://localhost:8080/

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