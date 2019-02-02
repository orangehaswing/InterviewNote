# Servlet / JSP / Web

## 1. 什么是Servlet

Servlet 是在服务器上运行的小程序。一个 servlet 就是一个 Java 类，并且可以通过 “请求—响应” 编程模式来访问的这个驻留在服务器内存里的 servlet 程序。

类的继承关系如下：

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/1535532891036.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/1535532891036.png)

Servlet三种实现方式：

- 实现javax.servlet.Servlet接口
- 继承javax.servlet.GenericServlet类
- 继承javax.servlet.http.HttpServlet类

　　通常会去继承HttpServlet类来完成Servlet。

## 2. Tomcat容器等级

Tomcat的容器分为4个等级，Servlet的容器管理Context容器，一个Context对应一个Web工程。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/4685968-b27b8782600dd0af.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/4685968-b27b8782600dd0af.png)

## 3. Servlet执行流程

主要描述了从浏览器到服务器，再从服务器到浏览器的整个执行过程

### 浏览器请求

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/20180521175251513.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/20180521175251513.png)

浏览器向服务器请求时，服务器不会直接执行我们的类，而是到 web.xml 里寻找路径名 ① 浏览器输入访问路径后，携带了请求行，头，体 ② 根据访问路径找到已注册的 servlet 名称 ③ 根据映射找到对应的 servlet 名 ④ 根据根据 servlet 名找到我们全限定类名，既我们自己写的类

### 服务器创建对象

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/20180521182037787.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/20180521182037787.png)

① 服务器找到全限定类名后，通过反射创建对象，同时也创建了 servletConfig，里面存放了一些初始化信息（注意服务器只会创建一次 servlet 对象，所以 servletConfig 也只有一个）

### 调用init方法

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/20180521183945631.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/20180521183945631.png)

① 对象创建好之后，首先要执行 init 方法，但是我们发现我们自定义类下没有 init 方法，所以程序会到其父类 HttpServlet 里找 ② 我们发现 HttpServlet 里也没有 init 方法，所以继续向上找，既向其父类 GenericServlet 中继续寻找,在 GenericServlet 中我们发现了 init 方法，则执行 init 方法（对接口 Servlet 中的 init 方法进行了重写）

注意： 在 GenericServlet 中执行 public void init(ServletConfig config) 方法的时候，又调用了自己无参无方法体的 init() 方法，其目的是为了方便开发者，如果开发者在初始化的过程中需要实现一些功能，可以重写此方法。

### 调用service方法

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/20180521212619975.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/20180521212619975.png)

接着，服务器会先创建两个对象：ServletRequest 请求对象和 ServletResponse 响应对象，用来封装浏览器的请求数据和封装向浏览器的响应数据 ① 接着服务器会默认在我们写的类里寻找 service(ServletRequest req, ServletResponse res) 方法，但是 DemoServlet 中不存在，那么会到其父类中寻找 ② 到父类 HttpServlet 中发现有此方法，则直接调用此方法，并将之前创建好的两个对象传入 ③ 然后将传入的两个参数强转，并调用 HttpServlet 下的另外个 service 方法 ④ 接着执行 `service(HttpServletRequest req, HttpServletResponse resp) `方法，在此方法内部进行了判断请求方式，并执行doGet和doPost，但是doGet和doPost方法已经被我们自己重写了，所以会执行我们重写的方法 看到这里，你或许有疑问：为什么我们不直接重写service方法？ 因为如果重写service方法的话，我们需要将强转，以及一系列的安全保护判断重新写一遍，会存在安全隐患

### 向浏览器响应

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/20180521214423142.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/20180521214423142.png)

## 4. Servlet生命周期

- `void init(ServletConfig servletConfig) `：Servlet对象创建之后马上执行的初始化方法，只执行一次；
- `void service(ServletRequest servletRequest, ServletResponse servletResponse) `：每次处理请求都是在调用这个方法，它会被调用多次；
- `void destroy() `：在Servlet被销毁之前调用，负责释放 Servlet 对象占用的资源的方法；

特性：

- 线程不安全的，所以它的效率高。
- 单例，一个类只有一个对象，当然可能存在多个 Servlet 类

Servlet 类由自己编写，但对象由服务器来创建，并由服务器来调用相应的方法　

服务器启动时 ( web.xml中配置`load-on-startup=1`，默认为0 ) 或者第一次请求该 servlet 时，就会初始化一个 Servlet 对象，也就是会执行初始化方法 init(ServletConfig conf)

该 servlet 对象去处理所有客户端请求，在 `service(ServletRequest req，ServletResponse res)` 方法中执行

最后服务器关闭时，才会销毁这个 servlet 对象，执行 destroy() 方法。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/1535535812505.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/1535535812505.png)

**总结（面试会问）：**　　　

1）Servlet何时创建

