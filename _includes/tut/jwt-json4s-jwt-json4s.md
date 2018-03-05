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
res0: String = eyJhbGciOiJub25lIn0.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> val token = JwtJson4s.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> JwtJson4s.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[org.json4s.JObject] = Success(JObject(List((user,JInt(1)), (nbf,JInt(1431520421)))))

scala> JwtJson4s.decode(token, key, Seq(JwtAlgorithm.HS256))
res2: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None))
```

### Encoding

```scala
scala> val header = JObject(("typ", "JWT"), ("alg", "HS256"))
header: org.json4s.JsonAST.JObject = JObject(List((typ,JString(JWT)), (alg,JString(HS256))))

scala> JwtJson4s.encode(claim)
res3: String = eyJhbGciOiJub25lIn0.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.

scala> JwtJson4s.encode(claim, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ

scala> JwtJson4s.encode(header, claim, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJuYmYiOjE0MzE1MjA0MjF9.VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ
```

### Decoding

```scala
scala> // You can decode to JsObject
     | JwtJson4s.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[org.json4s.JObject] = Success(JObject(List((user,JInt(1)), (nbf,JInt(1431520421)))))

scala> JwtJson4s.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(org.json4s.JObject, org.json4s.JObject, String)] = Success((JObject(List((typ,JString(JWT)), (alg,JString(HS256)))),JObject(List((user,JInt(1)), (nbf,JInt(1431520421)))),VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ))

scala> // Or to case classes
     | JwtJson4s.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None))

scala> JwtJson4s.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None,None),JwtClaim({"user":1},None,None,None,None,Some(1431520421),None,None),VmfmoqRbRvna9lfpCx4lXf96eD_X_woBM0twLjBGLlQ))
```
