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
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1654738282),None,Some(1496953522),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NTQ3MzgyODIsImlhdCI6MTQ5Njk1MzUyMn0.Bv2Jf02uWPhhgdPng9hCR2_t0Il_h8NVr-KroJeV1MA

scala> JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res0: scala.util.Try[upickle.Js.Value] =
Success({
    "exp": 1654738282,
    "iat": 1496953522
})

scala> JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1654738282),None,Some(1496953522),None))
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
    "expires": 1496953522
}

scala> val header = json.read( """{"typ":"JWT","alg":"HS256"}""")
header: upickle.Js.Value =
{
    "typ": "JWT",
    "alg": "HS256"
}

scala> // From just the claim to all possible attributes
     | JwtUpickle.encode(claimJson)
res3: String = eyJhbGciOiJub25lIn0.eyJleHBpcmVzIjoxNDk2OTUzNTIyfQ.

scala> JwtUpickle.encode(claimJson, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNDk2OTUzNTIyfQ.mxq7iotJHXpo4n6frv9TX6e4Ua9brMN0-TYzByBx-yI

scala> JwtUpickle.encode(header, claimJson, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNDk2OTUzNTIyfQ.mxq7iotJHXpo4n6frv9TX6e4Ua9brMN0-TYzByBx-yI
```

### Decoding

```scala
scala> val claim = JwtClaim(
     |   expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond),
     |   issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1654738283),None,Some(1496953523),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NTQ3MzgyODMsImlhdCI6MTQ5Njk1MzUyM30.Xjttwi-8JtBUTCx1JFAFHlGISCdDE9fKaaQHRQmT8Dg

scala> // You can decode to JsObject
     | JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[upickle.Js.Value] =
Success({
    "exp": 1654738283,
    "iat": 1496953523
})

scala> JwtUpickle.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(upickle.Js.Value, upickle.Js.Value, String)] =
Success(({
    "typ": "JWT",
    "alg": "HS256"
},{
    "exp": 1654738283,
    "iat": 1496953523
},Xjttwi-8JtBUTCx1JFAFHlGISCdDE9fKaaQHRQmT8Dg))

scala> // Or to case classes
     | JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1654738283),None,Some(1496953523),None))

scala> JwtUpickle.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None),JwtClaim({},None,None,None,Some(1654738283),None,Some(1496953523),None),Xjttwi-8JtBUTCx1JFAFHlGISCdDE9fKaaQHRQmT8Dg))
```