```
默认第一次访问servlet时创建该对象（调用init()方法）

```

2）Servlet何时销毁

```
服务器关闭servlet就销毁了(调用destroy()方法)

```

3）每次访问必须执行的方法

```
public void service(ServletRequest arg0, ServletResponse arg1)

```

## 5. Tomcat装载Servlet的三种情况

1. Servlet容器启动时自动装载某些Servlet，实现它只需要在web.xml文件中的 `` 之间添加以下代码：

```
<load-on-startup>1</load-on-startup>
```

　　其中，数字越小表示优先级越高。

　　例如：我们在 web.xml 中设置 TestServlet2 的优先级为 1，而 TestServlet1 的优先级为 2，启动和关闭Tomcat：优先级高的先启动也先关闭。　　

1. 客户端首次向某个Servlet发送请求
2. Servlet 类被修改后，Tomcat 容器会重新装载 Servlet。

## 6. forward和redirect

*本节参考：《Java程序员面试笔试宝典》P172*

　　在设计 Web 应用程序时，经常需要把一个系统进行结构化设计，即按照模块进行划分，让不同的 Servlet 来实现不同的功能，例如可以让其中一个 Servlet 接收用户的请求，另外一个 Servlet 来处理用户的请求。为了实现这种程序的模块化，就需要保证在不同的 Servlet 之间可以相互跳转，而 Servlet 中主要有两种实现跳转的方式：forward 与 redirect 方式。

　　forward 是服务器内部的重定向，服务器直接访问目标地址的 URL，把那个 URL 的响应内容读取过来，而客户端并不知道，因此在客户端浏览器的地址栏中不会显示转向后的地址，还是原来的地址。由于在整个定向的过程中用的是同一个 Request，因此 forward 会将 Request 的信息带到被定向的 JSP 或 Servlet 中使用。

　　redirect 则是客户端的重定向，是完全的跳转，即客户端浏览器会获取到跳转后的地址，然后重新发送请求，因此浏览器中会显示跳转后的地址。同事，由于这种方式比 forward 方式多了一次网络请求，因此其效率要低于 forward 方式。需要注意的是，客户端的重定向可以通过设置特定的 HTTP 头或改写 JavaScript 脚本实现。

　　下图可以更好的说明二者的区别：

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/1535537913258.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/1535537913258.png)

　　鉴于以上的区别，一般当 forward 方式可以满足需求时，尽可能地使用 forward 方式。但在有些情况下，例如，需要跳转到下一个其他服务器上的资源，则必须使用 redirect 方式。

引申：filter的作用是什么？主要实现什么方法？

filter 使用户可以改变一个 request 并且修改一个 response。filter 不是一个 Servlet，它不能产生一个 response，但它能够在一个 request 到达 Servlet 之前预处理 request，也可以在离开 Servlet 时处理 response。filter 其实是一个 “Servlet Chaining” (Servler 链)。

一个 filter 的作用包括以下几个方面：

1）在 Servlet 被调用之前截获

2）在 Servlet 被调用之前检查 Servlet Request

3）根据需要修改 Request 头和 Request 数据

4）根据需要修改 Response 头和 Response 数据

5）在 Servlet 被调用之后截获

## 7. Jsp和Servlet的区别

**1、不同之处在哪？**

- Servlet 在 Java 代码中通过 HttpServletResponse 对象动态输出 HTML 内容
- JSP 在静态 HTML 内容中嵌入 Java 代码，Java 代码被动态执行后生成 HTML 内容

**2、各自的特点**

- Servlet 能够很好地组织业务逻辑代码，但是在 Java 源文件中通过字符串拼接的方式生成动态 HTML 内容会导致代码维护困难、可读性差
- JSP 虽然规避了 Servlet 在生成 HTML 内容方面的劣势，但是在 HTML 中混入大量、复杂的业务逻辑同样也是不可取的

**3、通过MVC双剑合璧**

既然 JSP 和 Servlet 都有自身的适用环境，那么能否扬长避短，让它们发挥各自的优势呢？答案是肯定的——MVC(Model-View-Controller)模式非常适合解决这一问题。

MVC模式（Model-View-Controller）是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）：

- Controller——负责转发请求，对请求进行处理
- View——负责界面显示
- Model——业务功能编写（例如算法实现）、数据库设计以及数据存取操作实现

在 JSP/Servlet 开发的软件系统中，这三个部分的描述如下所示：

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/229cf9ff5b1729eaf408fac56238eeb3.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/229cf9ff5b1729eaf408fac56238eeb3.png)

1. Web 浏览器发送 HTTP 请求到服务端，被 Controller(Servlet) 获取并进行处理（例如参数解析、请求转发）
2. Controller(Servlet) 调用核心业务逻辑——Model部分，获得结果
3. Controller(Servlet) 将逻辑处理结果交给 View（JSP），动态输出 HTML 内容
4. 动态生成的 HTML 内容返回到浏览器显示

