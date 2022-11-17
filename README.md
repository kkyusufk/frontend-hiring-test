# :phone: Aircall Frontend Hiring Test

This test is a part of our hiring process at [Aircall](https://aircall.io/) for the Frontend Engineer position.

*Feel free to have a look at our open positions [here](https://aircall.io/careers/) and to apply! Drop us a line with your LinkedIn/GitHub/Twitter at jobs@aircall.io.*

## Context

Aircall is on a mission to revolutionize the business phone industry! This test is about (re) building a small part of Aircall’s main application. We provide you with dedicated APIs, documented below, so that you'll only have to focus on the frontend apps. 

## Exercise

### Expectations

The objective is to build a small part of Aircall's main application, our phone app. We ask you to use our GraphQL or Rest API to display a list of calls and their details.

Of course, you can think about more features to add on top of that. Some are mandatory, some are seen as bonuses and some are a waste of time. While reviewing your test, we will of course focus first on the mandatory items, so please be pragmatic. You can refer to the breakdown below to have a better understanding of what's expected:

- the application must:
  - have a basic authention screen. We provide the authentication API. Please think twice about how you handle the authentication token, we'll deeply review it. 
  - display a paginated list of calls that you’ll retrieve from the API
  - display the details of a call, on a different page (different route), when the user clicks on a call from the list. Please note that we expect this view to display all the date related to the call itself, not an extract from the list. Improvements such as caching and error handling are more than welcome here.
  - provide basic pagination and filtering features. You can for instance filters calls on their call direction, or call types or whether the've been archived or not. Implementing one filter is enough, we don't expect you to build a complex filtering system allowing users to filter on every properties of a call. 
  - let users group calls by date, meaning that calls made on a same day should appear within the same section. 

- as a bonus, the application could:
  - let users archive one or several calls.
  - handle real-time events, meaning that whenever a call is archived or a note is being added to a call, these changes should be reflected on the UI immediately. To test that the app handle real-time events, we usually open 2 tabs. We perform an action, like archiving a call, from one tab and expect this action to be reflected in the UI in the second tab. 
  - be tested, through unit test and/or e2e tests.
  - be deployed somewhere, like on Netlify or on Vercel. You can even prove that you're familiar with CI/CD by building a pipeline building, testing and deploying the app. 
  - be written with Typescript.
  - be well documented.

Bonuses might look appealling but please make sure to implement mandatory features first as we'll focus our review on them. 

### Stack

The application can be built using any frontend framework/library such as React, Angular, Vue, or even vanilla JS. Even though we do use React on our main apps and Vue for our website, we won't judge you if you decide to use another tool. 

You can also choose whatever design system you'd like to build the application, or none if it better suits you. We provide you with our own sweet, lovely and homemade design system called tractor :tractor:
- Storybook: [here](http://tractor.aircall.io/)
- NPM Repository [here](https://www.npmjs.com/package/@aircall/tractor).

Again, we won't judge you on the design system you'll choose, only on the quality of the implementation, user interface and user experience.

## APIs

We’ve built 2 APIs for this test, so you can choose between a REST API or a GraphQL API. Both expose the same data, so it’s really about which one you prefer.

### Model

Both APIs use the same models.

Call Model

```
type Call {
  id: ID! // "unique ID of call"
  direction: String! // "inbound" or "outbound" call
  from: String! // Caller's number
  to: String! // Callee's number
  duration: Float! // Duration of a call (in seconds)
  is_archived: Boolean! // Boolean that indicates if the call is archived or not
  call_type: String! // The type of the call, it can be a missed, answered or voicemail.
  via: String! // Aircall number used for the call.
  created_at: String! // When the call has been made.
  notes: Note[]! // Notes related to a given call
}
```

Note Model

```
type Note {
  id: ID!
  content: String!
}
```

### GraphQL API

GraphQL URL (HTTP): https://frontend-test-api.aircall.dev/graphql

Subscription URL (Websocket - Real-time): wss://frontend-test-api.aircall.dev/websocket

#### Authentication

You must first authenticate yourself before requesting the API. You can do so by executing the Login mutation. See below.

#### Queries

All the queries are protected by a middleware that checks if the user is authenticated with a valid JWT.

`paginatedCalls` returns a list of paginated calls. You can fetch the next page of calls by changing the values of `offset` and `limit` arguments.

```
paginatedCalls(
  offset: Float = 0
  limit: Float = 10
): PaginatedCalls!

type PaginatedCalls {
  nodes: [Call!]
  totalCount: Int!
  hasNextPage: Boolean!
}
```

`activitiy` returns a single call if any, otherwise it returns null.

```
call(id: Float!): Call
```

`me` returns the currently authenticated user.

```
me: UserType!
```

```
type UserType {
  id: String!
  username: String!
}
```

#### Mutations

To be able to grab a valid JWT token, you need to execute the `login` mutation.

`login` receives the username and password as 1st parameter and return the access_token and the user identity.

```graphql
login(input: LoginInput!): AuthResponseType!

input LoginInput {
  username: String!
  password: String!
}

interface AuthResponseType {
  access_token: String!
  refresh_token: String!
  user: UserType!
}

interface DeprecatedAuthResponseType {
  access_token: String!
  user: UserType!
}
```

Once you are correctly authenticated you need to pass the Authorization header for all the next calls to the GraphQL API.

```JSON
{
  "Authorization": "Bearer <YOUR_ACCESS_TOKEN>"
}
```

**New Refresh Token Mutation (RECOMMENDED)**

Note that the `access_token` is only available for 10 minutes and the `refresh_token` is available for 1 hour. You need to ask for another fresh access token by calling the `refreshTokenV2` mutation passing along the `refresh_token` in the `Authorization` header like so:

```JSON
{
  "Authorization": "Bearer <REFRESH_TOKEN>"
}
```

```graphql
mutation refreshTokenV2: AuthResponseType!
```

**Deprecated Refresh Token Mutation**

Note that the `access_token` is only available for 10 minutes. You need to ask for another fresh token by calling the refreshToken mutation before the token gets expired passing along the `access_token` in the `Authorization` header.

Like so:

```JSON
{
  "Authorization": "Bearer <ACCESS_TOKEN>"
}
```

```graphql
mutation refreshToken: DeprecatedAuthResponseType!
```

You must use the new tokens for the new requests made to the API.

`archiveCall` as the name implies it either archive or unarchive a given call.If the call doesn't exist, it'll throw an error.

```
archiveCall(id: ID!): Call!
```

`addNote` create a note and add it prepend it to the call's notes list.

```
addNote(input: AddNoteInput!): Call!

input AddNoteInput {
  activityId: ID!
  content: String!
}
```

#### Subscriptions

To be able to listen for the mutations/changes done on a call, you can call the run the `onUpdateCall` subscription.

```graphql
onUpdateCall: Call!
```

Now, we whenever an call is changed via the `addNote` or `archiveCall` mutations, you will receive a subscription event informing you of this change.

_Don't forget to pass the Authorization header with the right access token in order to be able to listen for these changes_

### REST API

Base URL: https://frontend-test-api.aircall.dev

#### Authentication

You must first authenticate yourself before requesting the API. You can do so by sending a POST request to `/auth/login`. See below.

#### GET endpoints

All the endpoints are protected by a middleware that checks if the user is authenticated with a valid JWT.

`GET` `/calls` returns a list of paginated calls. You can fetch the next page of calls by changing the values of `offset` and `limit` arguments.

```
/calls?offset=<number>&limit=<number>
```

Response:
```
{
  nodes: [Call!]
  totalCount: Int!
  hasNextPage: Boolean!
}
```

`GET` `/calls/:id` return a single call if any, otherwise it returns null.

```
/calls/:id<uuid>
```

`GET` `/me` return the currently authenticated user.

```
/me
```

Response
```
{
  id: String!
  username: String!
}
```

#### POST endpoints

To be able to grab a valid JWT token, you need to call the following endpoint:

`POST` `/auth/login` receives the username and password in the body and returns the access_token and the user identity.

Body 

```JSON
{
  "username": String!,
  "password": String!
}
```

Response

```JSON
{
  "access_token": String!,
  "refresh_token": String!,
  "user": UserType!
}
```

Once you are correctly authenticated you need to pass the Authorization header for all the next calls to the REST API.

```JSON
{
  "Authorization": "Bearer <YOUR_ACCESS_TOKEN>"
}
```

**New Refresh Token Endpoint (RECOMMENDED)**

Note that the `access_token` is only available for 10 minutes and the `refresh_token` is available for 1 hour. You need to ask for another fresh access token by calling the `/auth/refresh-token-v2` endpoint passing along the `refresh_token` in the `Authorization` header 

Like so:

```JSON
`POST` `/auth/refresh-token-v2`

Header
{
  "Authorization": "Bearer <REFRESH_TOKEN>"
}
```

**Deprecated Refresh Token Endpoint**

Note that the `access_token` is only available for 10 minutes. You need to ask for another fresh token by calling the `/auth/refresh-token` endpoint before the token gets expired.passing along the `access_token` in the `Authorization` header.

Like so:

```JSON
`POST` `/auth/refresh-token`

Header
{
  "Authorization": "Bearer <ACCESS_TOKEN>"
}
```

You must use the new tokens for the new requests made to the API.

`POST` `/calls/:id/note` create a note and add it prepend it to the call's notes list.

```
`/calls/:id/note`

Body
{
  content: String!
}
```

It returns the `Call` as a response or an error if the note doesn't exist.

#### PUT endpoints

`PUT` `/calls/:id/archive` as the name implies it either archive or unarchive a given call. If the call doesn't exist, it'll throw an error.

```
PUT /calls/:id/archive
```

#### Real-time

In order to be aware of the changes done on a call, you need to subscribe to this private channel: `private-aircall` and listen for the following event: `update-call` which will return the call payload.

This event will be called each time you add a note or archive a call.

Note that, you need to use Pusher SDK in order to listen for this event.

Because this channel is private you need to authenticate first, to do that, you need to make 
- `APP_AUTH_ENDPOINT` point to: `https://frontend-test-api.aircall.dev/pusher/auth`
- set `APP_KEY` to `d44e3d910d38a928e0be`
- and set `APP_CLUSTER` to `eu`

#### Errors

The REST API can return a different type of errors:

`400` `BAD_REQUEST` error, happens when you provide some data which doesn't respect a given shape.

Example
```
{
  "statusCode": 400,
  "message": [
    "content must be a string",
    "content should not be empty"
  ],
  "error": "Bad Request"
}
```

`401` `UNAUTHORIZED` error, happens when the user is not authorized to perform an action or if his token is no longer valid

Example
```
{
  "statusCode": 401,
  "message": "Unauthorized"
}
```

`404` `NOT_FOUND` error, happens when the user requests a resource that no longer exists.

Example
```
{
  "statusCode": 404,
  "message": "The call does not exist!",
  "error": "Not Found"
}
```

## Submission

We don’t provide any boilerplate as a simple [CRA](https://create-react-app.dev/) will be enough here. Please create a repository and submit your technical test through this [form](https://forms.gle/1TG1snJoGgvPKox5A).

We'll try to review it in the next 48 hours and get back to you to talk about your code!

Contact us at jobs@aircall.io if you need more details.
