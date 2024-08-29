### What is a JSON Web Token?

---

基于文本的消息格式，用于安全地传递信息。不仅可以用在令牌传输中，还可以用于发送任何数据格式。

最简单的 JWT 包含两个部分：

1. 主要数据 `payload` ，负载
2. JSON 格式的 `header`，包含关于负载和消息本身的元数据的名称/值对。

JWT 负载可以是任何东西 —— 只要能用 byte 数组表示，例如字符串、图片、文档等。

因为 JWT 的 `header` 是一个 JSON 对象，所以 `payload` 使用 JSON 对象也很合理。当使用 JSON 对象时，`payload` 也被叫做 `Claims`，并且每对 name/value 对就叫 `claim` 。

用 claim 传输认证信息很方便，而且任何人都可以这么做。正因如此，必须保证 claims 的来源是信得过的。

JWTs 的一个很棒的特性就是可以通过多种途径保护起来。JWT 可以被签名（即 JWS）或者加密（即 JWE）。通过验证签名或者解密 JWT，JWT的来源就变得更可靠。

最后，带空格的 JSON 可读性非常强，但作为信息格式来说效率不是很高。因此 JWT 可以被压缩为最小的表示 —— 一般是 Base64URL 编码的字符串 —— 以便在比如 HTTP header 或者 URL 中高效传输。

#### JWT 示例

`payload` 和 `header` 是如何组合起来的？

1. 假设有一个 JWT，`header` 是 JSON 对象，负载是一段文本：

   **header**

   ```java
   {
       "alg": "none"
   }
   ```

   **payload**

   ```text
   The true sign of intelligence is not knowledge but imagination.
   ```

2. 移除 JSON 中所有非必要的空格：

   ```java
   String header = "{\"alg\":\"none\"}";
   String payload = "The true sign of intelligence is not knowledge but imagination.";
   ```

3. 获取每个部分的 UTF-8 字节数组并用 Base64URL-encode 编码：

   ```java
   String encodedHeader = Base64.getUrlEncoder().encodeToString(header.getBytes(StandardCharsets.UTF_8));
   String encodedPayload = Base64.getUrlEncoder().encodeToString(payload.getBytes(StandardCharsets.UTF_8));
   ```

4. 拼凑编码后的各部分，并且加上句号  "."

   ```java
   String compact = encodedHeader + "." + encodedPayload + ".";
   ```

最终的 JWT 字符串看起来就像这样：

```text
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJKb2UifQ.BFAa5DaW1EPwi1Les_GxxwiqvRSpjFzEeadnSdLBvWoeyJhbGciOiJub25lIn0=.eyJhbGciOiJub25lIn0=.
```

这就是未受保护的 JWT，没有签名没有加密，无法确定是否被第三方篡改过。

如果需要对 JWT 签名，还需要多几个步骤。

#### JWS 示例

这次不用文本做负载，而是使用 JSON，携带认证相关的信息。并且对 JWT 签名，确保如果被第三方篡改内容，可以被我们看出来。

1. 假设有这样一个 `header` 和 `payload` ：

   **header**

   ```java
   {
       "alg": "HS256"
   }
   ```

   **payload**

   ```java
   {
       "sub": "Joe"
   }
   ```

   `header` 中说明了此次签名所用的算法为 "HS256" 算法（HMAC using SHA-256）。另外，负载中的 JSON 对象拥有一个 key 为 "sub"，值为 “Joe" 的 claim。

2. 移除所有不重要的空格：

   ```java
   String header = "{\"alg\":\"HS256\"}";
   String claims = "{\"sub\":\"Joe\"}";
   ```

3. 获取各个部分的 UTF-8 字节数组，用 Base64URL 编码：

   ```java
   String encodedHeader = Base64.getUrlEncoder().encodeToString(header.getBytes(StandardCharsets.UTF_8));
   String encodedClaims = Base64.getUrlEncoder().encodeToString(claims.getBytes(StandardCharsets.UTF_8));
   ```

