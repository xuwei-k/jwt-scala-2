## Jwt with ECDSA algorithms

### With generated keys

#### Generation

```scala
scala> import java.security.spec.{ECPrivateKeySpec, ECPublicKeySpec, ECGenParameterSpec, ECParameterSpec, ECPoint}
import java.security.spec.{ECPrivateKeySpec, ECPublicKeySpec, ECGenParameterSpec, ECParameterSpec, ECPoint}

scala> import java.security.{SecureRandom, KeyFactory, KeyPairGenerator}
import java.security.{SecureRandom, KeyFactory, KeyPairGenerator}

scala> import pdi.jwt.{Jwt, JwtAlgorithm}
import pdi.jwt.{Jwt, JwtAlgorithm}

scala> // We specify the curve we want to use
     | val ecGenSpec = new ECGenParameterSpec("P-521")
ecGenSpec: java.security.spec.ECGenParameterSpec = java.security.spec.ECGenParameterSpec@f14ff64

scala> // We are going to use a ECDSA algorithm
     | // and the Bouncy Castle provider
     | val generatorEC = KeyPairGenerator.getInstance("ECDSA", "BC")
generatorEC: java.security.KeyPairGenerator = org.bouncycastle.jcajce.provider.asymmetric.ec.KeyPairGeneratorSpi$ECDSA@197eec84

scala> generatorEC.initialize(ecGenSpec, new SecureRandom())

scala> // Generate a pair of keys, one private for encoding
     | // and one public for decoding
     | val ecKey = generatorEC.generateKeyPair()
ecKey: java.security.KeyPair = java.security.KeyPair@69231ef0
```

#### Usage

```scala
scala> val token = Jwt.encode("""{"user":1}""", ecKey.getPrivate, JwtAlgorithm.ES512)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzUxMiJ9.eyJ1c2VyIjoxfQ.MIGIAkIA8P8Zj3fpi9Q5xT5kvAdXrn5NZCNOBg5kQtwmPp14uzgQ-vqG7vh-eoMfVg5TS51GCG8e2q2nIpUFoT18S7-_zhUCQgHQAVNQR-zriNaxtrsSBB6Cr78isgIz6s4jVxAzxB-XPDS6jzGJJ8okqKcpC0b2L8KgK5Ucs7x8U7L8xRmMUbQ_dg

scala> Jwt.decode(token, ecKey.getPublic)
res6: scala.util.Try[String] = Success({"user":1})
```

### With saved keys

Let's say you already have your keys, it means you know the **S** param for the private key and both **(X, Y)** for the public key. So we will first recreate the keys from those params and then use them just as we did for the previously generated keys.

#### Creation

```scala
scala> import org.bouncycastle.jce.ECNamedCurveTable
import org.bouncycastle.jce.ECNamedCurveTable

scala> import org.bouncycastle.jce.spec.ECNamedCurveSpec
import org.bouncycastle.jce.spec.ECNamedCurveSpec

scala> // Our saved params
     | val S = BigInt("1ed498eedf499e5dd12b1ab94ee03d1a722eaca3ed890630c8b25f1015dd4ec5630a02ddb603f3248a3b87c88637e147ecc7a6e2a1c2f9ff1103be74e5d42def37d", 16)
S: scala.math.BigInt = 6613901871199679514978992377442332791772883750139682649180026721181145754406288864809268672916336616771322098646358350740695574983337611844870539970967696253

scala> val X = BigInt("16528ac15dc4c8e0559fad628ac3ffbf5c7cfefe12d50a97c7d088cc10b408d4ab03ac0d543bde862699a74925c1f2fe7c247c00fddc1442099dfa0671fc032e10a", 16)
X: scala.math.BigInt = 4788717607397834000577334218768345962584270240715443811504328769211482568413869731756754251916562138931236920542879855527456625408658570895535273562246144266

scala> val Y = BigInt("b7f22b3c1322beef766cadd1a5f0363840195b7be10d9a518802d8d528e03bc164c9588c5e63f1473d05195510676008b6808508539367d2893e1aa4b7cb9f9dab", 16)
Y: scala.math.BigInt = 2466312264860322508999473447396730125124042289038893260685870541813191061595286807641865361617478701313850198481171224807088921078747769513243538251646475691

scala> // Here we are using the P-521 curve but you need to change it
     | // to your own curve
     | val curveParams = ECNamedCurveTable.getParameterSpec("P-521")
curveParams: org.bouncycastle.jce.spec.ECNamedCurveParameterSpec = org.bouncycastle.jce.spec.ECNamedCurveParameterSpec@1b739184

scala> val curveSpec: ECParameterSpec = new ECNamedCurveSpec( "P-521", curveParams.getCurve(), curveParams.getG(), curveParams.getN(), curveParams.getH());
curveSpec: java.security.spec.ECParameterSpec = org.bouncycastle.jce.spec.ECNamedCurveSpec@334540c1

scala> val privateSpec = new ECPrivateKeySpec(S.underlying(), curveSpec)
privateSpec: java.security.spec.ECPrivateKeySpec = java.security.spec.ECPrivateKeySpec@20bf810a

scala> val publicSpec = new ECPublicKeySpec(new ECPoint(X.underlying(), Y.underlying()), curveSpec)
publicSpec: java.security.spec.ECPublicKeySpec = java.security.spec.ECPublicKeySpec@747d6733

scala> val privateKeyEC = KeyFactory.getInstance("ECDSA", "BC").generatePrivate(privateSpec)
privateKeyEC: java.security.PrivateKey =
EC Private Key
             S: 1ed498eedf499e5dd12b1ab94ee03d1a722eaca3ed890630c8b25f1015dd4ec5630a02ddb603f3248a3b87c88637e147ecc7a6e2a1c2f9ff1103be74e5d42def37d

scala> val publicKeyEC = KeyFactory.getInstance("ECDSA", "BC").generatePublic(publicSpec)
publicKeyEC: java.security.PublicKey =
EC Public Key
            X: 16528ac15dc4c8e0559fad628ac3ffbf5c7cfefe12d50a97c7d088cc10b408d4ab03ac0d543bde862699a74925c1f2fe7c247c00fddc1442099dfa0671fc032e10a
            Y: b7f22b3c1322beef766cadd1a5f0363840195b7be10d9a518802d8d528e03bc164c9588c5e63f1473d05195510676008b6808508539367d2893e1aa4b7cb9f9dab
```

#### Usage

```scala
scala> val token = Jwt.encode("""{"user":1}""", privateKeyEC, JwtAlgorithm.ES512)
token: String = eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzUxMiJ9.eyJ1c2VyIjoxfQ.MIGIAkIBUAsuiZQs2bFK_O0Dw5-PBLD7uJdCYiTALP45xFhHfWjDqLmYcfkzFUQZJWxDiIvzLPrOademL6TYkOWX4YWUoT4CQgES-5qu9k_YBEZ8a0VHC2gCFC6hl82jaLn07Ci7GICPV785NYaFzt3dU7ZKixaTTUuGlrG4xh2RZ8vYxha9oYYwug

scala> Jwt.decode(token, publicKeyEC)
res10: scala.util.Try[String] = Success({"user":1})

scala> // Wrong key...
     | Jwt.decode(token, ecKey.getPublic)
res12: scala.util.Try[String] = Failure(pdi.jwt.exceptions.JwtValidationException: Invalid signature for this token.)
```
