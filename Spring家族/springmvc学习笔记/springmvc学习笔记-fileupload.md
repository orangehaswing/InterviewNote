# springmvc学习笔记-fileupload

# 第一节

## Controller

```
@Controller
@RequestMapping("/fileupload")
public class FileUploadController {

   @ModelAttribute
   public void ajaxAttribute(WebRequest request, Model model) {
      model.addAttribute("ajaxRequest", AjaxUtils.isAjaxRequest(request));
   }

   @GetMapping
   public void fileUploadForm() {
   }

   @PostMapping
   public void processUpload(@RequestParam MultipartFile file, Model model) {
      model.addAttribute("message", "File '" + file.getOriginalFilename() + "' uploaded successfully");
   }
   
}
```

被@ModelAttribute注释的方法会在此controller每个方法执行前被执行，因此这里首先将判断WebRequest是否用Ajax方式的请求，然后将请求添加进model。

springmvc中，通过 MultipartFile来进行文件上传。所以，如果要实现文件的上传，只要在 spring-mvc.xml 中注册相应的 MultipartResolver 即可。

MultipartResolver 的实现类有两个：

1. CommonsMultipartResolver
2. StandardServletMultipartResolver

两个的区别：

1. 第一个需要使用 Apache 的 commons-fileupload 等 jar 包支持，但它能在比较旧的 servlet 版本中使用。
2. 第二个不需要第三方 jar 包支持，它使用 servlet 内置的上传功能，但是只能在 Servlet 3 以上的版本使用。

这个项目在config中配置

```
@Bean
public MultipartResolver multipartResolver() {
	return new CommonsMultipartResolver();
}
```

## Views

fileupload.jsp

```
</c:if>
   <div id="fileuploadContent">
      <!--
          File Uploads must include CSRF in the URL.
          See http://docs.spring.io/spring-security/site/docs/3.2.x/reference/htmlsingle/#csrf-multipart
      -->
      <c:url var="actionUrl" value="fileupload?${_csrf.parameterName}=${_csrf.token}"/>
      <form id="fileuploadForm" action="${actionUrl}" method="POST" enctype="multipart/form-data" class="cleanform">
         <div class="header">
            <h2>Form</h2>
            <c:if test="${not empty message}">
               <div id="message" class="success">${message}</div>       
            </c:if>
         </div>
         <label for="file">File</label>
         <input id="file" type="file" name="file" />
         <p><button type="submit">Upload</button></p>      
      </form>
      <script type="text/javascript">
         $(document).ready(function() {
            $('<input type="hidden" name="ajaxUpload" value="true" />').insertAfter($("#file"));
            $("#fileuploadForm").ajaxForm({ success: function(html) {
                  $("#fileuploadContent").replaceWith(html);
               }
            });
         });
      </script>  
   </div>
<c:if test="${!ajaxRequest}">
</body>
```