MVC 模式在 Web 开发中的好处是非常明显，它规避了 JSP 与 Servlet 各自的短板，Servlet 只负责业务逻辑而不会通过 out.append() 动态生成 HTML 代码；JSP 中也不会充斥着大量的业务代码。这大大提高了代码的可读性和可维护性。

## 8. tomcat和Servlet的联系

　　Tomcat是Web应用服务器，是一个Servlet/JSP容器。Tomcat 作为 Servlet 容器，负责处理客户请求，把请求传送给Servlet，并将Servlet的响应传送回给客户。而 Servlet 是一种运行在支持 Java 语言的服务器上的组件。Servlet最常见的用途是扩展 Java Web 服务器功能，提供非常安全的，可移植的，易于使用的CGI替代品。

　　从 http 协议中的请求和响应可以得知，浏览器发出的请求是一个请求文本，而浏览器接收到的也应该是一个响应文本。但是在上面这个图中，并不知道是如何转变的，只知道浏览器发送过来的请求也就是 request，我们响应回去的就用 response。忽略了其中的细节，现在就来探究一下。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/servlet-tomcat.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/servlet-tomcat.png)

① Tomcat 将 http 请求文本接收并解析，然后封装成 HttpServletRequest 类型的 request 对象，所有的 HTTP 头数据读可以通过 request 对象调用对应的方法查询到。

② Tomcat 同时会要响应的信息封装为 HttpServletResponse 类型的 response 对象，通过设置 response 属性就可以控制要输出到浏览器的内容，然后将 response 交给 tomcat，tomcat 就会将其变成响应文本的格式发送给浏览器

Java Servlet API 是 Servlet 容器(tomcat) 和 servlet 之间的接口，它定义了 serlvet 的各种方法，还定义了 Servlet 容器传送给 Servlet 的对象类，其中最重要的就是 ServletRequest 和 ServletResponse。所以说我们在编写 servlet 时，需要实现 Servlet 接口，按照其规范进行操作。

## 9. cookie和session的区别

类似这种面试题，实际上都属于“开放性”问题，你扯到哪里都可以。不过如果我是面试官的话，我还是希望对方能做到一点——不要混淆 session 和 session 实现。

本来 session 是一个抽象概念，开发者为了实现中断和继续等操作，将 user agent 和 server 之间一对一的交互，抽象为“会话”，进而衍生出“会话状态”，也就是 session 的概念。

而 cookie 是一个实际存在的东西，http 协议中定义在 header 中的字段。可以认为是 session 的一种后端无状态实现。

而我们今天常说的 “session”，是为了绕开 cookie 的各种限制，通常借助 cookie 本身和后端存储实现的，一种更高级的会话状态实现。

所以 cookie 和 session，你可以认为是同一层次的概念，也可以认为是不同层次的概念。具体到实现，session 因为 session id 的存在，通常要借助 cookie 实现，但这并非必要，只能说是通用性较好的一种实现方案。

**引申**

1. 由于 HTTP 协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是 Session。典型的场景比如购物车，当你点击下单按钮时，由于 HTTP 协议无状态，所以并不知道是哪个用户操作的，所以服务端要为特定的用户创建了特定的 Session，用用于标识这个用户，并且跟踪用户，这样才知道购物车里面有几本书。这个 Session 是保存在服务端的，有一个唯一标识。在服务端保存Session 的方法很多，内存、数据库、文件都有。集群的时候也要考虑 Session 的转移，在大型的网站，一般会有专门的 Session 服务器集群，用来保存用户会话，这个时候 Session 信息都是放在内存的，使用一些缓存服务比如 Memcached 之类的来放 Session。

2. 思考一下服务端如何识别特定的客户？

   这个时候 Cookie 就登场了。每次 HTTP 请求的时候，客户端都会发送相应的 Cookie 信息到服务端。实际上大多数的应用都是用 Cookie 来实现 Session 跟踪的，第一次创建 Session 的时候，服务端会在 HTTP 协议中告诉客户端，需要在 Cookie 里面记录一个Session ID，以后每次请求把这个会话 ID 发送到服务器，我就知道你是谁了。有人问，如果客户端的浏览器禁用了 Cookie 怎么办？一般这种情况下，会使用一种叫做URL重写的技术来进行会话跟踪，即每次 HTTP 交互，URL后面都会被附加上一个诸如 sid=xxxxx 这样的参数，服务端据此来识别用户。

3. Cookie 其实还可以用在一些方便用户的场景下，设想你某次登陆过一个网站，下次登录的时候不想再次输入账号了，怎么办？这个信息可以写到 Cookie 里面，访问网站的时候，网站页面的脚本可以读取这个信息，就自动帮你把用户名给填了，能够方便一下用户。这也是 Cookie 名称的由来，给用户的一点甜头。

