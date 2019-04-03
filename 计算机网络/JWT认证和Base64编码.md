# JWT认证

## 前言

在前后端分离的应用中，后端主要作为 Model 层，为前端提供数据访问 API。为了保证数据安全可靠地在用户与服务端之间传输，实现服务端的认证就显得极为必要。

常见的服务端认证方法有基于 Cookie 的认证，如 session；以及 Token （令牌）认证，如 JWT。前者依赖于 cookie 而实现，在每次请求时都需要带上 cookie ，取出相应字段并与服务器端的进行对比，以实现身份的认证。而后者仅仅需要在 HTTP 的头部附上 token，由服务器 check signature 即可实现，无须担心 cookie 存在的 CORS 问题。

##  JSON Web Token

它定义了一套简洁（compact）且 URL 安全（URL-safe）的方案，以安全地在客户端和服务器之间传输 JSON 格式的信息。

## 优点

- 体积小（一串字符串），因而传输速度快
- 传输方式多样。可以通过 HTTP 头部（推荐）/URL/POST 参数等方式传输
- 严谨的结构化。它自身（在 payload 中）就包含了所有与用户相关的验证消息，如用户可访问路由、访问有效期等信息，服务器无需再去连接数据库验证信息的有效性，并且 payload 支持应用定制
- 支持跨域验证，多应用于单点登录

单点登录（Single Sign On）：在多个应用系统中，用户只需登陆一次，就可以访问所有相互信任的应用。

##  JWT优势

除了上面说到的优点之外，相比传统的服务端验证， JWT 还有以下吸引点。

1. 充分依赖无状态 API ，契合 RESTful 设计原则（无状态的 HTTP）

   首先要理解几个概念：状态，有状态 API 以及无状态 API 。

   - 状态：请求的状态是 client 与 server 交互过程中，保存下来的相关信息，客户端的保存在 page/request/session/application 或者全局作用域中，而server 的一般存在 session 中。
   - 有状态 API：server 保存了 client 的请求状态， server 通过 client 传递的 sessionID 在其 session 作用域内找到之前交互的信息并应答。
   - 无状态 API：无状态是 RESTful 架构设计的一个非常重要的原则。无状态 API 的每一个请求都是独立的，它要求客户端保存所有需要的认证信息，每次发请求都要带上自己的状态，以 url 的形式提交包含 cookies 等状态的数据。

   JWT 的设计契合无状态原则：用户登录之后，服务器会返回一串 token 并保存在本地，在这之后的服务器访问都要带上这串 token，来获得访问相关路由、服务及资源的权限。比如单点登录就比较多地使用了 JWT，因为它的体积小，并且经过简单处理（使用 HTTP 头带上 Bearer 属性+ token ）就可以支持跨域操作。

2. 易于实现 CDN，将静态资源分布式管理

   在传统的 session 验证中，服务端必须保存 session ID，用于与用户传过来的 cookie 验证。而一开始 sessionID 只会保存在一台服务器上，所以只能由一台 server 应答，就算其他服务器有空闲也无法应答，无法充分利用到分布式服务器的优点。JWT 依赖的是在客户端本地保存验证信息，不需要利用服务器保存的信息来验证，所以任意一台服务器都可以应答，服务器的资源也能被较好地利用。

3. 验证解耦，随处生成

   无需使用特定的身份验证方案，只要拥有生成 token 所需的验证信息，在何处都可以调用相应接口生成 token，无需繁琐的耦合的验证操作，可谓是一次生成，永久使用。

4. 比 cookie 更支持原生移动端应用

   原生的移动应用对 cookie 与 session 的支持不够好，而对 token 的方式支持较好。

除此之外，JWT 可靠的结构化的标准，

## JWT 工作原理

