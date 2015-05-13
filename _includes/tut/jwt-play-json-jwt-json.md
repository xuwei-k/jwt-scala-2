## JwtJson Object

### Basic usage

```scala
scala> import pdi.jwt.{JwtJson, JwtAlgorithm}, play.api.libs.json.Json
import pdi.jwt.{JwtJson, JwtAlgorithm}
import play.api.libs.json.Json

scala> val claim = Json.obj(("user", 1), ("nbf", 1431520421))
claim: play.api.libs.json.JsObject = {"user":1,"nbf":1431520421}

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> JwtJson.encode(claim)
res0: String = e30.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> val token = JwtJson.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> JwtJson.decodeJson(token, key)
res1: scala.util.Try[play.api.libs.json.JsObject] = Success({"user":1,"nbf":1431520421})

scala> JwtJson.decode(token, key)
res2: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None))
```

### Encoding

```scala
scala> val header = Json.obj(("typ", "JWT"), ("alg", "HS256"))
header: play.api.libs.json.JsObject = {"typ":"JWT","alg":"HS256"}

scala> // From just the claim to all possible attributes
     | JwtJson.encode(claim)
res4: String = e30.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> JwtJson.encode(claim, None, None)
res5: String = e30.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> JwtJson.encode(claim, key, algo)
res6: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> JwtJson.encode(claim, Option(key), Option(algo))
res7: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> // This one will actually be unsigned since there is no key provided, even if there is an algorithm inside the header
     | JwtJson.encode(header, claim)
res9: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> JwtJson.encode(header, claim, key)
res10: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> JwtJson.encode(header, claim, Option(key))
res11: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ
```

### Decoding

```scala
scala> // You can decode to JsObject
     | JwtJson.decodeJson(token, key)
res13: scala.util.Try[play.api.libs.json.JsObject] = Success({"user":1,"nbf":1431520421})

scala> JwtJson.decodeJson(token, Option(key))
res14: scala.util.Try[play.api.libs.json.JsObject] = Success({"user":1,"nbf":1431520421})

scala> JwtJson.decodeJsonAll(token, key)
res15: scala.util.Try[(play.api.libs.json.JsObject, play.api.libs.json.JsObject, Option[String])] = Success(({"typ":"JWT","alg":"HS256"},{"user":1,"nbf":1431520421},Some(VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ)))

scala> JwtJson.decodeJsonAll(token, Option(key))
res16: scala.util.Try[(play.api.libs.json.JsObject, play.api.libs.json.JsObject, Option[String])] = Success(({"typ":"JWT","alg":"HS256"},{"user":1,"nbf":1431520421},Some(VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ)))

scala> // Or to case classes
     | JwtJson.decode(token, key)
res18: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None))

scala> JwtJson.decode(token, Option(key))
res19: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None))

scala> JwtJson.decodeAll(token, key)
res20: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, Option[String])] = Success((JwtHeader(Some(HS256),Some(JWT),None),JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None),Some(VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ)))

scala> JwtJson.decodeAll(token, Option(key))
res21: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, Option[String])] = Success((JwtHeader(Some(HS256),Some(JWT),None),JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None),Some(VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ)))
```

### Formating

The project provides implicit reader and writer for both `JwtHeader` and `JwtClaim`

```scala
scala> import pdi.jwt._
import pdi.jwt._

scala> // Reads
     | Json.fromJson[JwtHeader](header)
res23: play.api.libs.json.JsResult[pdi.jwt.JwtHeader] = JsSuccess(JwtHeader(Some(HS256),Some(JWT),None),)

scala> Json.fromJson[JwtClaim](claim)
res24: play.api.libs.json.JsResult[pdi.jwt.JwtClaim] = JsSuccess(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None),)

scala> // Writes
     | Json.toJson(JwtHeader(JwtAlgorithm.HS256))
res26: play.api.libs.json.JsValue = {"typ":"JWT","alg":"HS256"}

scala> Json.toJson(JwtClaim("""{"user":1}""").issuedNow.expiresIn(10))
res27: play.api.libs.json.JsValue = {"exp":1431543013,"iat":1431543003,"user":1}
```
