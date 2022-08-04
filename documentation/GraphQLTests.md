# Things To Try:

1. Try creating a link w/o auth token:
```
mutation {
  createLink(input: {title: "real link!", address: "www.graphql.org"}){
    user{
      name
    }
  }
}
```
This should return 
```
{
  "errors": [
    {
      "message": "access denied",
      "path": [
        "createLink"
      ]
    }
  ],
  "data": null
}
```

2. Now, create a user:
```
mutation {
  createUser(input: {username: "user1", password: "123"})
}
```
This should return 
```
{
  "data": {
    "createUser": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1ODE0NjAwODUsImlhdCI6MTU4MTQ1OTc4NX0.rYLOM123kSulGjvK5VP8c7S0kgk03WweS2VJUUbAgNA"
  }
}
```
The returnd value is your auth token. Copy it somewhere.

3. Try creating a link with auth token:
The same mutation is in step 1:
```
mutation {
  createLink(input: {title: "real link!", address: "www.graphql.org"}){
    user{
      name
    }
  }
}
```
But this time, from the bottom of the page select "HTTP Headers" button and fill it like this:
```
{
  "Authorization": "" // the auth token you have copied in step 2
}
```
The mutation should create a new link and not give any errors.

4. List all the links:
```
query {
  links {
    id
    title
    address
    user {
      id
      name
    }
  }
}
```
The link created in step 3 should be listed there.