4. 用句号 "." 将编码后的 header 和 claims 拼凑起来：

   ```java
   String concatenated = encodedHeader + "." + encodedClaims;
   ```

5. 使用密钥 secret ，选用一种签名算法，对拼凑后的字符串签名：

   ```java
   String secret = "my_secret";
   Mac hmac = Mac.getInstance("HmacSHA256");
   SecretKeySpec secretKey = new SecretKeySpec(secret.getBytes(StandardCharacter.UTF-8), "HmacSHA256");
   hmac.init(secretKey);
   byte[] bytes = hmax.doFinal(concatenated.getBytes());
   ```

6. 签名都是 byte 数组，所以先用 Base64 编码，然后用 "." 符号跟  `concatenated` 字符串拼接起来：

   ```java
   String encodedSignature = Base64.getUrlEncoder().encodeToString(bytes);
   String compact = concatenated + "." + encodedSignature;
   ```

最终的 `compact` 字符串：

```text
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJKb2Ui.wtl52E258XyiMcPe4Rl7ya9pPpLg4f0w3-9SW1w2l5g=
```

这些步骤全部由自己手动操作，实在是太麻烦了。稍有差错，就有可能导致严重的安全问题。JWT 就是用来这些操作的：自动化创建、解析、验证所有的 JWS。



#### JWE 示例

JWT 不受保护，JWS 被签名，但是两者其中的信息都是可见的，JWS 只是防止信息被人篡改，很多时候对于非敏感信息来说这是可以的。

但是对于敏感信息，比如邮件地址，银行账户，这类敏感的信息呢？

我们可以用加密的 JWT —— JWE 来确保 payload 被加密，未授权的人无法看到其中的内容，也无法篡改其中的内容。JWE 需要一个 AEAD(Authenticated Encryption with Associated Data) 算法来加密和保护数据。

这是利用  AEAD 算法生成的最终的 JWE：

```text
eyJhbGciOiJBMTI4S1ciLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0.
6KB707dM9YTIgHtLvtgWQ8mKwboJW3of9locizkDTHzBC2IlrT1oOQ.
AxY8DCtDaGlsbGljb3RoZQ.
KDlTtXchhZTGufMYmOYGS4HffxPSUrfmqCHXaI9wOGY.
U0m_YmjN04DJvceFICbCVQ
```

#### JJWT 的使用

##### 依赖

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.6</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if Gson is preferred -->
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
```

### 上手

---

JWT 已经处理好了细节，只需要简单的使用：

```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import java.security.Key;

// 一般来说签名的 key 是从配置中读取的
SecretKey key = Jwts.SIG.HS256.key().build();

String jws = Jwts.builder().subject("Joe").SighWith(key).compact();
```

上述代码完成了：

1. 创建一个带有 sub键，值为 ”Joe“ 的claim 的 JWT
2. 使用一个适用于 HS256 算法的 key，对 JWT 签名
3. 拼凑成字符串形式，形成 JWS

最终的结果：

```text
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJKb2UifQ.Alo8lLyLk589dBUyrr5Y4ae9GLxRYrDpLcVJOE6KwXw
```

验证：

```java
assert Jwts.parser().verifyWith(key).build().parseSignedClaims(jws).getPayload().getSubject().equals("Joe");
```

如果 JWT 验证失败，可以使用：

```java
try {
    Jwts.parser().verifyWith(key).build().parseSignedClaims(jws);
} catch (JwtException e) {
    // 不信任该 JWT
}
```

#### 创建 JWT 

步骤：

1. 使用 `Jwts.builder()` 方法，创建一个 `JwtBuilder` 实例
2. 选择性设置 `header` 参数
3. 调用 builder 的方法，设置 `payload` 的内容或者 claims
4. 选择性调用 `signWith` 或者 `encryptWith` 方法来对 JWT 签名或者加密
5. 调用 `compact()` 方法，生成最终的 JWT 字符串

举例：

```java
String jwt = Jwts.builder()						// (1)
    .header()									// (2) optional
    	.keyId("aKeyId")
    	.and()
    .subject("Bob")								// (3) JSON Claims, or
    //.content(aByteArray, "text/plain") 		// 	any byte[] content, with media type
    
    .signWith(signingKey)						// (4) if signing, or
    //.encryptWith(key, keyAlg, encryptionAlg)	// 	   if encrypting
    
    .compact();									// (5)