![https://pic2.zhimg.com/v2-ed3e354747522c4cecb085cf1e9be299_b.jpg](file:///C:/Users/67355/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

首先，某 client 使用自己的账号密码发送 post 请求 login，由于这是首次接触，服务器会校验账号与密码是否合法，如果一致，则根据密钥生成一个 token 并返回，client 收到这个 token 并保存在本地。在这之后，需要访问一个受保护的路由或资源时，只要附加上token（通常使用 Header 的Authorization 属性）发送到服务器，服务器就会检查这个 token 是否有效，并做出响应。

## JWT 组成

```
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  // reserved claims
  "iss": "a.com",
  "exp": "1d",
  // public claims
  "http://a.com": true,
  // private claims
  "company": "A",
  "awesome": true
}

// $Signature
HS256(Base64(Header) + "." + Base64(Payload), secretKey)

// JWT
JWT = Base64(Header) + "." + Base64(Payload) + "." + $Signature
```

JWT 是由 . 连接的三部分组成。

1. 经过 Base64 编码的 Header。Header是一个 JSON 对象，对象里有一个值为 “JWT” 的 typ 属性，以及 alg 属性，值为HS256，表明最终使用的加密算法是 HS256。
2. 经过 Base64 编码的 Payload。Payload被定义为实体的状态，就像 token 自身附加元数据一样，claim包含我们想要传输的信息，以及用于服务器验证的信息，一般有 reserved/public/private 三类。
3. Signature。它由 Header 指定的算法 HS256 加密产生。该算法有两个参数，第一个参数是经过 Base64 分别编码的 Header 及 Payload 通过. 连接组成的字符串，第二个参数是生成的密钥，由服务器保存。

注意：从这里我们可以看出，JWT 仅仅是对 payload 做了简单的 sign 和 encode 处理，并未被加密，并不能保证数据的安全性，所以建议只在其中保存非敏感的用于身份验证的数据。

## 服务端验证

服务端接收到 token 之后，会逆向构造过程，decode 出 JWT 的三个部分，这一步可以得到 sign 的算法及 payload，结合服务端配置的 secretKey，可以再次进行 Signature 的生成得到新的 Signature，与原有的 Signature 比对以验证 token 是否有效，完成用户身份的认证，验证通过才会使用 payload 的数据。 

如你所见，服务端最终只是为了验证 Signature 是否仍是自己当时下发给 client 的那个，如果验证通过，则说明该 JWT 有效并且来自可靠来源，否则说明可能是对应用程序的潜在攻击，以此完成认证。

 JWT是什么？

JWT (JSON Web Token)是一个开放标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)），指基于JSON的、用于在网络上声明某种主张的令牌（token），以保证各方之间安全的传输信息。

JWT通过将用户信息加密到token中，服务端不需要保存任何用户信息。服务端只需要通过保存的密钥来验证token正确性，如果正确即通过验证。

## JWT的组成

JWS实际上就是一个字符串，由三部分组成，头部(Header)、载荷(Payload)、签名(Signature)，并以`.`进行拼接。其中头部和载荷都是以JSON格式存放数据，只是进行了编码。

