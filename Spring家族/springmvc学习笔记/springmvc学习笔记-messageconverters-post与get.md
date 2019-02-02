# springmvc学习笔记-messageconverters

- XML**（Extensible Markup Language）**是一种用来编码文档的标记语言，人和机器都能够快速理解XML文档的含义。它的其中一个目标就是能在互联网上广泛应用，现在XML已经被广泛地应用在各种应用、WEB服务和网站中。

```
<?xml version="1.0" encoding="utf-8" ?>
<me>
    <name>linji</name>
    <email>
        <personal>linji@xxx.com</personal>
        <personal>linji@xxx.cn</personal>
        <company>linji@company.com</company>
    </email>
    <learnt>
        <name>SVG</name>
        <status>0</status>
    </learnt>
    <learnt>
        <name>jQuery</name>
        <status>0.5</status>
    </learnt>
</me>
```

- JSON**（JavaScript Object Notation）**是一种轻量级的数据格式，它以”name / value”的格式来传输数据对象，JSON的目的就是为了能替代XML，现在也有很多编程语言支持JSON格式了。

```
{
    name: "linji",
    email: {
        personal: ["linji@xxx.com", "linji@xxx.cn"],
        company: "linji@company.com"
    },
    learnt: [
        {
            name: "SVG",
            status: 0
        },{
            name: "jQuery",
            status: 0.5
        }
    ]
}
```

- atom是GitHub 打造的现代编辑器，速度快，跨平台，支持各种插件以及可以异常方便地自定义扩展。

```
<feed xmlns="http://www.w3.org/2005/Atom" xmlns:georss="http://www.georss.org/georss">
<title>USGS Magnitude 1.0+ Earthquakes, Past Hour</title>
<updated>2014-03-24T07:15:02Z</updated>
<link rel="self" href="http://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/1.0_hour.atom"/>
<icon>http://earthquake.usgs.gov/favicon.ico</icon>
<entry>
<id>urn:earthquake-usgs-gov:ak:11196992</id>
<updated>2014-03-24T07:10:18.463Z</updated>
<link rel="alternate" type="text/html" href="http://earthquake.usgs.gov/earthquakes/eventpage/ak11196992"/>
<summary type="html">
...
</summary>
<georss:point>63.595 -150.6957</georss:point>
<category label="Age" term="Past Hour"/>
</entry>
</feed>
```

- RSS是基于文本的格式。它是XML（可扩展标识语言）的一种形式。通常RSS文件都是标为XML。RSS是站点用来和其他站点之间共享内容的一种简易方式（也叫聚合内容），通常被用于新闻和其他按顺序排列的网站，例如Blog。

```
<rssversion="2.0">
	<channel>
		<title>网站标题</title>
		<link>网站首页地址</link>
		<description>描述</description>
		<copyright>授权信息</copyright>
		<language>使用的语言（zh-cn表示简体中文）</language>
		<pubDate>发布的时间</pubDate>
		<lastBuildDate>最后更新的时间</lastBuildDate>
		<generator>生成器</generator>
		
		<item>
			<title>标题</title>
			<link>链接地址</link>
			<description>内容简要描述</description>
			<pubDate>发布时间</pubDate>
			<category>所属目录</category>
			<author>作者</author>
		</item>
	</channel>
</rss>
```

# 第一节

## Controller

```
// StringHttpMessageConverter

@PostMapping("/string")
public String readString(@RequestBody String string) {
   return "Read string '" + string + "'";
}

@GetMapping("/string")
public String writeString() {
   return "Wrote a string";
}
```

使用Post和Get两种不同的请求方式分别对"/messageconverters/string"请求。Post对应的class是textForm，Get对应的class是textLink。

## Views

```
<li>
   <form id="readString" class="textForm" action="<c:url value="/messageconverters/string" />" method="post">
      <input id="readStringSubmit" type="submit" value="Read a String" />
   </form>
</li>
<li>
   <a id="writeString" class="textLink" href="<c:url value="/messageconverters/string" />">Write a String</a>
</li>
```

# 第二节

## Controller

```
// Form encoded data (application/x-www-form-urlencoded)

@PostMapping("/form")
public String readForm(@ModelAttribute JavaBean bean) {
   return "Read x-www-form-urlencoded: " + bean;
}

@GetMapping("/form")
public MultiValueMap<String, String> writeForm() {
   MultiValueMap<String, String> map = new LinkedMultiValueMap<String, String>();
   map.add("foo", "bar");
   map.add("fruit", "apple");
   return map;
}
```

使用@ModelAttribute注解从form表单中获取参数将bean类型填满。

## Views

