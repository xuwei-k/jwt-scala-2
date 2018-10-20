## JwtSprayJson Object

### Basic usage

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import pdi.jwt.{JwtSprayJson, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtSprayJson, JwtAlgorithm, JwtClaim}

scala> val claim = JwtClaim(
     |     expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond),
     |     issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1697820662),None,Some(1540035902),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtSprayJson.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2OTc4MjA2NjIsImlhdCI6MTU0MDAzNTkwMn0.8Dz0saewfSNHY2pRPHQjs5IQYCOunFIXeLA4Wiq9vVc

scala> JwtSprayJson.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res0: scala.util.Try[spray.json.JsObject] = Success({"exp":1697820662,"iat":1540035902})

scala> JwtSprayJson.decode(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1697820662),None,Some(1540035902),None))
```

### Encoding

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import spray.json._
import spray.json._

scala> import pdi.jwt.{JwtSprayJson, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtSprayJson, JwtAlgorithm, JwtClaim}

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val claimJson = s"""{"expires":${Instant.now.getEpochSecond}}""".parseJson.asJsObject
claimJson: spray.json.JsObject = {"expires":1540035902}

scala> val header = """{"typ":"JWT","alg":"HS256"}""".parseJson.asJsObject
header: spray.json.JsObject = {"typ":"JWT","alg":"HS256"}

scala> // From just the claim to all possible attributes
     | JwtSprayJson.encode(claimJson)
res3: String = eyJhbGciOiJub25lIn0.eyJleHBpcmVzIjoxNTQwMDM1OTAyfQ.

scala> JwtSprayJson.encode(claimJson, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNTQwMDM1OTAyfQ.St1mB7kgR6VycehHjFiG-CXB6QWRd0IyjFflu4-rOnI

scala> JwtSprayJson.encode(header, claimJson, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNTQwMDM1OTAyfQ.St1mB7kgR6VycehHjFiG-CXB6QWRd0IyjFflu4-rOnI
```

### Decoding

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import pdi.jwt.{JwtSprayJson, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtSprayJson, JwtAlgorithm, JwtClaim}

scala> val claim = JwtClaim(
     |     expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond),
     |     issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1697820663),None,Some(1540035903),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtSprayJson.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2OTc4MjA2NjMsImlhdCI6MTU0MDAzNTkwM30.XCADNqi8cjh_s6ogZnHbHOfQjq8gkF0omr59IF9aht8

scala> // You can decode to JsObject
     | JwtSprayJson.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[spray.json.JsObject] = Success({"exp":1697820663,"iat":1540035903})

scala> JwtSprayJson.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(spray.json.JsObject, spray.json.JsObject, String)] = Success(({"typ":"JWT","alg":"HS256"},{"exp":1697820663,"iat":1540035903},XCADNqi8cjh_s6ogZnHbHOfQjq8gkF0omr59IF9aht8))

scala> // Or to case classes
     | JwtSprayJson.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1697820663),None,Some(1540035903),None))

scala> JwtSprayJson.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None,None),JwtClaim({},None,None,None,Some(1697820663),None,Some(1540035903),None),XCADNqi8cjh_s6ogZnHbHOfQjq8gkF0omr59IF9aht8))
```