![jwt](https://img1.wangjunfeng.com/images/201808/jwt.png)

### 1. 头部(Header)

每个JWT都会带有头部信息，这里主要声明使用的算法。声明算法的字段名为`alg`，同时还有一个`typ`的字段，默认`JWT`即可。以下示例中算法为HS256。

| `1234` | `{  "alg": "HS256",  "typ": "JWT"}` |
| ------ | ----------------------------------- |
|        |                                     |

因为JWT是字符串，所以我们还需要对以上内容进行Base64编码，编码后字符串如下：

| `1`  | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9` |
| ---- | -------------------------------------- |
|      |                                        |

### 2. 载荷(Payload)

载荷即消息体，这里会存放实际的内容，也就是Token的数据声明(Claim)。这一段有一些是标准字段，当然也可以根据自己需要添加自己需要的字段。标准字段如下：

- `iss`: Token签发者。格式是区分大小写的字符串或者uri，用于唯一标识签发token的一方。
- `sub`: Token的主体，即它的所有人。格式是区分大小写的字符串或者uri。
- `aud`: 接收Token的一方。格式为区分大小写的字符串或uri，或者这两种的数组。
- `exp`: Token的过期时间，格式为时间戳。
- `nbf`: 指定Token在nbf时间之前不能使用，即token开始生效的时间，格式为时间戳。
- `iat`: Token的签发时间，格式为时间戳。
- `jti`: 指此Token的唯一标识符字符串。主要用于实现唯一性保证，防止重放。

下面是一个示例：

| `12345` | `{  "sub": "1234567890",  "name": "John Doe",  "iat": 1516239022}` |
| ------- | ---------------------------------------- |
|         |                                          |

同样进行Base64编码后，字符串如下：

| `1`  | `eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ` |
| ---- | ---------------------------------------- |
|      |                                          |

### 3. 签名(Signature)

签名是对头部和载荷内容进行签名，一旦前面两部分数据被篡改，只要服务器加密用的密钥没有泄露，得到的签名肯定和之前的签名不一致。

签名的过程：

1. 对header的json数据进行Base64URL编码，得到一个字符串str1
2. 对payload的json数据进行Base64URL编码，得到一个字符串str2
3. 使用`.`对以上两个字符串进行拼接，得到字符串str3
4. 使用header中声明的算法，以及服务端的密钥，对拼接字符串进行加密，生成签名

如果用伪代码表示就是(以HS256为例)：

| `12345` | `HMACSHA256(    base64UrlEncode(header) + "." +    base64UrlEncode(payload),    secret)` |
| ------- | ---------------------------------------- |
|         |                                          |

将三组字符串，以`.`相连，就得到了一个完整的token，例如：

| `1`  | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.OFHM3R8PSyHDT_vuzRF5fYkYWdhExM_9pE81kG05qAk` |
| ---- | ---------------------------------------- |
|      |                                          |

## 如何使用JWT？

在之前的传统的方法时在服务端存储一个session，并给客户端返回一个cookie。而如果是使用jwt来做身份鉴定的话，当用户登录成功，会给用户一个token，前端只需要在本地保存该token即可(通常使用localStorage，也可以使用cookie)。

当用户需要访问一个受保护的资源时，需要再Header中使用Bearer模式的Authorization头。其内容看起来是下面这样：

| `1`  | `Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.OFHM3R8PSyHDT_vuzRF5fYkYWdhExM_9pE81kG05qAk` |
| ---- | ---------------------------------------- |
|      |                                          |

## JWT的优缺点

优点：

- json具有通用性，所以可以跨语言。
- 组成简单，字节占用小，便于传输
- 服务端无需保存会话信息，很容易进行水平扩展
- 一处生成，多处使用，可以在分布式系统中，解决单点登录问题
- 可防护CSRF攻击

缺点：

- payload部分仅仅是进行简单编码，所以只能用于存储逻辑必需的非敏感信息
- 需要保护好加密密钥，一旦泄露后果不堪设想
- 为避免token被劫持，最好使用https协议
- 针对已经办法的令牌，无法作废，不容易应对数据过期的问题。

# Base64编码

## Base64编码原理

Base64编码是基于64个字符A-Z,a-z，0-9，+，/的编码方式，因为2的6次方正好为64，所以就用6bit就可以表示出64个字符，eg:000000对应A，000001对应B。

**BASE64 的编码原理：**都是按字符串长度，以每 3 个 字符（1Byte=8bit）为一组，然后针对每组，首先获取每个字符的 ASCII 编码(字符'a'=97=01100001)，然后将 ASCII 编码转换成 8 bit 的二进制，得到一组 3 * 8=24 bit 的字节。然后再将这 24 bit 划分为 4 个 6 bit 的字节，并在每个 6 bit 的字节前面都填两个高位 0，得到 4 个 8 bit 的字节，然后将这 4 个 8 bit 的字节转换成十进制，对照 BASE64 编码表 （下表），得到对应编码后的字符。

![image](https://user-gold-cdn.xitu.io/2018/8/22/1656180d4f070bcb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

注：1. 要求被编码字符是8bit的，所以须在ASCII编码范围内，\u0000-\u00ff，中文就不行。  2. 如果被编码字符长度不是3的倍数的时候，则都用0代替，对应的输出字符为“=”

**Base64编码本质**上是一种将二进制数据转成文本数据的方案。对于非二进制数据，是先将其转换成二进制形式，然后每连续6比特（2的6次方=64）计算其十进制值，根据该值在A--Z,a--z,0--9,+,/ 这64个字符中找到对应的字符，最终得到一个文本字符串。基本规则如下几点：

1. 标准Base64只有64个字符（英文大小写、数字和+、/）以及用作后缀等号；
2. Base64是把3个字节变成4个可打印字符，所以Base64编码后的字符串一定能被4整除（不算用作后缀的等号）；
3. 等号一定用作后缀，且数目一定是0个、1个或2个。这是因为如果原文长度不能被3整除，Base64要在后面添加\0凑齐3n位。为了正确还原，添加了几个\0就加上几个等号。显然添加等号的数目只能是0、1或2；
4. 严格来说Base64不能算是一种加密，只能说是编码转换。

## Base64解码原理

解码原理是将4个字节转换成3个字节.先读入4个6位(用或运算),每次左移6位,再右移3次,每次8位，这样就还原了。

## Base64编码字符串实例

**1、字符长度为能被3整除时：比如“Tom” ：**

![img](https://user-gold-cdn.xitu.io/2018/8/22/16561833fd196b15?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**2、字符串长度不能被3整除时，比如“Lucy”：**

![img](https://user-gold-cdn.xitu.io/2018/8/22/1656183baeeb643b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## Base64编码的应用

1. 实现简单的数据加密，使用户一眼望去完全看不出真实数据内容，base64算法的复杂程度要小，效率要高相对较高。

2. Base64编码的主要的作用不在于安全性，而在于让内容能在各个网关间无错的传输，这才是Base64编码的核心作用。

3. 在计算机中任何数据都是按ascii码存储的，而ascii码的128～255之间的值是不可见字符。而在网络上交换数据时，比如说从A地传到B地，往往要经过多个路由设备，由于不同的设备对字符的处理方式有一些不同，这样那些不可见字符就有可能被处理错误，这是不利于传输的。所以就先把数据先做一个Base64编码，统统变成可见字符，这样出错的可能性就大降低了。

4. Base64 编码在URL中的应用：

   Base64编码可用于在HTTP环境下传递较长的标识信息。例如，在Java持久化系统Hibernate中，就采用了Base64来将一个较长的唯一标识符（一般为128-bit的UUID）编码为一个字符串，用作HTTP表单和HTTP GET URL中的参数。在其他应用程序中，也常常需要把二进制数据编码为适合放在URL（包括隐藏表单域）中的形式。此时，采用Base64编码不仅比较简短，同时也具有不可读性，即所编码的数据不会被人用肉眼所直接看到。

   然而，标准的Base64并不适合直接放在URL里传输，因为URL编码器会把标准Base64中的“/”和“+”字符变为形如“%XX”的形式，而这些“%”号在存入数据库时还需要再进行转换，因为ANSI SQL中已将“%”号用作通配符。

   （1）为解决此问题，可采用一种用于URL的改进Base64编码，它不在末尾填充'='号，并将标准Base64中的“+”和“/”分别改成了“-”和“*”，这样就免去了在URL编解码和数据库存储时所要作的转换，避免了编码信息长度在此过程中的增加，并统一了数据库、表单等处对象标识符的格式。（2）另有一种用于正则表达式的改进Base64变种，它将“+”和“/”改成了“!”和“-”，因为“+”，“\*”以及前面在IRCu中用到的“[”和“]”在正则表达式中都可能具有特殊含义。此外还有一些变种，它们将“+/”改为“*-”或“.*”（用作编程语言中的标识符名称）或“.-”（用于XML中的Nmtoken）甚至“*:”（用于XML中的Name）。

   很多下载类网站都提供“迅雷下载”的链接，其地址通常是加密的迅雷专用下载地址。  如thunder://QUFodHRwOi8vd3d3LmJhaWR1LmNvbS9pbWcvc3NsbTFfbG9nby5naWZaWg==  其实迅雷的“专用地址”也是用Base64加密的，其加密过程如下：

   - 一、在地址的前后分别添加AA和ZZ

   [如www.baidu.com/img/sslm1_logo.gif变成](https://link.juejin.im?target=http%3A%2F%2Fxn--www-eo8e.baidu.com%2Fimg%2Fsslm1_logo.gif%25E5%258F%2598%25E6%2588%2590)[AAwww.baidu.com/img/sslm1_l…](https://link.juejin.im?target=http%3A%2F%2FAAwww.baidu.com%2Fimg%2Fsslm1_logo.gifZZ)

   - 二、对新的字符串进行Base64编码

   [如AAwww.baidu.com/img/sslm1_logo.gifZZ用Base64编码得到QUFodHRwOi8vd3d3LmJhaWR1LmNvbS9pbWcvc3NsbTFfbG9nby5naWZaWg==](https://link.juejin.im?target=http%3A%2F%2Fxn--AAwww-gv5i.baidu.com%2Fimg%2Fsslm1_logo.gifZZ%25E7%2594%25A8Base64%25E7%25BC%2596%25E7%25A0%2581%25E5%25BE%2597%25E5%2588%25B0QUFodHRwOi8vd3d3LmJhaWR1LmNvbS9pbWcvc3NsbTFfbG9nby5naWZaWg%3D%3D)

   - 三、在上面得到的字符串前加上“thunder://”就成了

   thunder://QUFodHRwOi8vd3d3LmJhaWR1LmNvbS9pbWcvc3NsbTFfbG9nby5naWZaWg==

## Base64具体实现

### 1. 对字符串进行Base64编码

```
//对字符串进行Base64编码
    public void base64Encode(View view) {
        String str = "a";
        stringBase64 = Base64.encodeToString(str.getBytes(), Base64.NO_PADDING);
        
        test.setText("a 的Base64编码为："+stringBase64);
    }
复制代码
```

### 2. 对字符串进行Base64解码

```
//对字符串进行Base64解码
    public void base64Decode(View view) {
        byte[] decode = Base64.decode(stringBase64, Base64.DEFAULT);
        String string = new String(decode);
        
        test.setText("base64解码为："+string);
    }
复制代码
```

### 3. 对文件进行Base64编码

```
File file = new File("/storage/emulated/0/pimsecure_debug.txt");
FileInputStream inputFile = null;
try {
    inputFile = new FileInputStream(file);
    byte[] buffer = new byte[(int) file.length()];
    inputFile.read(buffer);
    inputFile.close();
    encodedString = Base64.encodeToString(buffer, Base64.DEFAULT);
    Log.e("Base64", "Base64---->" + encodedString);
} catch (Exception e) {
    e.printStackTrace();
}
复制代码
```

### 4. 对文件进行Base64编码

```
File desFile = new File("/storage/emulated/0/pimsecure_debug_1.txt");
FileOutputStream  fos = null;
try {
    byte[] decodeBytes = Base64.decode(encodedString.getBytes(), Base64.DEFAULT);
    fos = new FileOutputStream(desFile);
    fos.write(decodeBytes);
    fos.close();
} catch (Exception e) {
    e.printStackTrace();
}
复制代码
```

### 5. 针对Base64.DEFAULT参数说明

无论是编码还是解码都会有一个参数Flags，Android提供了以下几种

1. DEFAULT 这个参数是默认，使用默认的方法来加密

   对“a”进行Base64编码结果为：YQ==，并且编码后出现换行符

2. NO_PADDING 这个参数是略去加密字符串最后的”=”

   对“a”进行Base64编码结果为：YQ

3. NO_WRAP 这个参数意思是略去所有的换行符（设置后CRLF就没用了）

   对“a”进行Base64编码结果为：YQ==，并且编码后不出现换行符

4. CRLF 这个参数看起来比较眼熟，它就是Win风格的换行符，意思就是使用CR LF这一对作为一行的结尾而不是Unix风格的LF

5. URL_SAFE 这个参数意思是加密时不使用对URL和文件名有特殊意义的字符来作为加密字符，具体就是以-和_取代+和/

   对“a”进行Base64编码结果为：YQ==，并且编码后出现换行符