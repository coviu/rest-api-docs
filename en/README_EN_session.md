Clyde - Session access provided for api customers.
=====================================

Clyde provides a session based algebra for creating, and restricting access to coviu calls. The core metaphors used are

Session: A coviu call that occurs between two or more parties at a specified time, and has a finite duration.
Participants: Users who may participate in a coviu call.

Participants join a call by following a _session link_ in their browser, or mobile app. The _session link_
identifies the participant, including their name, optional avatar, and importantly their _role_. As such,
it is important that each person joining the call be issued a different _session link_, i.e. have a distinct
_participant_ created for them. A participant's _role_ identifies whether that user may access the call directly,
or if they are required the be _let in_ by an existing participant.


##  API

NOTE: Data identified as being of type `date string` is in the form specified by
RFC-1123. That is the human readable representation of a timestamp in the UTC timezone
of the form `EEE, dd MMM yyyy HH:mm:ss GMT`.
In javascript it can be constructed by using the `toUTCString` on a `Date` object,
```
new Date().toUTCString();
```

### Authorization

Coviu uses a number of common OAuth2 rfc6749 mechanisms for authenticating and authorizing an api client. The basic approach is to
follow one of the OAuth2 authorization flows, to be issued an access token and bearer token that may then be used for access to user
resources via the coviu api.

The most basic use case is an api client acting on behalf of the owner of the client. In this case the client may follow the OAuth2 `client_credentials` flow
to be issued a access and refresh tokens directly.

Api users will have been issued an `api_key` and `key_secret` pair.

#### Request access token with Resource Owner Credentials

https://tools.ietf.org/html/rfc6749#section-4.3

```
POST /v1/auth/token
Authorization: Basic Base64(api_key:key_secret)
Body: grant_type=client_credentials

Response: 200, application/json;UTF-8
{
  access_token: <A bearer token used for authenticating api requests>,
  refresh_token: <A refresh token that may be used to recover a new access token>,
  expires_in: <seconds until it expires>,
  token_type: "Bearer",
  scope: <issued scope associated with client>
}
```

### Refresh an access token

https://tools.ietf.org/html/rfc6749#section-6

```
POST /v1/auth/token
Authorization: Basic Base64(<clientId>:<clientPassword>)
Body: grant_type=refresh_token&refresh_token=<refresh_token>

Response: 200, application/json;UTF-8
{
  access_token: <A bearer token used for authenticating api requests>,
  refresh_token: <A refresh token that may be used to recover a new access token>,
  expires_in: <seconds until it expires>,
  token_type: "Bearer",
  scope: <issued scope associated with client>
}
```

### Create A Session

Create a new session. Note that the session start time must be in the
future, and the session end time must be after the session start time.

#### Method

`POST`

#### URL

`/v1/sessions`

#### Authorization

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

#### Request Body
```
{
  "session_name": [optional display name for the session],
  "start_time": [date string],
  "end_time": [date string],
  "picture": [optional url for room image],
  "participants": [{
    "display_name": [optional string for participant display name],
    "picture": [option url for participant avatar],
    "role": [*required* - "guest", or "host"],
    "state": [option content for client use, e.g. local user id of client]
   }, ...]
}
```

#### Example
```
{
  "session_name": "A test session with Dr. Who",
  "start_time": "Wed, 08 Jun 2016 04:07:11 GMT",
  "end_time": "Wed, 08 Jun 2016 04:17:11 GMT",
  "picture": "http://www.fillmurray.com/200/300",
  "participants": [
    {
      "display_name": "Dr. Who",
      "role": "host",
      "picture": "http://fillmurray.com/200/300",
      "state": "drwho1234"
    },
    {
      "display_name": "Dr. Who",
      "role": "guest",
      "picture": "http://fillmurray.com/200/300",
      "state": "drwho1234"
    }
  ]
}
```

### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "session_id": [server defined session id],
  "session_name": [optional display name for the session],
  "start_time": [date string],
  "end_time": [date string],
  "picture": [optional url for room image],
  "team_id": [server defined id for hosting team],
  "client_id": [server defined id for api client],
  "participants": [
    {
      "client_id": [server defined id for api client],
      "display_name": [optional string for participant display name],
      "entry_url": [string - the url used for accessing the session],
      "participant_id": [server defined id for participant],
      "picture": [option url for participant avatar],
      "role": [*required* - "guest", or "host"],
      "session_id": [server defined session id],
      "state": [option content for client use, e.g. local user id of client]
    }...]
}
```

#### Example Response
```
{
  "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
  "session_name": "A test session with Dr. Who",
  "start_time": "Wed, 08 Jun 2016 04:07:11 GMT",
  "end_time": "Wed, 08 Jun 2016 04:17:11 GMT",
  "picture": "http://www.fillmurray.com/200/300"
  "team_id": "780af7e5-7737-4ee1-9f91-ec2c86397b01",
  "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
  "participants": [
    {
      "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
      "display_name": "Dr. Who",
      "entry_url": "https://coviu.com/session/c30aabaa-b9e2-4644-a432-8e78624ead42",
      "participant_id": "c30aabaa-b9e2-4644-a432-8e78624ead42",
      "picture": "http://fillmurray.com/200/300",
      "role": "HOST",
      "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
      "state": "drwho1234"
    },
    {
      "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
      "display_name": "Dr. Who",
      "entry_url": "https://coviu.com/session/b58e8d15-a3e6-4310-8f1d-881fbfb71f00",
      "participant_id": "b58e8d15-a3e6-4310-8f1d-881fbfb71f00",
      "picture": "http://fillmurray.com/200/300",
      "role": "GUEST",
      "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
      "state": "drwho1234"
    }
  ],
}

```

#### Get A Session

Get a single session by id.

#### Method

`GET`

#### URL

`/v1/sessions/<session id>`

#### Authorization

`Authorization: Bearer <access_token>`

### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "session_id": [server defined session id],
  "session_name": [optional display name for the session],
  "start_time": [date string],
  "end_time": [date string],
  "picture": [optional url for room image],
  "team_id": [server defined id for hosting team],
  "client_id": [server defined id for api client],
  "participants": [
    {
      "client_id": [server defined id for api client],
      "display_name": [optional string for participant display name],
      "entry_url": [string - the url used for accessing the session],
      "participant_id": [server defined id for participant],
      "picture": [option url for participant avatar],
      "role": [*required* - "guest", or "host"],
      "session_id": [server defined session id],
      "state": [option content for client use, e.g. local user id of client]
    }...]
}
```

#### Example Response
```
{
  "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
  "session_name": "A test session with Dr. Who",
  "start_time": "Wed, 08 Jun 2016 04:07:11 GMT",
  "end_time": "Wed, 08 Jun 2016 04:17:11 GMT",
  "picture": "http://www.fillmurray.com/200/300"
  "team_id": "780af7e5-7737-4ee1-9f91-ec2c86397b01",
  "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
  "participants": [
    {
      "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
      "display_name": "Dr. Who",
      "entry_url": "https://coviu.com/session/c30aabaa-b9e2-4644-a432-8e78624ead42",
      "participant_id": "c30aabaa-b9e2-4644-a432-8e78624ead42",
      "picture": "http://fillmurray.com/200/300",
      "role": "HOST",
      "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
      "state": "drwho1234"
    },
    {
      "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
      "display_name": "Dr. Who",
      "entry_url": "https://coviu.com/session/b58e8d15-a3e6-4310-8f1d-881fbfb71f00",
      "participant_id": "b58e8d15-a3e6-4310-8f1d-881fbfb71f00",
      "picture": "http://fillmurray.com/200/300",
      "role": "GUEST",
      "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
      "state": "drwho1234"
    }
  ],
}

```

### Update A Session

It is possible to update some attributes of a session, namely the start and end time,
session name and image. A session that has already finished may not be changed.

#### Method

`PUT`

#### URL

`/v1/sessions/<session id>`

#### Authorization

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

#### Request Body
```
{
  "session_name": [optional display name for the session],
  "start_time": [date string],
  "end_time": [date string],
  "picture": [optional url for room image],
}
```