所以，总结一下：

- Session 是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中；
- Cookie 是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现 Session 的一种方式。

## 10. JavaEE中的三层结构和MVC

做企业应用开发时，经常采用三层架构分层：表示层、业务层、持久层。表示层负责接收用户请求、转发请求、显示数据等；业务层负责组织业务逻辑；持久层负责持久化业务对象。

这三个分层，每一层都有不同的模式，就是架构模式。**表示层**最常用的架构模式就是MVC。

因此，MVC 是三层架构中表示层最常用的架构模式。

MVC 是**客户端**的一种设计模式，所以他天然就不考虑数据如何存储的问题。作为客户端，只需要解决用户界面、交互和业务逻辑就好了。在 MVC 模式中，View 负责的是用户界面，Controller 负责交互，Model 负责业务逻辑。至于数据如何存储和读取，当然是由 Model 调用服务端的接口来完成。

在三层架构中，并没有客户端/服务端的概念，所以表示层、业务层的任务其实和 MVC 没什么区别，而持久层在 MVC 里面是没有的。

各层次的关系：表现层的控制->服务层->数据持久化层。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/jee-3-ties.bmp)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/jee-3-ties.bmp)

参考资料：

- [JavaEE中的三层结构和MVC - cuiyi's blog（崔毅 crazycy） - BlogJava](http://www.blogjava.net/crazycy/archive/2006/07/03/56387.html)
- [三层构架和 MVC 不同吗？ - 知乎](https://www.zhihu.com/question/24291079)

## 11. RESTful 架构

### 什么是REST

可以总结为一句话：REST 是所有 Web 应用都应该遵守的架构设计指导原则。 Representational State Transfer，翻译是”表现层状态转化”。 面向资源是 REST 最明显的特征，对于同一个资源的一组不同的操作。资源是服务器上一个可命名的抽象概念，资源是以名词为核心来组织的，首先关注的是名词。REST要求，必须通过统一的接口来对资源执行各种操作。对于每个资源只能执行一组有限的操作。（7个HTTP方法：GET/POST/PUT/DELETE/PATCH/HEAD/OPTIONS）

### 什么是RESTful API

符合REST架构设计的API。

### RESTful 风格

以豆瓣网为例

1. 应该尽量将 API 部署在专用域名之下 `http://api.douban.com`/v2/user/1000001?apikey=XXX

2. 应该将 API 的版本号放入URL `http://api.douban.com/v2`/user/1000001?apikey=XXX

3. 在 RESTful 架构中，每个网址代表一种资源（resource），所以网址中`不能有动词，只能有名词`，而且所用的`名词往往与数据库的表格名对应`。一般来说，数据库中的表都是同种记录的”集合”（collection），所以 API 中的名词也应该使用复数。[http://api.douban.com/v2/`book`/:id](http://api.douban.com/v2/%60book%60/:id) (获取图书信息) [http://api.douban.com/v2/`movie`/subject/:id](http://api.douban.com/v2/%60movie%60/subject/:id) (电影条目信息)[http://api.douban.com/v2/`music`/:id](http://api.douban.com/v2/%60music%60/:id) (获取音乐信息) [http://api.douban.com/v2/`event`/:id](http://api.douban.com/v2/%60event%60/:id) (获取同城活动)

4. 对于资源的具体操作类型，由HTTP动词表示。常用的HTTP动词有下面四个(对应`增/删/改/查`)。 **GET**（`select`）：从服务器取出资源（一项或多项）。 eg. 获取图书信息 `GET` [http://api.douban.com/v2/book/:id](http://api.douban.com/v2/book/:id)\

   **POST**（`create`）：在服务器新建一个资源。 eg. 用户收藏某本图书 `POST` [http://api.douban.com/v2/book/:id/collection](http://api.douban.com/v2/book/:id/collection)

   **PUT**（`update`）：在服务器更新资源（客户端提供改变后的完整资源）。 eg. 用户修改对某本图书的收藏 `PUT`[http://api.douban.com/v2/book/:id/collection](http://api.douban.com/v2/book/:id/collection)

   **DELETE**（`delete`）：从服务器删除资源。 eg. 用户删除某篇笔记 `DELETE` [http://api.douban.com/v2/book/annotation/:id](http://api.douban.com/v2/book/annotation/:id)

5. 如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果

   ?limit=10：指定返回记录的数量 eg. 获取图书信息 `GET` [http://api.douban.com/v2/book/:id](http://api.douban.com/v2/book/:id)`?limit=10`

6. 服务器向用户返回的状态码和提示信息 每个状态码代表不同意思, 就像代号一样

   2系 代表正常返回

   4系 代表数据异常

   5系 代表服务器异常