```
<li>
   <form id="readForm" action="<c:url value="/messageconverters/form" />" method="post">
      <input id="readFormSubmit" type="submit" value="Read Form Data" />    
   </form>
</li>
<li>
   <a id="writeForm" href="<c:url value="/messageconverters/form" />">Write Form Data</a>
</li>
```

# 第三节(XML)

## Controller

```
// Jaxb2RootElementHttpMessageConverter (requires JAXB2 on the classpath - useful for serving clients that expect to work with XML)

@PostMapping("/xml")
public String readXml(@RequestBody JavaBean bean) {
   return "Read from XML: " + bean;
}

@GetMapping("/xml")
public JavaBean writeXml() {
   return new JavaBean("bar", "apple");
}
```

class使用readXmlForm的形式请求响应

## Views

```
<li>
   <form id="readXml" class="readXmlForm" action="<c:url value="/messageconverters/xml" />" method="post">
      <input id="readXmlSubmit" type="submit" value="Read XML" />       
   </form>
</li>
<li>
   <a id="writeXmlAccept" class="writeXmlLink" href="<c:url value="/messageconverters/xml" />">Write XML via Accept=application/xml</a>
</li>
```

readXmlForm

```
$("form.readXmlForm").submit(function() {
   var form = $(this);
   var button = form.children(":first");
   $.ajax({ type: "POST", url: form.attr("action"), data: "<?xml version=\"1.0\" encoding=\"UTF-8\" standalone=\"yes\"?><javaBean><foo>bar</foo><fruit>apple</fruit></javaBean>", contentType: "application/xml", dataType: "text", success: function(text) { MvcUtil.showSuccessResponse(text, button); }, error: function(xhr) { MvcUtil.showErrorResponse(xhr.responseText, button); }});
   return false;
});
```

writeXmlLink

```
$("a.writeXmlLink").click(function() {
   var link = $(this);
   $.ajax({ url: link.attr("href"),
      beforeSend: function(req) { 
         if (!this.url.match(/\.xml$/)) {
            req.setRequestHeader("Accept", "application/xml");
         }
      },
      success: function(xml) {
         MvcUtil.showSuccessResponse(MvcUtil.xmlencode(xml), link);
      },
      error: function(xhr) { 
         MvcUtil.showErrorResponse(xhr.responseText, link);
      }
   });
   return false;
});             
```

# 第四节(Json)

## Controller

```
// MappingJacksonHttpMessageConverter (requires Jackson on the classpath - particularly useful for serving JavaScript clients that expect to work with JSON)

@PostMapping("/json")
public String readJson(@Valid @RequestBody JavaBean bean) {
   return "Read from JSON: " + bean;
}

@GetMapping("/json")
public JavaBean writeJson() {
   return new JavaBean("bar", "apple");
}
```

使用json的方式请求响应

## Views

```
<li>
   <form id="readJson" class="readJsonForm" action="<c:url value="/messageconverters/json" />" method="post">
      <input id="readJsonSubmit" type="submit" value="Read JSON" />  
   </form>
</li>
<li>
   <form id="readJsonInvalid" class="readJsonForm invalid" action="<c:url value="/messageconverters/json" />" method="post">
      <input id="readInvalidJsonSubmit" type="submit" value="Read invalid JSON (400 response code)" />   
   </form>
</li>
<li>
   <a id="writeJsonAccept" class="writeJsonLink" href="<c:url value="/messageconverters/json" />">Write JSON via Accept=application/json</a>
</li>
            <li>
                <a id="writeJsonExt" class="writeJsonLink" href="<c:url value="/messageconverters/json.json" />">Write JSON via ".json"</a>
            </li>
```

readJsonForm

```
$("form.readJsonForm").submit(function() {
   var form = $(this);
   var button = form.children(":first");
   var data = form.hasClass("invalid") ?
         "{ \"foo\": \"bar\" }" : 
         "{ \"foo\": \"bar\", \"fruit\": \"apple\" }";
   $.ajax({ type: "POST", url: form.attr("action"), data: data, contentType: "application/json", dataType: "text", success: function(text) { MvcUtil.showSuccessResponse(text, button); }, error: function(xhr) { MvcUtil.showErrorResponse(xhr.responseText, button); }});
   return false;
});
```

writeJsonLink

```
$("a.writeJsonLink").click(function() {
   var link = $(this);
   $.ajax({ url: this.href,
      beforeSend: function(req) {
         if (!this.url.match(/\.json$/)) {
            req.setRequestHeader("Accept", "application/json");
         }
      },
      success: function(json) {
         MvcUtil.showSuccessResponse(JSON.stringify(json), link);
      },
      error: function(xhr) {
         MvcUtil.showErrorResponse(xhr.responseText, link);
      }});
   return false;
});
```

