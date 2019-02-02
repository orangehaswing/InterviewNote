# springboot学习笔记-使用Spring Boot发送邮件

在项目的维护过程中，我们通常会在应用中加入短信或者邮件预警功能，比如当应用出现异常宕机时应该及时地将预警信息发送给运维或者开发人员，本文将介绍如何在Spring Boot中发送邮件。

在Spring Boot中发送邮件使用的是Spring提供的`org.springframework.mail.javamail.JavaMailSender`，其提供了许多简单易用的方法，可发送简单的邮件、HTML格式的邮件、带附件的邮件，并且可以创建邮件模板。

# 引入依赖

在Spring Boot中发送邮件，需要用到`spring-boot-starter-mail`，引入`spring-boot-starter-mail`：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

# 邮件配置

在application.properties中进行简单的配置（以qq邮件为例）：

```
spring.mail.host=smtp.qq.com
spring.mail.username=**@qq.com
#qq邮箱密码
spring.mail.password=*******
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
```

`spring.mail.username`，`spring.mail.password`填写自己的邮箱账号密码即可。

# 发送简单的邮件

编写`EmailController`，注入`JavaMailSender`:

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Autowired
    private JavaMailSender jms;
    
    @Value("${spring.mail.username}")
    private String from;
    
    @RequestMapping("sendSimpleEmail")
    public String sendSimpleEmail() {
        try {
            SimpleMailMessage message = new SimpleMailMessage();
            message.setFrom(from);
            message.setTo("888888@qq.com"); // 接收地址
            message.setSubject("一封简单的邮件"); // 标题
            message.setText("使用Spring Boot发送简单邮件。"); // 内容
            jms.send(message);
            return "发送成功";
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }
}
```

启动项目访问[http://localhost/email/sendSimpleEmail](http://localhost/email/sendSimpleEmail)，提示发送成功：

![QQ截图20180509111645.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20180509111645.png)

# 发送HTML格式的邮件

改造`EmailController`，`SimpleMailMessage`替换为`MimeMessage`：

```
import javax.mail.internet.MimeMessage;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Autowired
    private JavaMailSender jms;
    
    @Value("${spring.mail.username}")
    private String from;
    
    @RequestMapping("sendHtmlEmail")
    public String sendHtmlEmail() {
        MimeMessage message = null;
        try {
            message = jms.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from); 
            helper.setTo("888888@qq.com"); // 接收地址
            helper.setSubject("一封HTML格式的邮件"); // 标题
            // 带HTML格式的内容
            StringBuffer sb = new StringBuffer("<p style='color:#42b983'>使用Spring Boot发送HTML格式邮件。</p>");
            helper.setText(sb.toString(), true);
            jms.send(message);
            return "发送成功";
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }
}

```

`helper.setText(sb.toString(), true);`中的`true`表示发送HTML格式邮件。启动项目，访问[http://localhost/email/sendHtmlEmail](http://localhost/email/sendHtmlEmail)，提示发送成功，可看到文本已经加上了颜色`#42b983`：

![QQ截图20180509112837.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20180509112837.png)

# 发送带附件的邮件

发送带附件的邮件和普通邮件相比，其实就只是多了个传入附件的过程。不过使用的仍是`MimeMessage`：

```
package com.springboot.demo.controller;

import java.io.File;

import javax.mail.internet.MimeMessage;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Autowired
    private JavaMailSender jms;
    
    @Value("${spring.mail.username}")
    private String from;
	
    @RequestMapping("sendAttachmentsMail")
    public String sendAttachmentsMail() {
        MimeMessage message = null;
        try {
            message = jms.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from); 
            helper.setTo("888888@qq.com"); // 接收地址
            helper.setSubject("一封带附件的邮件"); // 标题
            helper.setText("详情参见附件内容！"); // 内容
            // 传入附件
            FileSystemResource file = new FileSystemResource(new File("src/main/resources/static/file/项目文档.docx"));
            helper.addAttachment("项目文档.docx", file);
            jms.send(message);
            return "发送成功";
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }
}

```

启动项目访问[http://localhost/email/sendAttachmentsMail](http://localhost/email/sendAttachmentsMail)，提示发送成功：

![QQ截图20180510101405.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20180510101405.png)

# 发送带静态资源的邮件

发送带静态资源的邮件其实就是在发送HTML邮件的基础上嵌入静态资源（比如图片），嵌入静态资源的过程和传入附件类似，唯一的区别在于需要标识资源的cid：

```
package com.springboot.demo.controller;

import java.io.File;

import javax.mail.internet.MimeMessage;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Autowired
    private JavaMailSender jms;
    
    @Value("${spring.mail.username}")
    private String from;
	
    @RequestMapping("sendInlineMail")
    public String sendInlineMail() {
        MimeMessage message = null;
        try {
            message = jms.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from); 
            helper.setTo("888888@qq.com"); // 接收地址
            helper.setSubject("一封带静态资源的邮件"); // 标题
            helper.setText("<html><body>博客图：<img src='cid:img'/></body></html>", true); // 内容
            // 传入附件
            FileSystemResource file = new FileSystemResource(new File("src/main/resources/static/img/sunshine.png"));
            helper.addInline("img", file); 
            jms.send(message);
            return "发送成功";
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }
}

```

`helper.addInline("img", file);`中的img和图片标签里cid后的名称相对应。启动项目访问[http://localhost/email/sendInlineMail](http://localhost/email/sendInlineMail)，提示发送成功：

![QQ截图20180510111000.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20180510111000.png)

# 使用模板发送邮件

在发送验证码等情况下可以创建一个邮件的模板，唯一的变量为验证码。这个例子中使用的模板解析引擎为Thymeleaf，所以首先引入Thymeleaf依赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

```

在template目录下创建一个`emailTemplate.html`模板：

```
<!DOCTYPE html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8" />
    <title>模板</title>
</head>

<body>
    您好，您的验证码为{code}，请在两分钟内使用完成操作。
</body>
</html>

```

发送模板邮件，本质上还是发送HTML邮件，只不过多了绑定变量的过程，详细如下所示：

```
package com.springboot.demo.controller;

import java.io.File;

import javax.mail.internet.MimeMessage;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Autowired
    private JavaMailSender jms;
    
    @Value("${spring.mail.username}")
    private String from;
    
    @Autowired
    private TemplateEngine templateEngine;
	
    @RequestMapping("sendTemplateEmail")
    public String sendTemplateEmail(String code) {
        MimeMessage message = null;
        try {
            message = jms.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from); 
            helper.setTo("888888@qq.com"); // 接收地址
            helper.setSubject("邮件摸板测试"); // 标题
            // 处理邮件模板
            Context context = new Context();
            context.setVariable("code", code);
            String template = templateEngine.process("emailTemplate", context);
            helper.setText(template, true);
            jms.send(message);
            return "发送成功";
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }
}

```

其中`code`对应模板里的`${code}`变量。启动项目，访问[http://localhost/email/sendTemplateEmail?code=EOS9](http://localhost/email/sendTemplateEmail?code=EOS9)，页面提示发送成功：

![QQ截图20180510134244.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20180510134244.png)















