# 前言
和前端进行数据交互时或者和第三方商家对接时，需要对隐私数据进行加密。单向加密，对称加密，非对称加密，其对应的算法也各式各样。java提供了统一的框架来规范（java.security）安全加密这类API。下面将一一介绍
-	加密算法概念及分类
-	秘钥生成
-	摘要算法工具-MessageDigest
-	签名算法工具-Signature
-	常用加密工具类-Cipher
-	Certificate-证书的保存 
-	KeyStore-密钥证书的实体类
-	https证书加载

关注公众号，一起交流；微信搜一搜: 潜行前行
---
# 1 加密算法概念及分类
常用的加密算法类型有三种，如下:
-	单向加密：也就是不可逆的加密，例如MD5，SHA，HMAC
-	对称加密：也就是加密方和解密方利用同一个秘钥对数据进行加密和解密，例如DES，PBE等等
-	非对称加密：非对称加密分为公钥和秘钥，二者是非对称的，例如用私钥加密的内容需要使用公钥来解密，使用公钥加密的内容需要用私钥来解密，DSA，RSA

# 2 秘钥生成

## 对称加密密钥的生成
-	KeyGenerator用于生成对称秘钥(可逆加密)，或者一个密码性秘钥
-	支持算法：AES、ARCFOUR、DES、DESede、HmacMD5、HmacSHA1、HmacSHA224、HmacSHA256、HmacSHA384、HmacSHA512、RC2
```java
public static final KeyGenerator getInstance(String algorithm, String provider)
public static final KeyGenerator getInstance(String algorithm)
public final void init(int keysize)
public final void init(int keysize, SecureRandom random)
public final void init(SecureRandom random)
public final void init(AlgorithmParameterSpec params, SecureRandom random)
public final SecretKey generateKey()
```
-	示例
```java
public static void main(String[] args) throws  Exception {
    SecretKey secretKey = generatorDesKey();
    System.out.println(secretKey);
}
public static SecretKey generatorDesKey() throws NoSuchAlgorithmException {
    KeyGenerator keyGen = KeyGenerator.getInstance("DES");
    SecureRandom random = new SecureRandom();
    random.nextBytes(new byte[128]);
    keyGen.init(56,random);
    SecretKey key = keyGen.generateKey();
    return key;
}
------------输出结果------------------
com.sun.crypto.provider.DESKey@185c3
```
## 非对称加密秘钥的生成
-	KeyPairGenerator用于生成非对称加密算法的密钥对KeyPair，KeyPair会包括一个公钥和私钥
-	支持算法：DiffieHellman、DSA、RSA、RSASSA-PSS、EC
```java
//KeyPairGenerator.java
public static KeyPairGenerator getInstance(String algorithm)
public static KeyPairGenerator getInstance(String algorithm, String provider)
public void initialize(int keysize, SecureRandom random)
public void initialize(AlgorithmParameterSpec params, SecureRandom random)
public final KeyPair genKeyPair() 
//KeyPair.java
public PublicKey getPublic()
public PrivateKey getPrivate()
```
-	示例
```java
public static void main(String[] args) throws Exception {
    KeyPair keyPair = generatorRsaKey();
    System.out.println(keyPair);
}
public static KeyPair generatorRsaKey() throws Exception {
    KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
    SecureRandom random = new SecureRandom();
    random.nextBytes(new byte[516]);
    keyGen.initialize(516,random);
    KeyPair keyPair = keyGen.genKeyPair();
    System.out.println(keyPair.getPrivate());
    System.out.println(keyPair.getPublic());
    return keyPair;
}
```
-	输出结果
```java
SunRsaSign RSA private CRT key, 516 bits
  params: null
  modulus: 126519853979546358862851378153247782379894323767375778571361894186790679401365500006956495592162216057219204240578435837612184688685910973224797092901015673
  private exponent: 84346569319697572575234252102165188253262882511583852380907929457860452934243188047935652497010382336410866699832067872276413297543254894848799721123249067
Sun RSA public key, 516 bits
  params: null
  modulus: 126519853979546358862851378153247782379894323767375778571361894186790679401365500006956495592162216057219204240578435837612184688685910973224797092901015673
  public exponent: 3
java.security.KeyPair@5010be6
```
## 密钥Key和密钥规格KeySpec的相互转化
>If the key is stored on a hardware device, its specification may contain information that helps identify the key on the device

