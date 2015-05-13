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
session: pdi.jwt.JwtSession = JwtSession({},{},None)

scala> // We can add a (key, value)
     | session = session + ("user", 1)
session: pdi.jwt.JwtSession = JwtSession({},{"user":1},None)

scala> // Or several of them
     | session = session ++ (("nbf", 1431520421), ("key", "value"), ("key2", 2), ("key3", 3))
session: pdi.jwt.JwtSession = JwtSession({},{"user":1,"nbf":1431520421,"key":"value","key2":2,"key3":3},None)

scala> // Also remove a key
     | session = session - "key"
session: pdi.jwt.JwtSession = JwtSession({},{"user":1,"nbf":1431520421,"key2":2,"key3":3},None)

scala> // Or several
     | session = session -- ("key2", "key3")
session: pdi.jwt.JwtSession = JwtSession({},{"user":1,"nbf":1431520421},None)

scala> // We can access a specific key
     | session.get("user")
res8: play.api.libs.json.JsValue = 1

scala> // Test if the session is empty or not
     | // (it is not here since we have several keys in the claimData)
     | session.isEmpty
res11: Boolean = false

scala> // Serializing the session is the same as encoding it as a JSON Web Token
     | val token = session.serialize
token: String = e30.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> // You can create a JwtSession from a token of course
     | JwtSession.deserialize(token)
res14: pdi.jwt.JwtSession = JwtSession({},{"user":1,"nbf":1431520421},None)

scala> // You could refresh the session to set its expiration in a few seconds from now
     | // but you need to set "session.maxAge" in your "application.conf" and since this
     | // is not a real Play application, we cannot do that, so here, the refresh will do nothing.
     | session = session.refresh
session: pdi.jwt.JwtSession = JwtSession({},{"user":1,"nbf":1431520421},None)
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
formatUser: play.api.libs.json.OFormat[User] = play.api.libs.json.OFormat$$anon$1@3b5f9625

scala> // Next, adding it to a new session
     | val session2 = JwtSession() + ("user", User(42, "Paul"))
session2: pdi.jwt.JwtSession = JwtSession({},{"user":{"id":42,"name":"Paul"}},None)

scala> // Finally, accessing it
     | session2.getAs[User]("user")
res21: Option[User] = Some(User(42,Paul))
```

## Play RequestHeader

You can extract a `JwtSession` from a `RequestHeader`.

```scala
scala> import pdi.jwt._, play.api.test.{FakeRequest, FakeHeaders}
import pdi.jwt._
import play.api.test.{FakeRequest, FakeHeaders}

scala> // Default JwtSession
     | FakeRequest().jwtSession
res23: pdi.jwt.JwtSession = JwtSession({},{},None)

scala> // What about some headers?
     | // (the default header for a JSON Web Token is "Authorization" and it should be prefixed by "Bearer ")
     | val request = FakeRequest().withHeaders(("Authorization", "Bearer " + session2.serialize))
request: play.api.test.FakeRequest[play.api.mvc.AnyContentAsEmpty.type] = GET /

scala> request.jwtSession
res26: pdi.jwt.JwtSession = JwtSession({},{"user":{"id":42,"name":"Paul"}},None)

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
res34: pdi.jwt.JwtSession = JwtSession({},{"user":{"id":42,"name":"Paul"}},None)

scala> // Setting a new empty JwtSession
     | result.withNewJwtSession
res36: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer e30.e30.))

scala> // Or from an existing JwtSession
     | result.withJwtSession(session2)
res38: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer e30.eyJ1c2VyIjp7ImlkIjo0MiwibmFtZSI6IlBhdWwifX0.))

scala> // Or from a JsObject
     | result.withJwtSession(Json.obj(("user", 1), ("key", "value")))
res40: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer e30.eyJ1c2VyIjoxLCJrZXkiOiJ2YWx1ZSJ9.))

scala> // Or from (key, value)
     | result.withJwtSession(("user", 1), ("key", "value"))
res42: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer e30.eyJ1c2VyIjoxLCJrZXkiOiJ2YWx1ZSJ9.))

scala> // We can add stuff to the current session (only (String, String))
     | result.addingToJwtSession(("key2", "value2"), ("key3", "value3"))
res44: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer e30.eyJ1c2VyIjp7ImlkIjo0MiwibmFtZSI6IlBhdWwifSwia2V5MiI6InZhbHVlMiIsImtleTMiOiJ2YWx1ZTMifQ.))

scala> // Or directly classes or objects if you have the correct implicit Writes
     | result.addingToJwtSession("user2", User(1, "Louis"))
res46: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer e30.eyJ1c2VyIjp7ImlkIjo0MiwibmFtZSI6IlBhdWwifSwidXNlcjIiOnsiaWQiOjEsIm5hbWUiOiJMb3VpcyJ9fQ.))

scala> // Removing from session
     | result.removingFromJwtSession("key2", "key3")
res48: play.api.mvc.Result = Result(200, Map(Authorization -> Bearer e30.eyJ1c2VyIjp7ImlkIjo0MiwibmFtZSI6IlBhdWwifX0.))

scala> // Refresh the current session
     | result.refreshJwtSession
res50: play.api.mvc.Result = Result(200, Map())

scala> // So, at the end, you can do
     | result.jwtSession.getAs[User]("user")
res52: Option[User] = Some(User(42,Paul))
```
