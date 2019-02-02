# springmvc学习笔记-validation

# 第一节

## Controller

```
@RestController
public class ValidationController {

   // enforcement of constraints on the JavaBean arg require a JSR-303 provider on the classpath
   
   @GetMapping("/validate")
   public String validate(@Valid JavaBean bean, BindingResult result) {
      if (result.hasErrors()) {
         return "Object has validation errors";
      } else {
         return "No errors";
      }
   }
}
```

@Valid：对JavaBean进行后台数据校验。@Valid 和 BindingResult 是一一对应的，否则spring会在校验不通过时直接抛出异常。如果有多个@Valid，那么每个@Valid后面跟着的BindingResult就是这个@Valid的验证结果，顺序不能乱。

## Views

```
<li>
   <a id="validateNoErrors" class="textLink" href="<c:url value="/validate?number=3&date=2029-07-04" />">Validate, no errors</a>
</li>
<li>
   <a id="validateErrors" class="textLink" href="<c:url value="/validate?number=3&date=2010-07-01" />">Validate, errors</a>
</li>
```

# JavaBean

```
public class JavaBean {
   
   @NotNull
   @Max(5)
   private Integer number;

   @NotNull
   @Future
   @DateTimeFormat(iso=ISO.DATE)
   private Date date;

   public Integer getNumber() {
      return number;
   }

   public void setNumber(Integer number) {
      this.number = number;
   }

   public Date getDate() {
      return date;
   }

   public void setDate(Date date) {
      this.date = date;
   }

}
```

number设定 @NotNull，@Max(5)。表示不能为空值，最大值是5。date使用@DateTimeFormat注解ISO.DATE的标准时间格式。

@Future：