>KeySpec是一个接口，用来组成加密密钥的密钥内容的（透明）规范。如果密钥存储在硬件设备上，则其规范可以包含有助于标识该设备上的密钥的信息
-	KeySpec具有规范性，所以一般会根据外部参数生成KeySpec，再根据KeySpec生成对应的Key（个人理解，如有高见，请说出你的见解）。SecretKeyFactory、KeyFactory的作用就是转换Key与KeySpec
### SecretKeyFactory：用于对称加密的密钥和密钥规格之间的转换，配合KeyGenerator使用
-	支持算法：AES、ARCFOUR、DES、DESede、PBEWithMD5AndDES、PBEWithHmacSHA256AndAES_128、PBKDF2WithHmacSHA256
```java
public static final SecretKeyFactory getInstance(String algorithm)
public static final SecretKeyFactory getInstance(String algorithm, String provider)
public final SecretKey translateKey(SecretKey key)
public final SecretKey generateSecret(KeySpec keySpec)
public final KeySpec getKeySpec(SecretKey key, Class<?> keySpec)
```
-	示例
```java
public static void main(String[] args) throws Exception {
    SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
    byte[] DESKey = "helloWWW".getBytes(StandardCharsets.UTF_8);// 设置密钥
    DESKeySpec keySpec = new DESKeySpec(DESKey);// 设置密钥参数
    SecretKey key = keyFactory.generateSecret(keySpec);// 得到密钥对象
    System.out.println(key);
}
------------输出结果------------------
com.sun.crypto.provider.DESKey@18e49
```
### KeyFactory：用于非对称加密的密钥和密钥规格之间的转换，配合KeyPairGenerator使用
-	支持算法：DiffieHellman、DSA、RSA、RSASSA-PSS、EC

