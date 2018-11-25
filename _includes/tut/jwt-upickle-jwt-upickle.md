## JwtUpickle Object

### Basic usage

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import upickle.json
import upickle.json

scala> import upickle.default._
import upickle.default._

scala> import pdi.jwt.{JwtUpickle, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtUpickle, JwtAlgorithm, JwtClaim}

scala> val claim = JwtClaim(
     |   expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond),
     |   issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1700948949),None,Some(1543164189),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3MDA5NDg5NDksImlhdCI6MTU0MzE2NDE4OX0.eS1UX0_M_FunShj1ohq7BK17aXdoHs0a7zJkq2bOgnA

scala> JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res0: scala.util.Try[upickle.Js.Value] = Success({"exp":1700948949,"iat":1543164189})

scala> JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1700948949),None,Some(1543164189),None))
```

### Encoding

```scala
scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val claimJson = json.read(s"""{"expires":${Instant.now.getEpochSecond}}""")
claimJson: upickle.Js.Value = {"expires":1543164189}

scala> val header = json.read( """{"typ":"JWT","alg":"HS256"}""")
header: upickle.Js.Value = {"typ":"JWT","alg":"HS256"}

scala> // From just the claim to all possible attributes
     | JwtUpickle.encode(claimJson)
res3: String = eyJhbGciOiJub25lIn0.eyJleHBpcmVzIjoxNTQzMTY0MTg5fQ.

scala> JwtUpickle.encode(claimJson, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNTQzMTY0MTg5fQ.-xlp_vd1_KdoiLTxxx1PFvHCuqQ_DUfzcPn9QNXVaYI

scala> JwtUpickle.encode(header, claimJson, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNTQzMTY0MTg5fQ.-xlp_vd1_KdoiLTxxx1PFvHCuqQ_DUfzcPn9QNXVaYI
```

### Decoding

```scala
scala> val claim = JwtClaim(
     |   expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond),
     |   issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1700948950),None,Some(1543164190),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3MDA5NDg5NTAsImlhdCI6MTU0MzE2NDE5MH0.Hq1leLAI3toGl9R43q0-IxjMkCWdvgTr0bIAHiMQcrU

scala> // You can decode to JsObject
     | JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[upickle.Js.Value] = Success({"exp":1700948950,"iat":1543164190})

scala> JwtUpickle.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(upickle.Js.Value, upickle.Js.Value, String)] = Success(({"typ":"JWT","alg":"HS256"},{"exp":1700948950,"iat":1543164190},Hq1leLAI3toGl9R43q0-IxjMkCWdvgTr0bIAHiMQcrU))

scala> // Or to case classes
     | JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1700948950),None,Some(1543164190),None))

scala> JwtUpickle.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None,None),JwtClaim({},None,None,None,Some(1700948950),None,Some(1543164190),None),Hq1leLAI3toGl9R43q0-IxjMkCWdvgTr0bIAHiMQcrU))
```
