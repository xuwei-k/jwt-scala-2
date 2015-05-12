## JWT object

```scala
scala> import pdi.jwt._
import pdi.jwt._

scala> Jwt.encode("""{"alg":"HS256","typ":"JWT"}""","""{user:1}""","secretKey", JwtAlgorithm.HS256)
res0: String = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.e3VzZXI6MX0.qzBa2uZTOUDeQtxLqhU2exQTztm7NByLZexYHI_98no
```
