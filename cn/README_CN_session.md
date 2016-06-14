Coviu Session API - Session access provided for api customers.
=====================================

Coviu会话API提供了一个基于会话形式来创建，和限制进入COVIU通话。主要的核心概念如下：

* 会话：是指COVIU通话在指定的时间双方或多方之间发生，并具有有时效性。
* 参会者：要加入到COVIU通话的用户。

参会者在自己的浏览器或移动应用程序中的一个 _session link_ （绘画链接）加入通话。该 _session link_ 确认参会者，包括姓名，可选头像，以及重要的是他们 _role_ (用户角色）。因此，每一个加入COVIU通话的用户都会被分配一个不同的 _session link_ ，也就是说 _session link_ 被创建是根据 _participant_ 属性。每个参与者的角色属性 _role_ 标示着该用户连接的权限，直连或间接连接 _let in_ 。 _let in_ 模式是指新加入通话用户需要被准许。

##  API

注：标识为类型的数据`date string`是在所指定 RFC-1123格式。此为可读的UTC时间标记，格式为`EEE的，DD MMM YYYY HH：MM：SS GMT`。

例如javascript可以用`Date`类的`toUTCString`创建UTC时间。
```
new Date().toUTCString();
```

### Authorization授权

COVIU使用OAuth2 rfc6749机制来认证和授权API客户端。基本的方法是按照OAuth2的授权流程来分配访问令牌和承载令牌。承载令牌用于访问用户资源。API客户端会被分配一个`api_key`和一个`key_secret`。API客户端使用这对密匙依据OAuth2 `client_credentials`流程来访问和更新令牌信息。

#### Request access token with Client Credentials 使用客户端凭证请求访问令牌

https://tools.ietf.org/html/rfc6749#section-4.4

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

### Refresh an access token 刷新访问令牌

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

### Create A Session创建会话

Create a new session. Note that the session start time must be in the
future, and the session end time must be after the session start time.

#### Method方法

`POST`

#### URL

`/v1/sessions`

#### Authorization授权

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

#### Request Body请求格式
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

#### Example请求举例
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

### Response成功回复
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

#### Example Response成功回复举例
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

#### Get A Session获得会话

通过id来获得会话

#### Method方法

`GET`

#### URL

`/v1/sessions/<session id>`

#### Authorization授权

`Authorization: Bearer <access_token>`

### Response成功回复
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

#### Example Response成功回复举例
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
      "display_name": "Patient",
      "entry_url": "https://coviu.com/session/b58e8d15-a3e6-4310-8f1d-881fbfb71f00",
      "participant_id": "b58e8d15-a3e6-4310-8f1d-881fbfb71f00",
      "picture": "http://fillmurray.com/200/400",
      "role": "GUEST",
      "session_id": "73ef93f1-16a8-4c88-aad8-2dc92ea8bfa6",
      "state": "drwho1234"
    }
  ],
}

```

### Update A Session更新会话

已结束的会话是不可以更新的。尚未结束的会话可以更新会话的开始和结束时间，会话名称，以及图片。

#### Method方法

`PUT`

#### URL

`/v1/sessions/<session id>`

#### Authorization授权

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

#### Request Body请求格式
```
{
  "session_name": [optional display name for the session],
  "start_time": [date string],
  "end_time": [date string],
  "picture": [optional url for room image],
}
```

#### Example请求举例
```
{
  "start_time": "Wed, 08 Jun 2016 04:32:27 GMT",
  "end_time": "Wed, 08 Jun 2016 04:42:27 GMT",
  "picture": "http://fillmurray.com/400/600",
  "session_name": "new session name"
}
```
### Response成功回复
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
#### Example Response成功回复举例
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


### Get a page of sessions获得多个会话

获取会话列表，按照开始和结束时间筛选。

#### Method方法

`GET`

#### URL

`/v1/session`

#### Authorization授权

`Authorization: Bearer <access_token>`

#### URL Parameters参数

```
page=[Integer] - Optional, zero baseed indexex
page_size=[Integer] - Optional, number of entries to return
start_time=[date string] - Optional, include sessions whose start time fall after start_time, url encoded.
end_time=[date string] - Optional, include sessions whos end time falls before end_time, url_encoded.
```

#### Example举例
```
/sessions

/sessions?page=1&page_size=100&start_time=Wed%2C%2008%20Jun%202016%2004%3A24%3A29%20GMT&end_time=Wed%2C%2008%20Jun%202016%2004%3A44%3A29%20GMT
```
### Response成功回复
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

#### Example Response成功回复举例
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
          "display_name": "Patient",
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

### Cancel A Session取消会话

一个尚未发生的会话可以被取消。在一个会话被取消后，没有参会者可以进入该会话。并且该会话在被取消后不可以在被恢复。但是如果一个会话已经开始后，该会话不可以被取消。

#### Method方法

`DELETE`

#### URL

`/v1/sessions/<session id>`

#### Authorization授权

`Authorization: Bearer <access_token>`

### Response成功回复
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "ok": true
}
```

#### Example Response成功回复举例
```
{
  "ok": true
}
```

### Add a new participant to a session添加参会者到会话
添加参会者到一个已经创建的会话是可能的，但是如果会话已经结束，是不可能在添加参会者。

#### Method方法

`POST`

#### URL

`/v1/sessions/<session id>/participants`

#### Authorization授权

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

#### Request Body请求格式

```
{
  "display_name": [optional string for entry display name],
  "picture": [optional url for participant avatar],
  "role": [*required* - "guest", or "host"],
  "state": [option content for client use, e.g. local user id of client]
}
```

#### Example请求举例

````
{
  "display_name": "Dr. Who",
  "role": "host",
  "picture": "http://fillmurray.com/200/300",
  "state": "drwho1234"
}
````

### Response成功回复
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
#### Example Response成功回复举例

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

### Get Session Participants获得会话的多个参会者

获得会话的参会者列表。

#### Method方法

`GET`

#### URL

`/v1/sessions/<session id>/participants`

#### Authorization授权

`Authorization: Bearer <access_token>`

### Response成功回复
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

### Get a specific participant获得某一个特定的参会者

获得某一个特定的参会者信息，根据id。

#### Method方法

`GET`

#### URL

`/v1/participants/<participant id>`

#### Authorization授权

`Authorization: Bearer <access_token>`

### Response成功回复
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

#### Example Response成功回复举例
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

### Update A Participant更新某个参会者信息

可以更新参会者的名称，角色，图片和状态。

#### Method方法

`PUT`

#### URL

`/v1/participants/<participant id>`

#### Authorization授权

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

#### Request Body请求格式

```
{
  "display_name": [optional string for participant display name],
  "picture": [option url for participant avatar],
  "role": [*required* - "guest", or "host"],
  "state": [option content for client use, e.g. local user id of client]
}
```

#### Example请求举例

````
{
  "display_name": "Dr. Who",
  "role": "host",
  "picture": "http://fillmurray.com/200/300",
  "state": "drwho1234"
}
````

### Response成功回复
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
#### Example Response成功回复举例

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

### Cancel A Participant取消一个参会者

当一个会话的参会者被取消后，该参会者不能在进入该会话。参会者被取消后会话资格后不能被恢复。已经结束的会话中的参会者是不能被取消的。

#### Method方法

`DELETE`

#### URL

`/v1/participants/<participant id>`

#### Authorization授权

`Authorization: Bearer <access_token>`

#### Accepts

`application/json`

### Response成功回复
```
Status:  200 Ok
Content-Type: application/json
Body:
{
  "ok": true
}
```
#### Example Response回复举例

```
{
  "ok": true
}
```