```

##### JWT Header

JWT header 是一个 JSON 对象，提供了各种关于内容、格式、payload 相关的加密操作的元信息。

###### JwtBuilder Header

设置一个或多个 JWT header 参数（键值对），最简单最推荐的方法就是使用 `JwtBuilder` 的 `header` ，然后再调用 `and()` 方法，回到 `JwtBuilder` 对象，进行后续操作。例如：

```java
String jwt = Jwts.builder()
    	.header()					// <--- 使用 header
    		.keyId("aKeyId")
    		.x509Url(aUri)
    		.add("someName", anyValue)
    		.add(mapValues)
    		// ... etc ...
    		.and()					// 回到 JwtBuilder
    
    	.subject("Joe")
    	// ... etc ...
    	.compact();
```

NOTE：不需要设置 `alg` , `enc` 或者 `zip` headers，JJWT 会自动设置，如果有需要的话。

可以自定义 header 参数，用 `add` 方法添加可选的 **键值对** 参数

```java
Jwts.builder()
    	.header()
    		.add("aHeaderName", aValue)
    		// ... etc ...
    		.and() // 回到 JwtBuilder
    // ... etc ...
```

也可以用 Map 作为 header 参数

```java
Jwts.builder()
    	.header()
    		.add(multipleHeaderParamsMap)
    		// ... etc ...
    		.and() // return to the JwtBuilder
   // ... etc ...
```

###### Jwts HeaderBuilder

如果要在 `JwtBuilder` 外创建一个 `Header` ，可以使用 `Jwts.header()` ，返回一个独立的 `Header` builder：

```java
Header header = Jwts.header()
    	.keyId("aKeyId")
    	.x509Url(aUri)
    	.add("someName", anyValue)
    	.add(mapValues)
    	// ... etc ...
    	
    	.build() // <---- not 'and()'
```

`Jwts.header()` 和 `Jwts.builder().header()` 只有两个区别：

1. `Jwts.header()` 创建一个跟任何特定 JWT  无关的，独立的 `Header` ，而 `Jwts.builder().header()` 总是在 `JwtBuilder` 构造的 JWT 的 header 上做改动。
2. `Jwts.header()` 使用 `build()` 方法创建一个显式的 `Header` 实例，而 `Jwts.builder().header()` 不会这么做（使用 `and()` 方法），而是由它的 `JwtBuilder` 隐式创建 header。

独立的 header 在你想要合并相同参数的时候很有用，只要将其添加到 `JwtBuilder` header 即可：

```java
Header commonHeaders = Jwts.header()
    .issuer("My Company")
    // ... etc ...
    .build();
Strign jwt = Jwts.builder()
    
    .header()
    	.add(commonHeaders)						// <--- 在这里添加 header
    	.add("specificHeader", specificValue) 	// 增加别的 header 配置
    	.and()
    .subject("whatever")
    // ... etc ...
    .compact();
```

##### JWT Payload

JWT 的负载可以是任何东西  - 任何可以用字节数组表示的东西，比如文本，图片，文档灯。既然 `header` 可以是一个 JSON，那负载自然也可以是 JSON，尤其是表示认证的 claims。

所以，`JwtBuilder` 支持两种不同的负载选择：

- `content` 如果想用字节数组作为负载
- `claims` 如果想要用 JSON Claims 作为负载

两种方式都可以，但是只能选择一种，不能同时使用两种，否则 `compact()` 的时候就会报错。

###### 任意内容

用 `JwtBuilder` 的 `content` 方法，可以使用任意的字节数组内容作为负载：

```java
byte[] content = "Hello World".getBytes(StandardCharsets.UTF_8);
String jwt = Jwts.builder()
    .content(content, "text/plain") // <---
    // .. etc ...
    .build();
