## JwtSession case class

Provides an API similar to the Play [Session](https://www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.mvc.Session) but using `JsValue` rather than `String` as values. It also separates `headerData` from `claimData` rather than having only one `data`.

### Basic usage

```scala
scala> import pdi.jwt.JwtSession
import pdi.jwt.JwtSession

scala> // Let's create a session, it will automatically assign a default header. No
     | // In your app, the default header would be generated from "application.conf" file
     | // but here, it will just use the default values (which are all empty)
     | var session = JwtSession()
session: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{},)

scala> // We can add a (key, value)
     | session = session + ("user", 1)
session: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":1},)

scala> // Or several of them
     | session = session ++ (("nbf", 1431520421), ("key", "value"), ("key2", 2), ("key3", 3))
session: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":1,"nbf":1431520421,"key":"value","key2":2,"key3":3},)

scala> // Also remove a key
     | session = session - "key"
session: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":1,"nbf":1431520421,"key2":2,"key3":3},)

scala> // Or several
     | session = session -- ("key2", "key3")
session: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":1,"nbf":1431520421},)

scala> // We can access a specific key
     | session.get("user")
res8: Option[play.api.libs.json.JsValue] = Some(1)

scala> // Test if the session is empty or not
     | // (it is not here since we have several keys in the claimData)
     | session.isEmpty
res11: Boolean = false

scala> // Serializing the session is the same as encoding it as a JSON Web Token
     | val token = session.serialize
token: String = eyJhbGciOiJub25lIn0.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> // You can create a JwtSession from a token of course
     | JwtSession.deserialize(token)
res14: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":1,"nbf":1431520421},)

scala> // You could refresh the session to set its expiration in a few seconds from now
     | // but you need to set "session.maxAge" in your "application.conf" and since this
     | // is not a real Play application, we cannot do that, so here, the refresh will do nothing.
     | session = session.refresh
session: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":1,"nbf":1431520421},)
```

### Using implicits

If you have implicit `Reads` and/or `Writes`, you can access and/or add data directly as case class or object.

```scala
scala> // First, creating the implicits
     | import play.api.libs.json.Json
import play.api.libs.json.Json

scala> import play.api.libs.functional.syntax._
import play.api.libs.functional.syntax._

scala> case class User(id: Long, name: String)
defined class User

scala> implicit val formatUser = Json.format[User]
formatUser: play.api.libs.json.OFormat[User] = play.api.libs.json.OFormat$$anon$1@22b9ca68

scala> // Next, adding it to a new session
     | val session2 = JwtSession() + ("user", User(42, "Paul"))
session2: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":{"id":42,"name":"Paul"}},)

scala> // Finally, accessing it
     | session2.getAs[User]("user")
res21: Option[User] = Some(User(42,Paul))
```

## Play RequestHeader

You can extract a `JwtSession` from a `RequestHeader`.

```scala
scala> import pdi.jwt._
import pdi.jwt._

scala> import pdi.jwt.JwtSession._
import pdi.jwt.JwtSession._

scala> import play.api.test.{FakeRequest, FakeHeaders}
import play.api.test.{FakeRequest, FakeHeaders}

scala> // Default JwtSession
     | FakeRequest().jwtSession
res23: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{},)

scala> // What about some headers?
     | // (the default header for a JSON Web Token is "Authorization" and it should be prefixed by "Bearer ")
     | val request = FakeRequest().withHeaders(("Authorization", "Bearer " + session2.serialize))
request: play.api.test.FakeRequest[play.api.mvc.AnyContentAsEmpty.type] = GET /

scala> request.jwtSession
res26: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":{"id":42,"name":"Paul"}},)

scala> // It means you can directly read case classes from the session!
     | // And that's pretty cool
     | request.jwtSession.getAs[User]("user")
res29: Option[User] = Some(User(42,Paul))
```

## Play Result

There are also implicit helpers around `Result` to help you manipulate the session inside it.

```scala
scala> // Several functions will need an implicit RequestHeader
     | // since this is the only way to read the headers of the Result
     | implicit val implRequest = request
implRequest: play.api.test.FakeRequest[play.api.mvc.AnyContentAsEmpty.type] = GET /

scala> // Let's begin by creating a Result
     | var result: play.api.mvc.Result = play.api.mvc.Results.Ok
result: play.api.mvc.Result = Result(200, Map())

scala> // We can already get a JwtSession from our implicit RequestHeader
     | result.jwtSession
res34: pdi.jwt.JwtSession = JwtSession({"alg":"none"},{"user":{"id":42,"name":"Paul"}},)

scala> // Setting a new empty JwtSession
     | result = result.withNewJwtSession
result: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer eyJhbGciOiJub25lIn0.e30.))

scala> // Or from an existing JwtSession
     | result = result.withJwtSession(session2)
result: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer eyJhbGciOiJub25lIn0.eyJ1c2VyIjp7ImlkIjo0MiwibmFtZSI6IlBhdWwifX0.))

scala> // Or from a JsObject
     | result = result.withJwtSession(Json.obj(("id", 1), ("key", "value")))
result: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer eyJhbGciOiJub25lIn0.eyJpZCI6MSwia2V5IjoidmFsdWUifQ.))

scala> // Or from (key, value)
     | result = result.withJwtSession(("id", 1), ("key", "value"))
result: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer eyJhbGciOiJub25lIn0.eyJpZCI6MSwia2V5IjoidmFsdWUifQ.))

scala> // We can add stuff to the current session (only (String, String))
     | result = result.addingToJwtSession(("key2", "value2"), ("key3", "value3"))
result: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer eyJhbGciOiJub25lIn0.eyJpZCI6MSwia2V5IjoidmFsdWUiLCJrZXkyIjoidmFsdWUyIiwia2V5MyI6InZhbHVlMyJ9.))

scala> // Or directly classes or objects if you have the correct implicit Writes
     | result = result.addingToJwtSession("user", User(1, "Paul"))
result: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer eyJhbGciOiJub25lIn0.eyJpZCI6MSwia2V5IjoidmFsdWUiLCJrZXkyIjoidmFsdWUyIiwia2V5MyI6InZhbHVlMyIsInVzZXIiOnsiaWQiOjEsIm5hbWUiOiJQYXVsIn19.))

scala> // Removing from session
     | result = result.removingFromJwtSession("key2", "key3")
result: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer eyJhbGciOiJub25lIn0.eyJpZCI6MSwia2V5IjoidmFsdWUiLCJ1c2VyIjp7ImlkIjoxLCJuYW1lIjoiUGF1bCJ9fQ.))

scala> // Refresh the current session
     | result = result.refreshJwtSession
result: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer eyJhbGciOiJub25lIn0.eyJpZCI6MSwia2V5IjoidmFsdWUiLCJ1c2VyIjp7ImlkIjoxLCJuYW1lIjoiUGF1bCJ9fQ.))

scala> // So, at the end, you can do
     | result.jwtSession.getAs[User]("user")
res44: Option[User] = Some(User(1,Paul))
```

## Play configuration

### Secret key

`play.http.secret.key`

> Default: none

The secret key is used to secure cryptographics functions. We are using the same key to sign Json Web Tokens so you don't need to worry about it.

### Session timeout

`play.http.session.maxAge`

> Default: none

Just like for the cookie session, you can use this key to specify the duration, in milliseconds or using the duration syntax (for example 30m or 1h), after which the user should be logout, which mean the token will no longer be valid. It means you need to refresh the expiration date at each request

### Signature algorithm

`play.http.session.algorithm`

> Default: HS256
>
> Supported: HMD5, HS1, HS224, HS256, HS384, HS512

You can specify which algorithm you want to use, among the supported ones, in order to create the signature which will assure you that nobody can actually change the token. You should probably stick with the default one or use HmacSHA512 for maximum security.

### Header name

`play.http.session.jwtName`

> Default: Authorization

You can change the name of the header in which the token should be stored. It will be used for both requests and responses.

### Response header name

`play.http.session.jwtResponseName`

> Default: none

If you need to have a different header for request and response, you can override the response header using this key.


### Token prefix

`play.http.session.tokenPrefix`

> Default: "Bearer "

Authorization header should have a prefix before the token, like "Basic" for example. For a JWT token, it should be "Bearer" (which is the default value) but you can freely change or remove it (using an empty string). The token prefix will be directly prepend before the token, so be sure to put any necessary whitespaces in it.
