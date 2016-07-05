## JwtClaim Case Class

```scala
scala> import pdi.jwt.JwtClaim
import pdi.jwt.JwtClaim

scala> JwtClaim()
res0: pdi.jwt.JwtClaim = JwtClaim({},None,None,None,None,None,None,None)

scala> // Specify the content as JSON string
     | // (don't use var in your code if possible, this is just to ease the sample)
     | var claim = JwtClaim("""{"user":1}""")
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1},None,None,None,None,None,None,None)

scala> // Append new content
     | claim = claim + """{"key1":"value1"}"""
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1"},None,None,None,None,None,None,None)

scala> claim = claim + ("key2", true)
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true},None,None,None,None,None,None,None)

scala> claim = claim ++ (("key3", 3), ("key4", Seq(1, 2)), ("key5", ("key5.1", "Subkey")))
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},None,None,None,None,None,None,None)

scala> // Stringify as JSON
     | claim.toJson
res5: String = {"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}}

scala> // Manipulate basic attributes
     | // Set the issuer
     | claim = claim.by("Me")
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),None,None,None,None,None,None)

scala> // Set the audience
     | claim = claim.to("You")
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),None,Some(Set(You)),None,None,None,None)

scala> // Set the subject
     | claim = claim.about("Something")
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),None,None,None,None)

scala> // Set the id
     | claim = claim.withId("42")
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),None,None,None,Some(42))

scala> // Set the expiration
     | // In 10 seconds from now
     | claim = claim.expiresIn(5)
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1467738375),None,None,Some(42))

scala> // At a specific timestamp (in seconds)
     | claim.expiresAt(1431520421)
res14: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1431520421),None,None,Some(42))

scala> // Right now! (the token is directly invalid...)
     | claim.expiresNow
res16: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1467738370),None,None,Some(42))

scala> // Set the beginning of the token (aka the "not before" attribute)
     | // 5 seconds ago
     | claim.startsIn(-5)
res19: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1467738375),Some(1467738365),None,Some(42))

scala> // At a specific timestamp (in seconds)
     | claim.startsAt(1431520421)
res21: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1467738375),Some(1431520421),None,Some(42))

scala> // Right now!
     | claim = claim.startsNow
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1467738375),Some(1467738371),None,Some(42))

scala> // Set the date when the token was created
     | // (you should always use claim.issuedNow, but I let you do otherwise if needed)
     | // 5 seconds ago
     | claim.issuedIn(-5)
res26: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1467738375),Some(1467738371),Some(1467738366),Some(42))

scala> // At a specific timestamp (in seconds)
     | claim.issuedAt(1431520421)
res28: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1467738375),Some(1467738371),Some(1431520421),Some(42))

scala> // Right now!
     | claim = claim.issuedNow
claim: pdi.jwt.JwtClaim = JwtClaim({"user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}},Some(Me),Some(Something),Some(Set(You)),Some(1467738375),Some(1467738371),Some(1467738371),Some(42))

scala> // We can test if the claim is valid => testing if the current time is between "not before" and "expiration"
     | claim.isValid
res31: Boolean = true

scala> // Also test the issuer and audience
     | claim.isValid("Me", "You")
res33: Boolean = true

scala> // Let's stringify the final version
     | claim.toJson
res35: String = {"iss":"Me","sub":"Something","aud":["You"],"exp":1467738375,"nbf":1467738371,"iat":1467738371,"jti":"42","user":1,"key1":"value1","key2":true,"key3":3,"key4":[1,2],"key5":{"key5.1":"Subkey"}}
```