```java
//KeyFactory.java
public static KeyFactory getInstance(String algorithm)
public static KeyFactory getInstance(String algorithm, String provider)
public final PublicKey generatePublic(KeySpec keySpec)
public final PrivateKey generatePrivate(KeySpec keySpec)
public final <T extends KeySpec> T getKeySpec(Key key, Class<T> keySpec)
```
-	示例
```java
public static void main(String[] args) throws Exception {
    //生成RSA秘钥对；generatorRsaKey是上面示例提供的函数
    KeyPair keyPair = generatorRsaKey();
    System.out.println(keyPair);
    //PublicKey转KeySpec;KeySpec再转PublicKey
    X509EncodedKeySpec pubKeySpec = new X509EncodedKeySpec(keyPair.getPublic().getEncoded());
    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    PublicKey pubKey = keyFactory.generatePublic(pubKeySpec);
    System.out.println(pubKey);
    //PrivateKey转KeySpec;KeySpec再转PrivateKey
    PKCS8EncodedKeySpec priKeySpec = new PKCS8EncodedKeySpec(keyPair.getPrivate().getEncoded());
    PrivateKey priKey = keyFactory.generatePrivate(priKeySpec);
    System.out.println(priKey);
}
```
-	输出结果
```java
java.security.KeyPair@78e03bb5
Sun RSA public key, 1024 bits
  params: null
  modulus: 94134923375030889337699664145116176095803777687781162111756914700229869014912695784710407302811615186395818803402552376808400599961548587586207216709744471870318354813036696801675648731428269930963470277811176883827680414539855481218813862408748594430021606927061565116386180650249935749556615770533203721821
  public exponent: 65537
SunRsaSign RSA private CRT key, 1024 bits
  params: null
  modulus: 94134923375030889337699664145116176095803777687781162111756914700229869014912695784710407302811615186395818803402552376808400599961548587586207216709744471870318354813036696801675648731428269930963470277811176883827680414539855481218813862408748594430021606927061565116386180650249935749556615770533203721821
  private exponent: 67868152791098303572124282937222322055125020915630253288684471666171190487123683962152169691286583419399765605089805755591451063493647416931630849589322449230367252892862038338916192807582203337302166911147185956153147905653905702289234855039234840869874793012808454810161546053566242403672442319692325665473

```
# 3 摘要算法-MessageDigest和javax.crypto.Mac（HMAC）
-	单向加密是不可逆的，MD5、SHA、MAC都是属于单向加密算法的一种，也称之为摘要算法
-	MD5、SHA它们会根据明文用哈希算法计算一个固定长度的摘要（哈希值），然后把明文和摘要发送给接收者，接收者根据同样的算法计算出摘要，对比两个摘要是否一样即可验证明文的正确性，它的应用场景是：防止篡改和校验数据
-	MD5、SHA等算法是开源的，容易被试探出来。有没有更安全的摘要算法呢？HMAC-带密钥(密码)的hash函数，用一个密钥和一个明文消息作为输入，生成一个消息摘要。密钥一般使用KeyGenerator创建，相当于一个密码值，其被试探出的概率小
-	MessageDigest支持的算法：MD2、MD5、SHA-1、SHA-224、SHA-256、SHA-384、SHA-512、SHA-512/224、SHA-512/256
-	javax.crypto.Mac支持的算法：HmacMD5、HmacSHA1、HmacSHA224、HmacSHA256、HmacSHA384、HmacSHA512、PBEWithHmacSHA1
-	MD5的示例
```java
MessageDigest digest = MessageDigest.getInstance("MD5");
System.out.println(new String(digest.digest("hello world!".getBytes())));
System.out.println(new String(digest.digest("hello world!".getBytes())));
------------输出结果------------------
0���G?�w
0���G?�w
```
-	MAC的示例
```
public static void main(String[] args) throws Exception {
    // 初始化HmacMD5摘要算法的密钥产生器
    KeyGenerator generator = KeyGenerator.getInstance("HmacMD5");
    // 产生密钥
    SecretKey secretKey = generator.generateKey();
    //SecretKeySpec继承于SecretKey和KeySpec，因此可直接用SecretKeySpec初始化Mac
    //SecretKey secretKey = new SecretKeySpec("password".getBytes(), "HmacMD5");
    Mac mac = Mac.getInstance("HmacMD5");
    mac.init(secretKey);
    //计算摘要
    String data = "hello world";
    byte[] result1 = mac.doFinal(data.getBytes());
    byte[] result2 = mac.doFinal(data.getBytes());
    System.out.println(new String(result1).equals(new String(result2)));
}
------------输出结果------------------    
true
```
# 4 签名算法工具-Signature
-	签名算法其实也是加密算法，它加密后的数据具有唯一标识性，就像一个人的签名能代表一个人身份。签名一般是指用非对称加密算法的私钥来加密明文的过程，生成的密文可以被持有公钥的人识别解密，只要你的公钥是准确对应无误的，就能保证你解密的数据是来自持有私钥的一方
-	如何保证公钥是正确无误，没被篡改的？1:一对一给你，2:获取公钥后通过权威机构认证，相关过程可以看下之前写的一篇文章[网络篇：朋友面试之https认证加密过程](https://juejin.cn/post/6918203360315179022)
-	支持算法：NONEwithRSA、MD2withRSA、MD5withRSA、SHA512/224withRSA、SHA512/256withRSA、RSASSA-PSS、NONEwithDSA、SHA512withDSA、NONEwithECDSA、SHA512withECDSA、MD5withRSAandMGF1（太多了，选择列举几个）
-	Signature.API示例，配合KeyPairGenerator使用
```java
public static void main(String[] args) throws Exception {
    KeyPair keyPair = generatorRsaKey();
    Signature signature = Signature.getInstance("MD5withRSA");
    signature.initSign(keyPair.getPrivate());
    //加解密数据
    byte[] data = "hello world".getBytes();
    //数据签名
    signature.update(data);
    byte[] digest = signature.sign();
    //数据解密加验证
    signature.initVerify(keyPair.getPublic());
    signature.update(data);
    System.out.println("验证结果:"+signature.verify(digest));
}
------------输出结果------------------
验证结果:true
```

# 5 常用加密工具类-Cipher
-	用于加密/解密数据。支持各种类型的算法：对称加密（例如AES），非对称加密（例如RSA）
-	支持算法：AES、AESWrap、ARCFOUR、Blowfish、DES、DESede、DESedeWrap、ECIES、RSA（太多了，选择列举几个）
-	示例
```java
public static void main(String[] args) throws Exception {
    KeyPair keyPair = generatorRsaKey();
    Cipher cipher = Cipher.getInstance("RSA");
    // 编码前设定编码方式及密钥
    cipher.init(Cipher.ENCRYPT_MODE, keyPair.getPrivate());
    //加解密数据
    byte[] data = "hello world".getBytes();
    //数据签名
    byte[] enData = cipher.doFinal(data);
    //数据解密
    cipher.init(Cipher.DECRYPT_MODE, keyPair.getPublic());
    byte[] newData = cipher.doFinal(enData);
    System.out.println("验证结果:"+new String(newData));
}
------------输出结果------------------
验证结果:hello world
```

# 6 Certificate-证书存储 
-	CertificateFactory：用于创建公钥证书(Certificate)和证书吊销列表（CRL）
-	Certificate及其子类X509Certificate
-	CertPath和CertPathBuilder：用于构建证书链（也称为证书路径）
-	CertPathValidator：用于验证证书链
-	CRL：证书吊销列表
-	CertStore：用于存储检索证书和CRL
-	CertificateFactory和Certificate的示例
-	示例
```java
//certificateStream是证书的输入流
public static PublicKey getPublicKeyByCer(InputStream certificateStream) throws Exception{
    CertificateFactory certificateFactory = CertificateFactory.getInstance("X509");
    Certificate certificate = certificateFactory.generateCertificate(certificateStream);
    return certificate.getPublicKey();
}
```

# 7 KeyStore-密钥证书的实体类
-	KeyStore用于存储私钥和证书(公钥在证书Certificate里面)
-	公钥：是一个详细的实体的数字关联,并有意让所有想同这个实体发生信任关系的其他实体知道.公共钥匙用来检验签名; 
-	私钥：是一些数字,私有和公共钥匙存在所有用公共钥匙加密的系统的钥匙对中.公共钥匙用来加密数据，私有钥匙用来计算签名.公钥加密的消息只能用私钥解密，私钥签名的消息只能用公钥检验签名。 
-	示例
```java
public static void main(String[] args) throws Exception {
    InputStream certificateStream = null;
    //根据Certificate生成KeyStore
    CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
    KeyStore keyStore = KeyStore.getInstance("PKCS12");
    keyStore.load(null);
    keyStore.setCertificateEntry("certificate", certificateFactory.generateCertificate(certificateStream));
    //加载jks文件，并生成KeyStore
    KeyStore trustKeyStore = KeyStore.getInstance("jks");
    FileInputStream trustKeyStoreFile = new FileInputStream("/root/trustKeyStore.jks");
    trustKeyStore.load(trustKeyStoreFile, "password".toCharArray());
}
```

# 8 java.https加载证书的API
-	KeyManagerFactory、TrustManagerFactory => KeyManager、TrustManager => SSLContext => SSLEngine、SSLSocketFactory、SSLSocket 
## 一般的证书加载过程
-	用Certificate、KeyStore生成创建KeyManagerFactory和TrustManagerFactory
-	KeyManagerFactory和TrustManagerFactory用来创建KeyManager和TrustManager
-	而KeyManager和TrustManager用来初始化SSLContext
-	然后使用SSLContext，创建实际实现SSL/TLS协议的对象（SSLSocketFactory、SSLSocket或者SSLEngine）
-	SSLSocket和SSLEngine可以直接在通信对象中使用
-	KeyManager和TrustManager作用：
	*	KeyManager负责向对等端显示使用的凭证（使用的密码标准、加密算法、证书、公钥、签名等）
	*	TrustManager负责验证从对等端收到的凭证，验证凭证有多种方式：其中之一是创建CertPath对象，并让JDK的内置公钥基础结构（PKI）框架处理验证。 在内部，CertPath实现可能会创建一个Signature对象，并使用它来验证证书链中的每个签名
-	示例：生成SSLContext，并使用SSLContext初始化apache-httpClient
```java
public static String postWithSSL(String url, String jsonBody) throws Exception {
    SSLContext sslContext = getSslContext();
    SSLConnectionSocketFactory sslConnectionSocketFactory = new SSLConnectionSocketFactory(
            sslContext, new String[]{"TLSv1.2", "TLSv1.1", "TLSv1"}, null,
            SSLConnectionSocketFactory.getDefaultHostnameVerifier());
    RequestConfig config = RequestConfig.custom()
            .setConnectTimeout(3000)
            .setSocketTimeout(3000)
            .build();
    CloseableHttpClient client = HttpClients.custom()
            .setSSLSocketFactory(sslConnectionSocketFactory)
            .setDefaultRequestConfig(config).build();
    HttpPost httpPost = new HttpPost(url);
    //httpPost.setHeaders(headers);
    httpPost.setHeader("Content-Type", "application/json; charset=utf-8");
    httpPost.setHeader("Accept", "application/json");
    httpPost.setEntity(new StringEntity(jsonBody, StandardCharsets.UTF_8));
    HttpResponse response = client.execute(httpPost);
    HttpEntity responseEntity = response.getEntity();
    String result = EntityUtils.toString(responseEntity, "UTF-8");
    return result;
}
//双向加密 SSLContext
private static SSLContext getSslContext() throws Exception {
    //自身私钥
    KeyStore identityKeyStore = KeyStore.getInstance("jks");
    FileInputStream identityKeyStoreFile = new FileInputStream("/root/myServer.jks");
    identityKeyStore.load(identityKeyStoreFile, "password1".toCharArray());
    //服务端信任证书
    KeyStore trustKeyStore = KeyStore.getInstance("jks");
    FileInputStream trustKeyStoreFile = new FileInputStream("/root/trustKeyStore.jks");
    trustKeyStore.load(trustKeyStoreFile, "password".toCharArray());
    //构建SSLContexts
    return SSLContexts.custom()
            .loadKeyMaterial(identityKeyStore, "password1".toCharArray()) // load identity keystore
            .loadTrustMaterial(trustKeyStore, null) // load trust keystore
            .build();
}
//双向加密 SSLContext 方式二
private static SSLContext getSslContext2() throws Exception{
    //自身私钥
    KeyManagerFactory keyFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
    KeyStore keystore = KeyStore.getInstance("jks");
    keystore.load(new FileInputStream(new File("/root/myServer.jks")), "password".toCharArray());
    keyFactory.init(keystore, "password".toCharArray());
    KeyManager[] keyManagers = keyFactory.getKeyManagers();
    //服务端信任证书
    TrustManagerFactory trustFactory = TrustManagerFactory.getInstance("SunX509");
    KeyStore tsStore = KeyStore.getInstance("jks");
    tsStore.load(new FileInputStream(new File("/root/trustKeyStore.jks")), "password".toCharArray());
    trustFactory.init(tsStore);
    TrustManager[] trustManagers = trustFactory.getTrustManagers();
    //初始化SSLContext
    SSLContext sslContext = SSLContext.getInstance("TLS");
    sslContext.init(keyManagers, trustManagers, null);
    return sslContext;
}
```

欢迎指正文中错误
---

# 参考文章
-	[JCA-Java加密框架](https://www.jianshu.com/p/a8194c237363)
-	[Java加密框架(JCA)简要说明](https://www.cnblogs.com/libin6505/p/10442194.html)
-	[Java加密解密之MAC](https://blog.csdn.net/x_san3/article/details/80613605)
-	[关于keyGenerator,KeyPairGenerator,SecretKeyFactory的解析](https://www.cnblogs.com/jeremy-blog/p/5289731.html)
-	[JCA 实践记录——SecretKeyFactory](https://www.jianshu.com/p/f66a0b93b919)
-	[HttpClient 双向认证](https://blog.csdn.net/xintonghanchuang/article/details/103298943)
-	[java内置可用加密算法文档](https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#KeyGenerator)
-	[key解析](https://www.cnblogs.com/littlefishxu/p/3969180.html)