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
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1645915449),None,Some(1488130689),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDU5MTU0NDksImlhdCI6MTQ4ODEzMDY4OX0.cBJXomimeL3TnB7npV-ixHlkHbiAzWPrM09tRiirA4A

scala> JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res0: scala.util.Try[upickle.Js.Value] =
Success({
    "exp": 1645915449,
    "iat": 1488130689
})

scala> JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1645915449),None,Some(1488130689),None))
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
    "expires": 1488130689
}

scala> val header = json.read( """{"typ":"JWT","alg":"HS256"}""")
header: upickle.Js.Value =
{
    "typ": "JWT",
    "alg": "HS256"
}

scala> // From just the claim to all possible attributes
     | JwtUpickle.encode(claimJson)
res3: String = eyJhbGciOiJub25lIn0.eyJleHBpcmVzIjoxNDg4MTMwNjg5fQ.

scala> JwtUpickle.encode(claimJson, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNDg4MTMwNjg5fQ.f-ievANSK20bH2tECrh-rOf55i2KRhwg33gY0YC19h8

scala> JwtUpickle.encode(header, claimJson, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNDg4MTMwNjg5fQ.f-ievANSK20bH2tECrh-rOf55i2KRhwg33gY0YC19h8
```

### Decoding

```scala
scala> val claim = JwtClaim(
     |   expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond),
     |   issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1645915450),None,Some(1488130690),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtUpickle.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDU5MTU0NTAsImlhdCI6MTQ4ODEzMDY5MH0.J0ehEPLiGzcSixvuhFchpl2uFXi0FTpRk_VMv-dkTQU

scala> // You can decode to JsObject
     | JwtUpickle.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[upickle.Js.Value] =
Success({
    "exp": 1645915450,
    "iat": 1488130690
})

scala> JwtUpickle.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(upickle.Js.Value, upickle.Js.Value, String)] =
Success(({
    "typ": "JWT",
    "alg": "HS256"
},{
    "exp": 1645915450,
    "iat": 1488130690
},J0ehEPLiGzcSixvuhFchpl2uFXi0FTpRk_VMv-dkTQU))

scala> // Or to case classes
     | JwtUpickle.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1645915450),None,Some(1488130690),None))

scala> JwtUpickle.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None),JwtClaim({},None,None,None,Some(1645915450),None,Some(1488130690),None),J0ehEPLiGzcSixvuhFchpl2uFXi0FTpRk_VMv-dkTQU))
```