```

该方法使用了两个参数：

1. 第一个参数是实际的负载的字节内容
2. 第二个参数是媒体类型的字符串标识

第二个参数将会导致 `JwtBuilder` 自动设置 `cty (Content Type)` header，根据 JWT 规范的推荐格式。

推荐使用 双参数 的方法，它能保证 JWT 接收者从 header 中的 `cty` 得知如何正确转化负载中的 字节数组 为程序用到的格式。

###### JWT Claims

JJWT 支持类型安全的 claims 创建方式。

**标准 claims：**

提供了各种标准的方法：

- `issuer`
- `subject`
- `audience` 
- `expiration` 
- `notBefore`
- `issuedAt`
- `id`

```java
String jws = Jwts.builder()
    .issuer("me")
    .subject("Bob")
    .audience().add("you").and()
    .expiration(expiration) // a java.util.Date
    .notBefore(notBefore)	// a java.util.Date
    .issuedAt(new Date())	// for example, now
    .id(UUID.randomUUID().toString())	// just an example id
    
    /// ... etc ...
```

**自定义 Claims：**

想要自定义 claims 的内容，可以使用 `JwtBuilder` 的 `claim` 方法：

```java
String jws = Jwts.builder()
    .claim("hello", "world")
    .claim("This", "is good")
    
    // ... etc ...
```

每次 `claim` 方法被调用，Jwt 只是简单地把新的 键值对 添加到内部的 `Claims` builder 中，可能会导致同名的键值对被覆盖。

**Claims Map**

如果想一次性添加多个 claims，可以使用 `JwtBuilder` 的 `claims(Map)` 方法：

```java
Map<String,?> claims = getMyClaimsMap();  // 返回一个 Map 的接口
String jws = Jwts.builder()
    .claims(claims)

    // ... etc ...
```

### 读取 JWT

---

流程：

1. 使用 `Jwts.parse()` 方法，创建一个 `JwtParserBuilder` 实例
2. 选择性调用 `keyLocator`, `verifyWith` 或者 `decryptWith` 方法（解析签名或加密过的 JWT）
3. 调用 `build()` 方法，创建一个线程安全的 `JwtParse()` 
4. 调用其中一个 `parse*` 方法，解析 JWT 字符串，取决于 JWT 的类型
5. 用 try/catch 块包裹解析过程的调用

例如：

```java
Jwt<?,?> jwt;
try {
    jwt = Jwts.parser()
        .keyLocator(keyLocator)		// 动态定位签名或者加密的 key
        //.verifyWith(key)			// 或者验证签名的常量 key
        //.decryptWith(key)			// 或者加密的 key
        
        .build()
        
        .parse(compact);			// 或者 parseSignedClaims, parseEncryptedClaims, parseSignedContent 等等
} catch (JwtException ex) {
    
}
```

#### 常量 key

如果 要解析的 JWT 是 JWS 或者 JWE，那就需要一个验证签名或者解密的 key。

- 用 `SecretKey` 签名的 JWS，解析时需要用同一个 `SecretKey` ：

  ```java
  Jwts.parser()
      .verifyWith(secretKey) 	// 使用相同的 secretKey
      .build()
      .parseSignedClaims(jwsString);
  ```

- 用 `PrivateKey` 签名的 JWS，解析时需要对应的 `PublicKey`：

  ```java
  Jwts.parser()
      .verifyWith(publicKey)	// 用 publicKey，不是 privateKey
      .build()
      .parseSignedClaims(jwsString);
  ```

- 用 `SecretKey` 加密的 JWE，解析时需要用对应的 `SecretKey`：

  ```java
  Jwts.parser()
      .decryptWith(secretKey)	// 或者用 Kyes.password(charArray)
      .build()
      .parseEncryptedClaims(jweString);
  ```

- 用 `PublicKey` 加密的 JWE，解析时需要用对应的 `PrivateKey`：

  ```java
  Jwts.parser()
      .decryptWith(privateKey)	// 不是 publicKey
      .build()
      .parseEncryptedClaims(jweString);
  ```

