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
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1678071415),None,Some(1520286655),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NzgwNzE0MTUsImlhdCI6MTUyMDI4NjY1NX0.IYc1oz8acYvJBRouercvDSab4uNvy0CyhmLkoHeDjgI

scala> JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res0: scala.util.Try[upickle.Js.Value] =
Success({
    "exp": 1678071415,
    "iat": 1520286655
})

scala> JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1678071415),None,Some(1520286655),None))
```

### Encoding

```scala
scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val claimJson = json.read(s"""{"expires":${Instant.now.getEpochSecond}}""")
claimJson: upickle.Js.Value =
{
    "expires": 1520286655
}

scala> val header = json.read( """{"typ":"JWT","alg":"HS256"}""")
header: upickle.Js.Value =
{
    "typ": "JWT",
    "alg": "HS256"
}

scala> // From just the claim to all possible attributes
     | JwtUpickle.encode(claimJson)
res3: String = eyJhbGciOiJub25lIn0.eyJleHBpcmVzIjoxNTIwMjg2NjU1fQ.

scala> JwtUpickle.encode(claimJson, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNTIwMjg2NjU1fQ.MwVXiScawZlRZJuBkGRGMNYPuu8v5MsHh7wzc7cfG-8

scala> JwtUpickle.encode(header, claimJson, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNTIwMjg2NjU1fQ.MwVXiScawZlRZJuBkGRGMNYPuu8v5MsHh7wzc7cfG-8
```

### Decoding

```scala
scala> val claim = JwtClaim(
     |   expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond),
     |   issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1678071416),None,Some(1520286656),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NzgwNzE0MTYsImlhdCI6MTUyMDI4NjY1Nn0.St-Np9QO99jL2rT9Ikqsqf-ftjqGKMy8BxrncK0opj4

scala> // You can decode to JsObject
     | JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[upickle.Js.Value] =
Success({
    "exp": 1678071416,
    "iat": 1520286656
})

scala> JwtUpickle.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(upickle.Js.Value, upickle.Js.Value, String)] =
Success(({
    "typ": "JWT",
    "alg": "HS256"
},{
    "exp": 1678071416,
    "iat": 1520286656
},St-Np9QO99jL2rT9Ikqsqf-ftjqGKMy8BxrncK0opj4))

scala> // Or to case classes
     | JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1678071416),None,Some(1520286656),None))

scala> JwtUpickle.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None,None),JwtClaim({},None,None,None,Some(1678071416),None,Some(1520286656),None),St-Np9QO99jL2rT9Ikqsqf-ftjqGKMy8BxrncK0opj4))
```
