# Users

In general, users have the same features as other objects, such as the flexible schema. The differences are that user objects must have a username and password, the password is automatically encrypted and stored securely, and Parse Server enforces the uniqueness of the username and email fields.

Therefore you can manage users objects using the `createUser`, `user`, `users`, `updateUser`, `deleteUser`, and the generic operations with `_User` in the `className` argument.

Additionally, you can use the `signUp`, `logIn`, and `logOut` operations, which will be presented in the following sections.

## Signing Up

Signing up a new user differs from creating another object in that the `username` and `password` fields are required. The `password` field is handled differently than the others; it is encrypted with bcrypt when stored in the database and never returned to any client request.

You can ask Parse Server to [verify user email addresses]({{ site.baseUrl }}/parse-server/guide/#welcome-emails-and-email-verification) in your application settings. With this setting enabled, all new user registrations with an email field will generate an email confirmation at that address. You can check whether the user has verified their email with the `emailVerified` field.

To sign up a new user, use the `signUp` mutation. For example:

```graphql
mutation SignUp {
  signUp(fields: {
    username: "somedude"
    password: "Parse_3.5_Rocks!"
  }) {
    objectId
    createdAt
    sessionToken
  }
}
```

The code above should resolve to something similar to this:

```json
{
  "data": {
    "signUp": {
      "objectId": "A13obRUwDE",
      "createdAt": "2019-08-27T07:22:25.251Z",
      "sessionToken": "r:caca30fa5c68f0ce467e7790a7208ff7"
    }
  }
}
```

Note that, in addition to the regular `objectId`, and `createdAt` fields, a new field called `sessionToken` has been returned. This token can be used to authenticate subsequent operations as this user.

## Logging In

After you allow users to sign up, you need to let them log in to their account with a username and password in the future. To do this, use the `logIn` mutation:

```graphql
mutation LogIn {
  logIn(fields: { username: "somedude" password: "Parse_3.5_Rocks!" }) {
    objectId
    username
    sessionToken
    createdAt
    updatedAt
  }
}
```

The code above should resolve to something similar to this:

```json
{
  "data": {
    "logIn": {
      "objectId": "A13obRUwDE",
      "username": "somedude",
      "sessionToken": "r:28eeeed86ad96e88e120783b2ea612ef",
      "createdAt": "2019-08-27T07:22:25.251Z",
      "updatedAt": "2019-08-27T07:22:25.251Z"
    }
  }
}
```

Note that, when the user logs in, Parse Server generates a new `sessionToken` for future operations.

## Using Session Token

For authenticating an operation as a specific user, you need to pass the `X-Parse-Session-Token` header with its valid session token.

You can easily do this in the GraphQL Playground. There is an option called `HTTP HEADERS` in its bottom left side. Use this option to replace the default `X-Parse-Master-Key` header by a valid `X-Parse-Session-Token` header. You should have something like this:

<img alt="Session Token Header" data-echo="{{ '/assets/images/graphql/session-token.png' | prepend: site.baseurl }}"/>

After setting up the `X-Parse-Session-Token` header, any operation will run as this user. For example, you can run the code below to validate the session token and return its associated user:

```graphql
query Viewer {
  viewer {
    username
  }
}
```

The code above should resolve to something similar to this:

```json
{
  "data": {
    "viewer": {
      "username": "somedude"
    }
  }
}
```

## Logging Out

You can log out a user through the `logOut` mutation. You need to send the `X-Parse-Session-Token` header and run code like the below example:

```graphql
mutation LogOut {
  logOut {
    username
  }
}
```

The code above should resolve to something similar to this:

```json
{
  "data": {
    "logOut": {
      "username": "somedude"
    }
  }
}
```
