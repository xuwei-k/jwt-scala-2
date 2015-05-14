## JwtJson4s Object

### Basic usage

```scala
scala> import pdi.jwt.{JwtJson4s, JwtAlgorithm}, org.json4s._, org.json4s.JsonDSL.WithBigDecimal._, org.json4s.native.JsonMethods._
import pdi.jwt.{JwtJson4s, JwtAlgorithm}
import org.json4s._
import org.json4s.JsonDSL.WithBigDecimal._
import org.json4s.native.JsonMethods._

scala> val claim = JObject(("user", 1), ("nbf", 1431520421))
claim: org.json4s.JsonAST.JObject = JObject(List((user,JInt(1)), (nbf,JInt(1431520421))))

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> JwtJson4s.encode(claim)
res0: String = e30.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> val token = JwtJson4s.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> JwtJson4s.decodeJson(token, key)
res1: scala.util.Try[org.json4s.JObject] = Success(JObject(List((user,JInt(1)), (nbf,JInt(1431520421)))))

scala> JwtJson4s.decode(token, key)
res2: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None))
```

### Encoding

```scala
scala> val header = JObject(("typ", "JWT"), ("alg", "HS256"))
header: org.json4s.JsonAST.JObject = JObject(List((typ,JString(JWT)), (alg,JString(HS256))))

scala> // From just the claim to all possible attributes
     | JwtJson4s.encode(claim)
res4: String = e30.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> JwtJson4s.encode(claim, None, None)
res5: String = e30.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> JwtJson4s.encode(claim, key, algo)
res6: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> JwtJson4s.encode(claim, Option(key), Option(algo))
res7: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> // This one will actually be unsigned since there is no key provided, even if there is an algorithm inside the header
     | JwtJson4s.encode(header, claim)
res9: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> JwtJson4s.encode(header, claim, key)
res10: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> JwtJson4s.encode(header, claim, Option(key))
res11: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ
```

### Decoding

```scala
scala> // You can decode to JsObject
     | JwtJson4s.decodeJson(token, key)
res13: scala.util.Try[org.json4s.JObject] = Success(JObject(List((user,JInt(1)), (nbf,JInt(1431520421)))))

scala> JwtJson4s.decodeJson(token, Option(key))
res14: scala.util.Try[org.json4s.JObject] = Success(JObject(List((user,JInt(1)), (nbf,JInt(1431520421)))))

scala> JwtJson4s.decodeJsonAll(token, key)
res15: scala.util.Try[(org.json4s.JObject, org.json4s.JObject, Option[String])] = Success((JObject(List((typ,JString(JWT)), (alg,JString(HS256)))),JObject(List((user,JInt(1)), (nbf,JInt(1431520421)))),Some(VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ)))

scala> JwtJson4s.decodeJsonAll(token, Option(key))
res16: scala.util.Try[(org.json4s.JObject, org.json4s.JObject, Option[String])] = Success((JObject(List((typ,JString(JWT)), (alg,JString(HS256)))),JObject(List((user,JInt(1)), (nbf,JInt(1431520421)))),Some(VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ)))

scala> // Or to case classes
     | JwtJson4s.decode(token, key)
res18: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None))

scala> JwtJson4s.decode(token, Option(key))
res19: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None))

scala> JwtJson4s.decodeAll(token, key)
res20: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, Option[String])] = Success((JwtHeader(Some(HS256),Some(JWT),None),JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None),Some(VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ)))

scala> JwtJson4s.decodeAll(token, Option(key))
res21: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, Option[String])] = Success((JwtHeader(Some(HS256),Some(JWT),None),JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None),Some(VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ)))
```
