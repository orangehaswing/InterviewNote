# springmvc学习笔记-convert

# 第一节

## Controller

ConvertController

```
// requires Joda-Time on the classpath
@GetMapping("date/{value}")
public String date(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date value) {
	return "Converted date " + value;
}

@GetMapping("collection")
public String collection(@RequestParam Collection<Integer> values) {
	return "Converted collection " + values;
}

@GetMapping("formattedCollection")
public String formattedCollection(@RequestParam @DateTimeFormat(iso=ISO.DATE) Collection<Date> values) {
	return "Converted formatted collection " + values;
}
```

@DateTimeFormat注解把2010-07-04转换成标准格式:

- pattern 属性：类型为字符串。指定解析/格式化字段数据的模式，如：”yyyy-MM-dd hh:mm:ss”
- iso 属性：类型为 DateTimeFormat.ISO。指定解析/格式化字段数据的ISO模式，包括四种：ISO.NONE（不使用） -- 默认、ISO.DATE(yyyy-MM-dd) 、ISO.TIME(hh:mm:ss.SSSZ)、ISO.DATE_TIME(yyyy-MM-dd hh:mm:ss.SSSZ)
- style 属性：字符串类型。通过样式指定日期时间的格式，由两位字符组成，第一位表示日期的格式，第二位表示时间的格式：S：短日期/时间格式、M：中日期/时间格式、L：长日期/时间格式、F：完整日期/时间格式、-：忽略日期或时间格式

当存在value值是values=1&values=2&values=3&values=4&values=5或者values=1,2,3,4,5，都可以先放入集合中。

values=2010-07-04,2011-07-04，把时间放入集合中。

## Views

```
<li>
   <a id="date" class="textLink" href="<c:url value="/convert/date/2010-07-04" />">Date</a>
</li>
<li>
   <a id="collection" class="textLink" href="<c:url value="/convert/collection?values=1&values=2&values=3&values=4&values=5" />">Collection 1 (multi-value parameter)</a>
</li>
<li>
   <a id="collection2" class="textLink" href="<c:url value="/convert/collection?values=1,2,3,4,5" />">Collection 2 (single comma-delimited parameter value)</a>
</li>
<li>
   <a id="formattedCollection" class="textLink" href="<c:url value="/convert/formattedCollection?values=2010-07-04,2011-07-04" />">@Formatted Collection</a>
</li>  
```

# 第二节

## Controller

```
@GetMapping("value")
public String valueObject(@RequestParam SocialSecurityNumber value) {
	return "Converted value object " + value;
}

@GetMapping("custom")
public String customConverter(@RequestParam @MaskFormat("###-##-####") String value) {
	return "Converted '" + value + "' with a custom converter";
}
```

使用自定义类SocialSecurityNumber，把value值通过类内部的@MaskFormat("###-##-####")注解和getValue方法，转换成标准格式，其中的@MaskFormat("###-##-####")是自定义注解，其中value=123456789。

@RequestParam @MaskFormat("###-##-####") String value表示直接用注解的方式把value格式转换，其中value=123-45-6789

## Views

```
<li>
	<a id="valueObject" class="textLink" href="<c:url value="/convert/value?value=123456789" />">Custom Value Object</a>
</li>
<li>
	<a id="customConverter" class="textLink" href="<c:url value="/convert/custom?value=123-45-6789" />">Custom Converter</a>
</li>
```

# SocialSecurityNumber类

```
public final class SocialSecurityNumber {

	private final String value;
	
	public SocialSecurityNumber(String value) {
		this.value = value;
	}
	
	@MaskFormat("###-##-####")
	public String getValue() {
		return value;
	}

	public static SocialSecurityNumber valueOf(@MaskFormat("###-##-####") String value) {
		return new SocialSecurityNumber(value);
	}
}
```

# @MaskFormat

```
@Target(value={ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MaskFormat {
	String value();
}
```
