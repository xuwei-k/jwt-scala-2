## JwtHeader Case Class

```scala
scala> import pdi.jwt.{JwtHeader, JwtAlgorithm}
import pdi.jwt.{JwtHeader, JwtAlgorithm}

scala> JwtHeader()
res0: pdi.jwt.JwtHeader = JwtHeader(None,None,None)

scala> JwtHeader(JwtAlgorithm.HS256)
res1: pdi.jwt.JwtHeader = JwtHeader(Some(HS256),Some(JWT),None)

scala> JwtHeader(JwtAlgorithm.HS256, "JWT")
res2: pdi.jwt.JwtHeader = JwtHeader(Some(HS256),Some(JWT),None)

scala> // You can stringify it to JSON
     | JwtHeader(JwtAlgorithm.HS256, "JWT").toJson
res4: String = {"typ":"JWT","alg":"HS256"}

scala> // You can assign the default type (but it would have be done automatically anyway)
     | JwtHeader(JwtAlgorithm.HS1).withType
res6: pdi.jwt.JwtHeader = JwtHeader(Some(HS1),Some(JWT),None)
```
