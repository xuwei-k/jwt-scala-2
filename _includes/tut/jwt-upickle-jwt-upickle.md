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
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1657206818),None,Some(1499422058),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NTcyMDY4MTgsImlhdCI6MTQ5OTQyMjA1OH0.Whz2pw5LxnlexC5PV3D6CMj_LmGaJ-GRBBR5oP-W2Yg

scala> JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res0: scala.util.Try[upickle.Js.Value] =
Success({
    "exp": 1657206818,
    "iat": 1499422058
})

scala> JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1657206818),None,Some(1499422058),None))
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
    "expires": 1499422059
}

scala> val header = json.read( """{"typ":"JWT","alg":"HS256"}""")
header: upickle.Js.Value =
{
    "typ": "JWT",
    "alg": "HS256"
}

scala> // From just the claim to all possible attributes
     | JwtUpickle.encode(claimJson)
res3: String = eyJhbGciOiJub25lIn0.eyJleHBpcmVzIjoxNDk5NDIyMDU5fQ.

scala> JwtUpickle.encode(claimJson, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNDk5NDIyMDU5fQ.40OTjS4cycy-OmgESM9yxvSFI_dLniic8eMra4eCRO0

scala> JwtUpickle.encode(header, claimJson, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNDk5NDIyMDU5fQ.40OTjS4cycy-OmgESM9yxvSFI_dLniic8eMra4eCRO0
```

### Decoding

```scala
scala> val claim = JwtClaim(
     |   expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond),
     |   issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1657206819),None,Some(1499422059),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NTcyMDY4MTksImlhdCI6MTQ5OTQyMjA1OX0.IxkUugxrpj1zjXYEYgNdS1Kg1kVDJzrpMhVbnqDqubA

scala> // You can decode to JsObject
     | JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[upickle.Js.Value] =
Success({
    "exp": 1657206819,
    "iat": 1499422059
})

scala> JwtUpickle.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(upickle.Js.Value, upickle.Js.Value, String)] =
Success(({
    "typ": "JWT",
    "alg": "HS256"
},{
    "exp": 1657206819,
    "iat": 1499422059
},IxkUugxrpj1zjXYEYgNdS1Kg1kVDJzrpMhVbnqDqubA))

scala> // Or to case classes
     | JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1657206819),None,Some(1499422059),None))

scala> JwtUpickle.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None),JwtClaim({},None,None,None,Some(1657206819),None,Some(1499422059),None),IxkUugxrpj1zjXYEYgNdS1Kg1kVDJzrpMhVbnqDqubA))
```
