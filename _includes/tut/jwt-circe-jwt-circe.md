## JwtCirce Object

### Basic usage

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}

scala> val claim = JwtClaim(
     |     expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond)
     |   , issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1690676656),None,Some(1532891896),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtCirce.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2OTA2NzY2NTYsImlhdCI6MTUzMjg5MTg5Nn0.PMZUy93WXpIK93nixF0573ocq5TchCO7lhcLmv_Z5IA

scala> JwtCirce.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res0: scala.util.Try[io.circe.Json] =
Success({
  "exp" : 1690676656,
  "iat" : 1532891896
})

scala> JwtCirce.decode(token, key, Seq(JwtAlgorithm.HS256))
res1: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1690676656),None,Some(1532891896),None))
```

### Encoding

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import cats.syntax.either._
import cats.syntax.either._

scala> import io.circe._, syntax._, jawn.{parse => jawnParse}
import io.circe._
import syntax._
import jawn.{parse=>jawnParse}

scala> import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val Right(claimJson) = jawnParse(s"""{"expires":${Instant.now.getEpochSecond}}""")
claimJson: io.circe.Json =
{
  "expires" : 1532891897
}

scala> val Right(header) = jawnParse( """{"typ":"JWT","alg":"HS256"}""")
header: io.circe.Json =
{
  "typ" : "JWT",
  "alg" : "HS256"
}

scala> // From just the claim to all possible attributes
     | JwtCirce.encode(claimJson)
res3: String = eyJhbGciOiJub25lIn0.eyJleHBpcmVzIjoxNTMyODkxODk3fQ.

scala> JwtCirce.encode(claimJson, key, algo)
res4: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNTMyODkxODk3fQ.VKDeke1XTYGnqpv0RBQsXG0xq3vEkOgyNj4eapnYyR8

scala> JwtCirce.encode(header, claimJson, key)
res5: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHBpcmVzIjoxNTMyODkxODk3fQ.VKDeke1XTYGnqpv0RBQsXG0xq3vEkOgyNj4eapnYyR8
```

### Decoding

```scala
scala> import java.time.Instant
import java.time.Instant

scala> import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}
import pdi.jwt.{JwtCirce, JwtAlgorithm, JwtClaim}

scala> val claim = JwtClaim(
     |     expiration = Some(Instant.now.plusSeconds(157784760).getEpochSecond)
     |   , issuedAt = Some(Instant.now.getEpochSecond)
     | )
claim: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,Some(1690676658),None,Some(1532891898),None)

scala> val key = "secretKey"
key: String = secretKey

scala> val algo = JwtAlgorithm.HS256
algo: pdi.jwt.JwtAlgorithm.HS256.type = HS256

scala> val token = JwtCirce.encode(claim, key, algo)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2OTA2NzY2NTgsImlhdCI6MTUzMjg5MTg5OH0.n0_g3wICr-pb_iVA1FuilfidbWXzl-sgrSQVPzKGa4M

scala> // You can decode to JsObject
     | JwtCirce.decodeJson(token, key, Seq(JwtAlgorithm.HS256))
res7: scala.util.Try[io.circe.Json] =
Success({
  "exp" : 1690676658,
  "iat" : 1532891898
})

scala> JwtCirce.decodeJsonAll(token, key, Seq(JwtAlgorithm.HS256))
res8: scala.util.Try[(io.circe.Json, io.circe.Json, String)] =
Success(({
  "typ" : "JWT",
  "alg" : "HS256"
},{
  "exp" : 1690676658,
  "iat" : 1532891898
},n0_g3wICr-pb_iVA1FuilfidbWXzl-sgrSQVPzKGa4M))

scala> // Or to case classes
     | JwtCirce.decode(token, key, Seq(JwtAlgorithm.HS256))
res10: scala.util.Try[pdi.jwt.JwtClaim] = Success(JwtClaim({},None,None,None,Some(1690676658),None,Some(1532891898),None))

scala> JwtCirce.decodeAll(token, key, Seq(JwtAlgorithm.HS256))
res11: scala.util.Try[(pdi.jwt.JwtHeader, pdi.jwt.JwtClaim, String)] = Success((JwtHeader(Some(HS256),Some(JWT),None,None),JwtClaim({},None,None,None,Some(1690676658),None,Some(1532891898),None),n0_g3wICr-pb_iVA1FuilfidbWXzl-sgrSQVPzKGa4M))
```
