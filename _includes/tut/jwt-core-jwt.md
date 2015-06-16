## Jwt object

### Basic usage

```scala
scala> import pdi.jwt.{Jwt, JwtAlgorithm, JwtHeader, JwtClaim}
import pdi.jwt.{Jwt, JwtAlgorithm, JwtHeader, JwtClaim}

scala> val token = Jwt.encode("""{"user":1}""", "secretKey", JwtAlgorithm.HS256)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxfQ.oG3iKnAvj_OKCv0tchT90sv2IFVeaREgvJmwgRcXfkI

scala> Jwt.decodeRawAll(token, "secretKey")
res0: scala.util.Try[(String, String, String)] = Success(({"typ":"JWT","alg":"HS256"},{"user":1},oG3iKnAvj_OKCv0tchT90sv2IFVeaREgvJmwgRcXfkI))

scala> Jwt.decodeRawAll(token, "wrongKey")
res1: scala.util.Try[(String, String, String)] = Failure(pdi.jwt.exceptions.JwtValidationException: Invalid signature for this token.)
```

### Encoding

```scala
scala> // Encode from string, header automatically generated
     | Jwt.encode("""{"user":1}""", "secretKey", JwtAlgorithm.HS384)
res3: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzM4NCJ9.eyJ1c2VyIjoxfQ.Do0PQWccbp1J7ZWcFL-_IY9OFaI-7t75k7-NxZ52jk2kAb0sFopJEeZapkiXthEp

scala> // Encode from case class, header automatically generated
     | // Set that the token has been issued now and expires in 10 seconds
     | Jwt.encode(JwtClaim({"""{"user":1}"""}).issuedNow.expiresIn(10), "secretKey", JwtAlgorithm.HS512)
res6: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJleHAiOjE0MzQ0NzA1MDksImlhdCI6MTQzNDQ3MDQ5OSwidXNlciI6MX0.ipmNq7b1l-O-Kcr4__Iv3kDrKm5NZBEzho0-gGypNhYoEuJ8wUVdDTGZ9dO0JuN_el8BjuTXZBzI5m7xmUILDg

scala> // You can encode without signing it
     | Jwt.encode("""{"user":1}""")
res8: String = eyJhbGciOiJub25lIn0.eyJ1c2VyIjoxfQ.

scala> // You can specify a string header but also need to specify the algorithm just to be sure
     | // This is not really typesafe, so please use it with care
     | Jwt.encode("""{"typ":"JWT","alg":"HS1"}""", """{"user":1}""", "key", JwtAlgorithm.HS1)
res11: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzEifQ.eyJ1c2VyIjoxfQ.jcR5T4yT0aqoR_7T87D8V4P8VFc

scala> // If using a case class header, no need to repeat the algorithm
     | // This is way better than the previous one
     | Jwt.encode(JwtHeader(JwtAlgorithm.HS1), JwtClaim("""{"user":1}"""), "key")
res14: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzEifQ.eyJ1c2VyIjoxfQ.jcR5T4yT0aqoR_7T87D8V4P8VFc
```

### Decoding

In JWT Scala, espcially when using raw strings which are not typesafe at all, there are a lot of possible errors. This is why nearly all `decode` functions will return a `Try` rather than directly the expected result. In case of failure, the wrapped exception should tell you what went wront.

```scala
scala> // Decode all parts of the token as string
     | Jwt.decodeRawAll(token, "secretKey")
res16: scala.util.Try[(String, String, String)] = Success(({"typ":"JWT","alg":"HS256"},{"user":1},oG3iKnAvj_OKCv0tchT90sv2IFVeaREgvJmwgRcXfkI))

scala> // Decode only the claim as a string
     | Jwt.decodeRaw(token, "secretKey")
res18: scala.util.Try[String] = Success({"user":1})

scala> // Decode all parts and cast them as a better type if possible.
     | // Since the implementation in JWT Core only use string, it is the same as decodeRawAll
     | // But check the result in JWT Play JSON to see the difference
     | Jwt.decodeAll(token, "secretKey")
res22: scala.util.Try[(String, String, String)] = Success(({"typ":"JWT","alg":"HS256"},{"user":1},oG3iKnAvj_OKCv0tchT90sv2IFVeaREgvJmwgRcXfkI))

scala> // Same as before, but only the claim
     | // (you should start to see a pattern in the naming convention of the functions)
     | Jwt.decode(token, "secretKey")
res25: scala.util.Try[String] = Success({"user":1})

scala> // Failure because the token is not a token at all
     | Jwt.decode("Hey there!")
res27: scala.util.Try[String] = Failure(pdi.jwt.exceptions.JwtLengthException: Expected token [Hey there!] to be composed of 2 or 3 parts separated by dots.)

scala> // Failure if not Base64 encoded
     | Jwt.decode("a.b.c")
res29: scala.util.Try[String] = Failure(java.lang.IllegalArgumentException: Input byte[] should at least have 2 bytes for base64 bytes)

scala> // Failure in case we use the wrong key
     | Jwt.decode(token, "wrongKey")
res31: scala.util.Try[String] = Failure(pdi.jwt.exceptions.JwtValidationException: Invalid signature for this token.)

scala> // Failure if the token only starts in 5 seconds
     | Jwt.decode(Jwt.encode(JwtClaim().startsIn(5)))
res33: scala.util.Try[String] = Failure(pdi.jwt.exceptions.JwtNotBeforeException: The token will only be valid after 2015-06-16T16:01:45Z)
```

