# Base64编码

## 一、Base64编码原理

Base64编码是基于64个字符A-Z,a-z，0-9，+，/的编码方式，因为2的6次方正好为64，所以就用6bit就可以表示出64个字符，eg:000000对应A，000001对应B。

**BASE64 的编码原理：**都是按字符串长度，以每 3 个 字符（1Byte=8bit）为一组，然后针对每组，首先获取每个字符的 ASCII 编码(字符'a'=97=01100001)，然后将 ASCII 编码转换成 8 bit 的二进制，得到一组 3 * 8=24 bit 的字节。然后再将这 24 bit 划分为 4 个 6 bit 的字节，并在每个 6 bit 的字节前面都填两个高位 0，得到 4 个 8 bit 的字节，然后将这 4 个 8 bit 的字节转换成十进制，对照 BASE64 编码表 （下表），得到对应编码后的字符。

![image](https://user-gold-cdn.xitu.io/2018/8/22/1656180d4f070bcb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

注：1. 要求被编码字符是8bit的，所以须在ASCII编码范围内，\u0000-\u00ff，中文就不行。  2. 如果被编码字符长度不是3的倍数的时候，则都用0代替，对应的输出字符为“=”

**Base64编码本质**上是一种将二进制数据转成文本数据的方案。对于非二进制数据，是先将其转换成二进制形式，然后每连续6比特（2的6次方=64）计算其十进制值，根据该值在A--Z,a--z,0--9,+,/ 这64个字符中找到对应的字符，最终得到一个文本字符串。基本规则如下几点：

1. 标准Base64只有64个字符（英文大小写、数字和+、/）以及用作后缀等号；
2. Base64是把3个字节变成4个可打印字符，所以Base64编码后的字符串一定能被4整除（不算用作后缀的等号）；
3. 等号一定用作后缀，且数目一定是0个、1个或2个。这是因为如果原文长度不能被3整除，Base64要在后面添加\0凑齐3n位。为了正确还原，添加了几个\0就加上几个等号。显然添加等号的数目只能是0、1或2；
4. 严格来说Base64不能算是一种加密，只能说是编码转换。

## 二、Base64解码原理

解码原理是将4个字节转换成3个字节.先读入4个6位(用或运算),每次左移6位,再右移3次,每次8位，这样就还原了。

## 三、Base64编码字符串实例

**1、字符长度为能被3整除时：比如“Tom” ：**

![img](https://user-gold-cdn.xitu.io/2018/8/22/16561833fd196b15?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**2、字符串长度不能被3整除时，比如“Lucy”：**

![img](https://user-gold-cdn.xitu.io/2018/8/22/1656183baeeb643b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 四、Base64编码的应用

1. 实现简单的数据加密，使用户一眼望去完全看不出真实数据内容，base64算法的复杂程度要小，效率要高相对较高。

2. Base64编码的主要的作用不在于安全性，而在于让内容能在各个网关间无错的传输，这才是Base64编码的核心作用。

3. 在计算机中任何数据都是按ascii码存储的，而ascii码的128～255之间的值是不可见字符。而在网络上交换数据时，比如说从A地传到B地，往往要经过多个路由设备，由于不同的设备对字符的处理方式有一些不同，这样那些不可见字符就有可能被处理错误，这是不利于传输的。所以就先把数据先做一个Base64编码，统统变成可见字符，这样出错的可能性就大降低了。

4. Base64 编码在URL中的应用：

   Base64编码可用于在HTTP环境下传递较长的标识信息。例如，在Java持久化系统Hibernate中，就采用了Base64来将一个较长的唯一标识符（一般为128-bit的UUID）编码为一个字符串，用作HTTP表单和HTTP GET URL中的参数。在其他应用程序中，也常常需要把二进制数据编码为适合放在URL（包括隐藏表单域）中的形式。此时，采用Base64编码不仅比较简短，同时也具有不可读性，即所编码的数据不会被人用肉眼所直接看到。

   然而，标准的Base64并不适合直接放在URL里传输，因为URL编码器会把标准Base64中的“/”和“+”字符变为形如“%XX”的形式，而这些“%”号在存入数据库时还需要再进行转换，因为ANSI SQL中已将“%”号用作通配符。

   （1）为解决此问题，可采用一种用于URL的改进Base64编码，它不在末尾填充'='号，并将标准Base64中的“+”和“/”分别改成了“-”和“*”，这样就免去了在URL编解码和数据库存储时所要作的转换，避免了编码信息长度在此过程中的增加，并统一了数据库、表单等处对象标识符的格式。（2）另有一种用于正则表达式的改进Base64变种，它将“+”和“/”改成了“!”和“-”，因为“+”，“\*”以及前面在IRCu中用到的“[”和“]”在正则表达式中都可能具有特殊含义。此外还有一些变种，它们将“+/”改为“*-”或“.*”（用作编程语言中的标识符名称）或“.-”（用于XML中的Nmtoken）甚至“*:”（用于XML中的Name）。

   很多下载类网站都提供“迅雷下载”的链接，其地址通常是加密的迅雷专用下载地址。  如thunder://QUFodHRwOi8vd3d3LmJhaWR1LmNvbS9pbWcvc3NsbTFfbG9nby5naWZaWg==  其实迅雷的“专用地址”也是用Base64加密的，其加密过程如下：

   -   一、在地址的前后分别添加AA和ZZ

   [如www.baidu.com/img/sslm1_logo.gif变成](https://link.juejin.im?target=http%3A%2F%2Fxn--www-eo8e.baidu.com%2Fimg%2Fsslm1_logo.gif%25E5%258F%2598%25E6%2588%2590)[AAwww.baidu.com/img/sslm1_l…](https://link.juejin.im?target=http%3A%2F%2FAAwww.baidu.com%2Fimg%2Fsslm1_logo.gifZZ)

   -   二、对新的字符串进行Base64编码

   [如AAwww.baidu.com/img/sslm1_logo.gifZZ用Base64编码得到QUFodHRwOi8vd3d3LmJhaWR1LmNvbS9pbWcvc3NsbTFfbG9nby5naWZaWg==](https://link.juejin.im?target=http%3A%2F%2Fxn--AAwww-gv5i.baidu.com%2Fimg%2Fsslm1_logo.gifZZ%25E7%2594%25A8Base64%25E7%25BC%2596%25E7%25A0%2581%25E5%25BE%2597%25E5%2588%25B0QUFodHRwOi8vd3d3LmJhaWR1LmNvbS9pbWcvc3NsbTFfbG9nby5naWZaWg%3D%3D)

   -   三、在上面得到的字符串前加上“thunder://”就成了

   thunder://QUFodHRwOi8vd3d3LmJhaWR1LmNvbS9pbWcvc3NsbTFfbG9nby5naWZaWg==

## 五、Base64具体实现

#### 1. 对字符串进行Base64编码

```
//对字符串进行Base64编码
    public void base64Encode(View view) {
        String str = "a";
        stringBase64 = Base64.encodeToString(str.getBytes(), Base64.NO_PADDING);
        
        test.setText("a 的Base64编码为："+stringBase64);
    }
复制代码
```

#### 2. 对字符串进行Base64解码

```
//对字符串进行Base64解码
    public void base64Decode(View view) {
        byte[] decode = Base64.decode(stringBase64, Base64.DEFAULT);
        String string = new String(decode);
        
        test.setText("base64解码为："+string);
    }
复制代码
```

#### 3. 对文件进行Base64编码

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

#### 4. 对文件进行Base64编码

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

#### 5. 针对Base64.DEFAULT参数说明

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