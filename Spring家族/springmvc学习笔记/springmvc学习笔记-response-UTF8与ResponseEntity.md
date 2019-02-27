# springmvc学习笔记-response

# 第一节

## Controller

ResponseController

```
@RestController
@RequestMapping(value="/response", method=RequestMethod.GET)
public class ResponseController {

	@GetMapping("/annotation")
	public String responseBody() {
		return "The String ResponseBody";
	}
}
```

body直接返回The String ResponseBody

```
@GetMapping("/charset/accept")
	public String responseAcceptHeaderCharset() {
		return "\u3053\u3093\u306b\u3061\u306f\u4e16\u754c\uff01 (\"Hello world!\" in Japanese)";
	}

	@GetMapping(value="/charset/produce", produces="text/plain;charset=UTF-8")
	public String responseProducesConditionCharset() {
		return "\u3053\u3093\u306b\u3061\u306f\u4e16\u754c\uff01 (\"Hello world!\" in Japanese)";
	}
```

- return的是日文，在View中使用 class="utf8TextLink" 传递body，保证不乱码
- 传递相同的日文格式，把produces中的charset设置为UTF-8同样可以保证不乱码

## View

```
<div id="responses">	
<li>
	<a id="responseBody" class="textLink" href="<c:url value="/response/annotation" />">@ResponseBody</a>			
</li>
<li>
	<a id="responseCharsetAccept" class="utf8TextLink" href="<c:url value="/response/charset/accept" />">@ResponseBody (UTF-8 charset requested)</a>
</li>
<li>
	<a id="responseCharsetProduce" class="textLink" href="<c:url value="/response/charset/produce" />">@ResponseBody (UTF-8 charset produced)</a>
</li>
</div>
```

# 第二节

## Controller

ResponseController

```
@GetMapping("/entity/status")
public ResponseEntity<String> responseEntityStatusCode() {
	return new ResponseEntity<String>("The String ResponseBody with custom status code (403 Forbidden)",HttpStatus.FORBIDDEN);
}

@GetMapping("/entity/headers")
public ResponseEntity<String> responseEntityCustomHeaders() {
	HttpHeaders headers = new HttpHeaders();
	headers.setContentType(MediaType.TEXT_PLAIN);
	return new ResponseEntity<String>("The String ResponseBody with custom header Content-Type=text/plain",headers, HttpStatus.OK);
}
```

ResponseEntity:处理HTTP响应。

ResponseEntity标识整个http相应：状态码、头部信息以及相应体内容。

MediaType.TEXT_PLAIN：纯文本格式的内容类型(Content-Type)。

HttpHeaders：HTTP协议首部

## View

home.jsp

```
<div id="responses">	
<li>
	<a id="responseEntityStatus" class="textLink" href="<c:url value="/response/entity/status" />">ResponseEntity (custom status)</a>			
</li>
<li>
	<a id="responseEntityHeaders" class="textLink" href="<c:url value="/response/entity/headers" />">ResponseEntity (custom headers)</a>			
</li>
</div>
```