# 第五节(atom)

## Controller

```
// AtomFeedHttpMessageConverter (requires Rome on the classpath - useful for serving Atom feeds)

@PostMapping("/atom")
public String readFeed(@RequestBody Feed feed) {
   return "Read " + feed.getTitle();
}

@GetMapping("/atom")
public Feed writeFeed() {
   Feed feed = new Feed();
   feed.setFeedType("atom_1.0");
   feed.setTitle("My Atom feed");
   return feed;
}
```

使用atom方式请求访问

## Views

```
<li>
   <form id="readAtom" action="<c:url value="/messageconverters/atom" />" method="post">
      <input id="readAtomSubmit" type="submit" value="Read Atom" />     
   </form>
</li>
<li>
   <a id="writeAtom" href="<c:url value="/messageconverters/atom" />">Write Atom</a>
</li>
```

readAtom

```
$("#readAtom").submit(function() {
   var form = $(this);
   var button = form.children(":first");
   $.ajax({ type: "POST", url: form.attr("action"), data: '<?xml version="1.0" encoding="UTF-8"?> <feed xmlns="http://www.w3.org/2005/Atom"><title>My Atom feed</title></feed>', contentType: "application/atom+xml", dataType: "text", success: function(text) { MvcUtil.showSuccessResponse(text, button); }, error: function(xhr) { MvcUtil.showErrorResponse(xhr.responseText, button); }});
   return false;
});
```

writeAtom

```
$("#writeAtom").click(function() {
   var link = $(this);
   $.ajax({ url: link.attr("href"),
      beforeSend: function(req) { 
         req.setRequestHeader("Accept", "application/atom+xml");
      },
      success: function(feed) {
         MvcUtil.showSuccessResponse(MvcUtil.xmlencode(feed), link);
      },
      error: function(xhr) { 
         MvcUtil.showErrorResponse(xhr.responseText, link);
      }
   });
   return false;
});
```

# 第六节(rss)

## Controller

```
@PostMapping("/rss")
public String readChannel(@RequestBody Channel channel) {
   return "Read " + channel.getTitle();
}

@GetMapping("/rss")
public Channel writeChannel() {
   Channel channel = new Channel();
   channel.setFeedType("rss_2.0");
   channel.setTitle("My RSS feed");
   channel.setDescription("Description");
   channel.setLink("http://localhost:8080/mvc-showcase/rss");
   return channel;
}
```

使用rss方式请求访问

## Views

```
<li>
   <form id="readRss" action="<c:url value="/messageconverters/rss" />" method="post">
      <input id="readRssSubmit" type="submit" value="Read Rss" />    
   </form>
</li>
<li>
   <a id="writeRss" href="<c:url value="/messageconverters/rss" />">Write Rss</a>
</li>
```

readRss

```
$("#readRss").submit(function() {
   var form = $(this);
   var button = form.children(":first");
   $.ajax({ type: "POST", url: form.attr("action"), data: '<?xml version="1.0" encoding="UTF-8"?> <rss version="2.0"><channel><title>My RSS feed</title></channel></rss>', contentType: "application/rss+xml", dataType: "text", success: function(text) { MvcUtil.showSuccessResponse(text, button); }, error: function(xhr) { MvcUtil.showErrorResponse(xhr.responseText, button); }});
   return false;
});
```

writeRss

```
$("#writeRss").click(function() {
   var link = $(this);    
   $.ajax({ url: link.attr("href"),
      beforeSend: function(req) { 
         req.setRequestHeader("Accept", "application/rss+xml");
      },
      success: function(feed) {
         MvcUtil.showSuccessResponse(MvcUtil.xmlencode(feed), link);
      },
      error: function(xhr) { 
         MvcUtil.showErrorResponse(xhr.responseText, link);
      }
   });
   return false;
});
```

# JavaBean

```
@XmlRootElement
public class JavaBean {
   
   @NotNull
   private String foo;

   @NotNull
   private String fruit;

   public JavaBean() {
   }

   public JavaBean(String foo, String fruit) {
      this.foo = foo;
      this.fruit = fruit;
   }

   public String getFoo() {
      return foo;
   }

   public void setFoo(String foo) {
      this.foo = foo;
   }

   public String getFruit() {
      return fruit;
   }

   public void setFruit(String fruit) {
      this.fruit = fruit;
   }
   
   @Override
   public String toString() {
      return "JavaBean {foo=[" + foo + "], fruit=[" + fruit + "]}";
   }

}
```