### Validating

If you only want to check if a token is valid without decoding it. You have two options: `validate` functions that will throw the exceptions we saw in the decoding section, so you know what went wrong, or `isValid` functions that will return a boolean in case you don't care about the actual error and don't want to bother with catching exception.

```scala
scala> // All good
     | Jwt.validate(token, "secretKey")

scala> Jwt.isValid(token, "secretKey")
res36: Boolean = true

scala> // Wrong key here
     | Jwt.validate(token, "wrongKey")
pdi.jwt.exceptions.JwtValidationException: Invalid signature for this token.
  at pdi.jwt.JwtCore$class.validate(Jwt.scala:414)
  at pdi.jwt.Jwt$.validate(Jwt.scala:23)
  at pdi.jwt.JwtCore$class.validate(Jwt.scala:422)
  at pdi.jwt.Jwt$.validate(Jwt.scala:23)
  at pdi.jwt.JwtCore$class.validate(Jwt.scala:469)
  at pdi.jwt.Jwt$.validate(Jwt.scala:23)
  ... 428 elided

scala> Jwt.isValid(token, "wrongKey")
res39: Boolean = false

scala> // No key for unsigned token => ok
     | Jwt.validate(Jwt.encode("{}"))

scala> Jwt.isValid(Jwt.encode("{}"))
res42: Boolean = true

scala> // No key while the token is actually signed => wrong
     | Jwt.validate(token)
pdi.jwt.exceptions.JwtNonEmptySignatureException: Non-empty signature found inside the token while trying to verify without a key.
  at pdi.jwt.JwtCore$class.validate(Jwt.scala:390)
  at pdi.jwt.Jwt$.validate(Jwt.scala:23)
  at pdi.jwt.JwtCore$class.validate(Jwt.scala:454)
  at pdi.jwt.Jwt$.validate(Jwt.scala:23)
  ... 460 elided

scala> Jwt.isValid(token)
res45: Boolean = false

scala> // The token hasn't started yet!
     | Jwt.validate(Jwt.encode(JwtClaim().startsIn(5)))
pdi.jwt.exceptions.JwtNotBeforeException: The token will only be valid after 2015-06-16T16:01:46Z
  at pdi.jwt.JwtTime$.validateNowIsBetween(JwtTime.scala:48)
  at pdi.jwt.JwtTime$.validateNowIsBetweenSeconds(JwtTime.scala:64)
  at pdi.jwt.JwtCore$class.validateTiming(Jwt.scala:384)
  at pdi.jwt.Jwt$.validateTiming(Jwt.scala:23)
  at pdi.jwt.JwtCore$class.validate(Jwt.scala:395)
  at pdi.jwt.Jwt$.validate(Jwt.scala:23)
  at pdi.jwt.JwtCore$class.validate(Jwt.scala:454)
  at pdi.jwt.Jwt$.validate(Jwt.scala:23)
  ... 476 elided

scala> Jwt.isValid(Jwt.encode(JwtClaim().startsIn(5)))
res48: Boolean = false

scala> // This is no token
     | Jwt.validate("a.b.c")
java.lang.IllegalArgumentException: Input byte[] should at least have 2 bytes for base64 bytes
  at java.util.Base64$Decoder.outLength(Base64.java:659)
  at java.util.Base64$Decoder.decode(Base64.java:525)
  at pdi.jwt.JwtBase64Impl$class.decode(JwtBase64Impl.scala:9)
  at pdi.jwt.JwtBase64$.decode(JwtBase64.scala:3)
  at pdi.jwt.JwtBase64$.decodeString(JwtBase64.scala:11)
  at pdi.jwt.JwtBase64$.decodeString(JwtBase64.scala:14)
  at pdi.jwt.JwtCore$class.pdi$jwt$JwtCore$$splitToken(Jwt.scala:219)
  at pdi.jwt.JwtCore$class.validate(Jwt.scala:453)
  at pdi.jwt.Jwt$.validate(Jwt.scala:23)
  ... 492 elided

scala> Jwt.isValid("a.b.c")
res51: Boolean = false
```