#### Example
```
{
  "start_time": "Wed, 08 Jun 2016 04:32:27 GMT",
  "end_time": "Wed, 08 Jun 2016 04:42:27 GMT",
  "picture": "http://fillmurray.com/400/600",
  "session_name": "new session name"
}
```
### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "session_id": [server defined session id],
  "session_name": [optional display name for the session],
  "start_time": [date string],
  "end_time": [date string],
  "picture": [optional url for room image],
  "team_id": [server defined id for hosting team],
  "client_id": [server defined id for api client],
  "participants": [
    {
      "client_id": [server defined id for api client],
      "display_name": [optional string for participant display name],
      "entry_url": [string - the url used for accessing the session],
      "participant_id": [server defined id for participant],
      "picture": [option url for participant avatar],
      "role": [*required* - "guest", or "host"],
      "session_id": [server defined session id],
      "state": [option content for client use, e.g. local user id of client]
    }...]
}
```
#### Example Response
```
{
  "team_id": "f1f19c89-0411-427f-af7b-c075c6f447bc",
  "client_id": "1db362e7-690c-475f-adc9-65b1cad8fd1a",
  "participants": [
    {
      "client_id": "1db362e7-690c-475f-adc9-65b1cad8fd1a",
      "display_name": "Dr. Who",
      "entry_url": "https://coviu.com/session/62e66c27-b815-4763-bf18-dcc7d0953f05",
      "participant_id": "62e66c27-b815-4763-bf18-dcc7d0953f05",
      "picture": "http://fillmurray.com/200/300",
      "role": "HOST",
      "session_id": "ca24e9af-3960-4ae9-b610-0edcf61fa0d7",
      "state": "drwho1234"
    }
  ],
  "session_id": "ca24e9af-3960-4ae9-b610-0edcf61fa0d7",
  "session_name": "new session name",
  "start_time": "Wed, 08 Jun 2016 04:32:27 GMT",
  "end_time": "Wed, 08 Jun 2016 04:42:27 GMT",
  "picture": "http://fillmurray.com/400/600"
}
```


### Get a page of sessions

Get a list of sessions, filtering by start and end time, page by page.

#### Method

`GET`

#### URL

`/v1/session`

#### Authorization

`Authorization: Bearer <access_token>`

#### URL Parameters

```
page=[Integer] - Optional, zero baseed indexex
page_size=[Integer] - Optional, number of entries to return
start_time=[date string] - Optional, include sessions whose start time fall after start_time, url encoded.
end_time=[date string] - Optional, include sessions whos end time falls before end_time, url_encoded.
```

#### Example
```
/sessions

/sessions?page=1&page_size=100&start_time=Wed%2C%2008%20Jun%202016%2004%3A24%3A29%20GMT&end_time=Wed%2C%2008%20Jun%202016%2004%3A44%3A29%20GMT
```
### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "content": [list of session structures],
  "page": [integer, which page],
  "page_size": [integer, number of sessions in page],
  "more": [boolean, there are more sessions in the next page]
}
```

#### Example Response
```
{
  "content": [
    {
      "team_id": "56aea3f0-bdc1-4d2a-a04b-5b7d31837afd",
      "client_id": "7b7b3792-c1dc-4bb3-ae40-2c5ebcf235c1",
      "participants": [
        {
          "client_id": "7b7b3792-c1dc-4bb3-ae40-2c5ebcf235c1",
          "display_name": "Dr. Who",
          "entry_url": "https://coviu.com/session/e181bbc9-9042-46dc-8f37-9fd1ced34787",
          "participant_id": "e181bbc9-9042-46dc-8f37-9fd1ced34787",
          "picture": "http://fillmurray.com/200/300",
          "role": "HOST",
          "session_id": "7b3a6e93-7533-4678-a4e8-ed9b23463592",
          "state": "drwho1234"
        },
        {
          "client_id": "7b7b3792-c1dc-4bb3-ae40-2c5ebcf235c1",
          "display_name": "Dr. Who",
          "entry_url": "https://coviu.com/session/7ee87418-e60d-4495-8465-da942ed6525b",
          "participant_id": "7ee87418-e60d-4495-8465-da942ed6525b",
          "picture": "http://fillmurray.com/200/300",
          "role": "GUEST",
          "session_id": "7b3a6e93-7533-4678-a4e8-ed9b23463592",
          "state": "drwho1234"
        }
      ],
      "session_id": "7b3a6e93-7533-4678-a4e8-ed9b23463592",
      "session_name": "A test session with Dr. Who",
      "start_time": "Wed, 08 Jun 2016 04:24:29 GMT",
      "end_time": "Wed, 08 Jun 2016 04:44:29 GMT",
      "picture": "http://www.fillmurray.com/200/300"
    }
  ],
  "page": 0,
  "page_size": 1,
  "more": false
}
```

### Cancel A Session

A session that is scheduled for the future may be canceled. No participants
will be able to join the session after it has been canceled, and it can not be
uncanceled once it has been canceled. A session that has already started
may not be canceled.

#### Method

`DELETE`

#### URL

`/v1/sessions/<session id>`

#### Authorization

`Authorization: Bearer <access_token>`

### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "ok": true
}
```

#### Example Response
```
{
  "ok": true
}
```

### Add a new participant to a session

It is sometimes useful to add a participant to a session after the session has been created.
It is possible to add a participant to a session that has already start, but not to a session
that has already finished.

#### Method

`POST`

#### URL

`/v1/sessions/<session id>/participants`

#### Authorization

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

#### Request Body

```
{
  "display_name": [optional string for entry display name],
  "picture": [optional url for participant avatar],
  "role": [*required* - "guest", or "host"],
  "state": [option content for client use, e.g. local user id of client]
}
```

#### Example

````
{
  "display_name": "Dr. Who",
  "role": "host",
  "picture": "http://fillmurray.com/200/300",
  "state": "drwho1234"
}
````

### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "client_id": [server defined id for api client],
  "display_name": [optional string for participant display name],
  "entry_url": [string - the url used for accessing the session],
  "participant_id": [server defined id for participant],
  "picture": [option url for participant avatar],
  "role": [*required* - "guest", or "host"],
  "session_id": [server defined session id],
  "state": [option content for client use, e.g. local user id of client]
}
```
#### Example Response

```
{
  "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
  "display_name": "Dr. Who",
  "entry_url": "https://coviu.com/session/c30aabaa-b9e2-4644-a432-8e78624ead42",
  "participant_id": "c30aabaa-b9e2-4644-a432-8e78624ead42",
  "picture": "http://fillmurray.com/200/300",
  "role": "HOST",
  "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
  "state": "drwho1234"
}
```

### Get Session Participants

Get the list of participants for a session.

#### Method

`GET`

#### URL

`/v1/sessions/<session id>/participants`

#### Authorization

`Authorization: Bearer <access_token>`

### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
[{
  "client_id": [server defined id for api client],
  "display_name": [optional string for participant display name],
  "entry_url": [string - the url used for accessing the session],
  "participant_id": [server defined id for participant],
  "picture": [option url for participant avatar],
  "role": [*required* - "guest", or "host"],
  "session_id": [server defined session id],
  "state": [option content for client use, e.g. local user id of client]
},...]
```

### Get a specific participant

Get a single participant by its id.

#### Method

`GET`

#### URL

`/v1/participants/<participant id>`

#### Authorization

`Authorization: Bearer <access_token>`

### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "client_id": [server defined id for api client],
  "display_name": [optional string for participant display name],
  "entry_url": [string - the url used for accessing the session],
  "participant_id": [server defined id for participant],
  "picture": [option url for participant avatar],
  "role": [*required* - "guest", or "host"],
  "session_id": [server defined session id],
  "state": [option content for client use, e.g. local user id of client]
}
```

#### Example Response
```
{
  "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
  "display_name": "Dr. Who",
  "entry_url": "https://coviu.com/session/c30aabaa-b9e2-4644-a432-8e78624ead42",
  "participant_id": "c30aabaa-b9e2-4644-a432-8e78624ead42",
  "picture": "http://fillmurray.com/200/300",
  "role": "HOST",
  "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
  "state": "drwho1234"
}
```

### Update A Participant

It is possible to update a participant's name, role, picture, and state after it has been created.

#### Method

`PUT`

#### URL

`/v1/participants/<participant id>`

#### Authorization

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

#### Request Body

```
{
  "display_name": [optional string for participant display name],
  "picture": [option url for participant avatar],
  "role": [*required* - "guest", or "host"],
  "state": [option content for client use, e.g. local user id of client]
}
```

#### Example

````
{
  "display_name": "Dr. Who",
  "role": "host",
  "picture": "http://fillmurray.com/200/300",
  "state": "drwho1234"
}
````

### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "client_id": [server defined id for api client],
  "display_name": [optional string for participant display name],
  "entry_url": [string - the url used for accessing the session],
  "participant_id": [server defined id for participant],
  "picture": [option url for participant avatar],
  "role": [*required* - "guest", or "host"],
  "session_id": [server defined session id],
  "state": [option content for client use, e.g. local user id of client]
}
```
#### Example Response

```
{
  "client_id": "8cc48982-6180-480c-b28f-27dc066b11f4",
  "display_name": "Dr. Who",
  "entry_url": "https://coviu.com/session/c30aabaa-b9e2-4644-a432-8e78624ead42",
  "participant_id": "c30aabaa-b9e2-4644-a432-8e78624ead42",
  "picture": "http://fillmurray.com/200/300",
  "role": "HOST",
  "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
  "state": "drwho1234"
}
```

### Cancel A Participant

Cancel a session participant. The participant will no longer be able to enter the session.
Once a participant has been canceled, it may not be uncanceled. A participant may not
be canceled after a session has finished.

#### Method

`DELETE`

#### URL

`/v1/participants/<participant id>`

#### Authorization

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

### Response
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "ok": true
}
```
#### Example Response

```
{
  "ok": true
}
```
