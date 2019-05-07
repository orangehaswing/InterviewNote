# webserver 并发模型

# Rack 的协议与实现

作为 Rails 开发者，基本上每天都与 Rails 的各种 API 以及数据库打交道，Rails 的世界虽然非常简洁，不过其内部的实现还是很复杂的，很多刚刚接触 Rails 的开发者可能都不知道 Rails 其实就是一个 [Rack](https://github.com/rack/rack) 应用，在这一系列的文章中，我们会分别介绍 Rack 以及一些常见的遵循 Rack 协议的 webserver 的实现原理。

![rack-logo](https://img.draveness.me/2017-10-29-rack-logo.png)

不只是 Rails，几乎所有的 Ruby 的 Web 框架都是一个 Rack 的应用，除了 Web 框架之外，Rack 也支持相当多的 Web 服务器，可以说 Ruby 世界几乎一切与 Web 相关的服务都与 Rack 有关。

![rack-and-web-servers-frameworks](https://img.draveness.me/2017-10-29-rack-and-web-servers-frameworks.png)

所以如果想要了解 Rails 或者其他 Web 服务底层的实现，那么一定需要了解 Rack 是如何成为应用容器（webserver）和应用框架之间的桥梁的，本文中介绍的是 2.0.3 版本的 rack。

## Rack 协议

在 Rack 的协议中，将 Rack 应用描述成一个可以响应 `call` 方法的 Ruby 对象，它仅接受来自外界的一个参数，也就是环境，然后返回一个只包含三个值的数组，按照顺序分别是状态码、HTTP Headers 以及响应请求的正文。

> A Rack application is a Ruby object (not a class) that responds to call. It takes exactly one argument, the environment and returns an Array of exactly three values: The status, the headers, and the body.

![rack-protoco](https://img.draveness.me/2017-10-29-rack-protocol.png)

Rack 在 webserver 和应用框架之间提供了一套最小的 API 接口，如果 webserver 都遵循 Rack 提供的这套规则，那么所有的框架都能通过协议任意地改变底层使用 webserver；所有的 webserver 只需要在 `Rack::Handler` 的模块中创建一个实现了 `.run` 方法的类就可以了：

```
module Rack
  module Handler
    class WEBrick < ::WEBrick::HTTPServlet::AbstractServlet
      def self.run(app, options={})
        # ..
      end
    end
  end
end

```

这个类方法接受两个参数，分别是一个 Rack 应用对象和一个包含各种参数的 `options` 字典，其中可能包括自定义的 ip 地址和端口号以及各种配置，根据 Rack 协议，所有应用对象在接受到一个 `#call` 方法并且传入 `env`时，都会返回一个三元组：

![rack-app](https://img.draveness.me/2017-10-29-rack-app.png)

最后的 `body` 响应体其实是一个由多个响应内容组成的数组，Rack 使用的 webserver 会将 `body` 中几个部分的连接到一起最后拼接成一个 HTTP 响应后返回。

## Rack 的使用

我们在大致了解 Rack 协议之后，其实可以从一段非常简单的代码入手来了解 Rack 是如何启动 webserver 来处理来自用户的请求的，我们可以在任意目录下创建如下所示的 config.ru 文件：

```
# config.ru

run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }

```

> 因为 `Proc` 对象也能够响应 `#call` 方法，所以上述的 Proc 对象也可以看做是一个 Rack 应用。

接下来，我们在同一目录使用 `rackup` 命令在命令行中启动一个 webserver 进程：

```
$ rackup config.ru
[2017-10-26 22:59:26] INFO  WEBrick 1.3.1
[2017-10-26 22:59:26] INFO  ruby 2.3.3 (2016-11-21) [x86_64-darwin16]
[2017-10-26 22:59:26] INFO  WEBrick::HTTPServer#start: pid=83546 port=9292

```

从命令的输出我们可以看到，使用 rackup 运行了一个 WEBrick 的进程，监听了 9292 端口，如果我们使用 curl 来访问对应的请求，就可以得到在 config.ru 文件中出现的 `'get rack\'d'` 文本：

> 在这篇文章中，作者都会使用开源的工具 [httpie](https://github.com/jakubroztocil/httpie) 代替 curl 在命令行中发出 HTTP 请求，相比 curl 而言 httpie 能够提供与 HTTP 响应有关的更多信息。

```
$ http http://localhost:9292
HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: text/html
Date: Thu, 26 Oct 2017 15:07:47 GMT
Server: WEBrick/1.3.1 (Ruby/2.3.3/2016-11-21)
Transfer-Encoding: chunked

get rack'd

```

从上述请求返回的 HTTP 响应头中的信息，我们可以看到 WEBrick 确实按照 config.ru 文件中的代码对当前的 HTTP 请求进行了处理。

### 中间件

Rack 协议和中间件是 Rack 能达到今天地位不可或缺的两个功能或者说特性，Rack 协议规定了 webserver 和 Rack 应用之间应该如何通信，而 Rack 中间件能够在上层改变 HTTP 的响应或者请求，在不改变应用的基础上为 Rack 应用增加新的功能。

Rack 的中间件是一个实现了两个方法 `.initialize` 和 `#call` 的类，初始化方法会接受两个参数，分别是 `app` 和 `options` 字典，而 `#call` 方法接受一个参数也就是 HTTP 请求的环境参数 `env`，在这里我们创建了一个新的 Rack 中间件 `StatusLogger`：

```
class StatusLogger
  def initialize(app, options={})
    @app = app
  end

  def call(env)
    status, headers, body = @app.call(env)
    puts status
    [status, headers, body]
  end
end

```

在所有的 `#call` 方法中都**应该**调用 `app.call` 让应用对 HTTP 请求进行处理并在方法结束时将所有的参数按照顺序返回。

```
use StatusLogger
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }

```

如果需要使用某一个 Rack 中间件只需要在当前文件中使用 `use` 方法，在每次接收到来自用户的 HTTP 请求时都会打印出当前响应的状态码。

```
$ rackup
[2017-10-27 19:46:40] INFO  WEBrick 1.3.1
[2017-10-27 19:46:40] INFO  ruby 2.3.3 (2016-11-21) [x86_64-darwin16]
[2017-10-27 19:46:40] INFO  WEBrick::HTTPServer#start: pid=5274 port=9292
200
127.0.0.1 - - [27/Oct/2017:19:46:53 +0800] "GET / HTTP/1.1" 200 - 0.0004

```

除了直接通过 `use` 方法直接传入 `StatusLogger` 中间件之外，我们也可以在 `use` 中传入配置参数，所有的配置都会通过 `options` 最终初始化一个中间件的实例，比如，我们有以下的中间件 `BodyTransformer`：

```
class BodyTransformer
  def initialize(app, options={})
    @app = app
    @count = options[:count]
  end

  def call(env)
    status, headers, body = @app.call(env)
    body = body.map { |str| str[0...@count].upcase + str[@count..-1] }
    [status, headers, body]
  end
end

```

上述中间件会在每次调用时都将 Rack 应用返回的 `body` 中前 `count` 个字符变成大写的，我们可以在 config.ru 中添加一个新的中间件：

```
use StatusLogger
use BodyTransformer, count: 3
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }

```

当我们再次使用 http 命令请求相同的 URL 时，就会获得不同的结果，同时由于我们保留了 `StatusLogger`，所以在 console 中也会打印出当前响应的状态码：

```
# session 1
$ rackup
[2017-10-27 21:04:05] INFO  WEBrick 1.3.1
[2017-10-27 21:04:05] INFO  ruby 2.3.3 (2016-11-21) [x86_64-darwin16]
[2017-10-27 21:04:05] INFO  WEBrick::HTTPServer#start: pid=7524 port=9292
200
127.0.0.1 - - [27/Oct/2017:21:04:19 +0800] "GET / HTTP/1.1" 200 - 0.0005

# session 2
$ http http://localhost:9292
HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: text/html
Date: Fri, 27 Oct 2017 13:04:19 GMT
Server: WEBrick/1.3.1 (Ruby/2.3.3/2016-11-21)
Transfer-Encoding: chunked

GET rack'd

```

Rack 的中间件的使用其实非常简单，我们只需要定义符合要求的类，然后在合适的方法中返回合适的结果就可以了，在接下来的部分我们将介绍 Rack 以及中间件的实现原理。

## Rack 的实现原理

到这里，我们已经对 Rack 的使用有一些基本的了解了，包括如何使用 `rackup` 命令启动一个 webserver，也包括 Rack 的中间件如何使用，接下来我们就准备开始对 Rack 是如何实现上述功能进行分析了。

### rackup 命令

那么 `rackup` 到底是如何工作的呢，首先我们通过 `which` 命令来查找当前 `rackup` 的执行路径并打印出该文件的全部内容：

```
$ which rackup
/Users/draveness/.rvm/gems/ruby-2.3.3/bin/rackup

$ cat /Users/draveness/.rvm/gems/ruby-2.3.3/bin/rackup
#!/usr/bin/env ruby_executable_hooks
#
# This file was generated by RubyGems.
#
# The application 'rack' is installed as part of a gem, and
# this file is here to facilitate running it.
#

require 'rubygems'

version = ">= 0.a"

if ARGV.first
  str = ARGV.first
  str = str.dup.force_encoding("BINARY") if str.respond_to? :force_encoding
  if str =~ /\A_(.*)_\z/ and Gem::Version.correct?($1) then
    version = $1
    ARGV.shift
  end
end

load Gem.activate_bin_path('rack', 'rackup', version)

```

从上述文件中的注释中可以看到当前文件是由 RubyGems 自动生成的，在文件的最后由一个 `load` 方法加载了某一个文件中的代码，我们可以在 pry 中尝试运行一下这个命令。

首先，通过 `gem list` 命令得到当前机器中所有 rack 的版本，然后进入 pry 执行 `.activate_bin_path` 命令：

```
$ gem list "^rack$"

*** LOCAL GEMS ***

rack (2.0.3, 2.0.1, 1.6.8, 1.2.3)

$ pry
[1] pry(main)> Gem.activate_bin_path('rack', 'rackup', '2.0.3')
=> "/Users/draveness/.rvm/gems/ruby-2.3.3/gems/rack-2.0.3/bin/rackup"

$ cat /Users/draveness/.rvm/gems/ruby-2.3.3/gems/rack-2.0.3/bin/rackup
#!/usr/bin/env ruby

require "rack"
Rack::Server.start

```

> `rackup` 命令定义在 rack 工程的 bin/rackup 文件中，在通过 rubygems 安装后会生成另一个加载该文件的可执行文建。

在最后打印了该文件的内容，到这里我们就应该知道 `.activate_bin_path` 方法会查找对应 gem 当前生效的版本，并返回文件的路径；在这个可执行文件中，上述代码只是简单的 `require` 了一下 rack 方法，之后运行 `.start` 启动了一个 `Rack::Server`。

### Server 的启动

从这里开始，我们就已经从 rackup 命令的执行进入了 rack 的源代码，可以直接使用 pry 找到 `.start` 方法所在的文件，从方法中可以看到当前类方法初始化了一个新的实例后，在新的对象上执行了 `#start` 方法：

```
$ pry
[1] pry(main)> require 'rack'
=> true
[2] pry(main)> $ Rack::Server.start

From: lib/rack/server.rb @ line 147:
Owner: #<Class:Rack::Server>

def self.start(options = nil)
  new(options).start
end

```

### 初始化和配置

在 `Rack::Server` 启动的过程中初始化了一个新的对象，初始化的过程中其实也包含了整个服务器的配置过程：

```
From: lib/rack/server.rb @ line 185:
Owner: #<Class:Rack::Server>

def initialize(options = nil)
  @ignore_options = []

  if options
    @use_default_options = false
    @options = options
    @app = options[:app] if options[:app]
  else
    argv = defined?(SPEC_ARGV) ? SPEC_ARGV : ARGV
    @use_default_options = true
    @options = parse_options(argv)
  end
end

```

在这个 `Server` 对象的初始化器中，虽然可以通过 `options` 从外界传入参数，但是当前类中仍然存在这个 `#options` 和 `#default_options` 两个实例方法：

```
From: lib/rack/server.rb @ line 199:
Owner: Rack::Server

def options
  merged_options = @use_default_options ? default_options.merge(@options) : @options
  merged_options.reject { |k, v| @ignore_options.include?(k) }
end

From: lib/rack/server.rb @ line 204:
Owner: Rack::Server

def default_options
  environment  = ENV['RACK_ENV'] || 'development'
  default_host = environment == 'development' ? 'localhost' : '0.0.0.0'
  {
    :environment => environment,
    :pid         => nil,
    :Port        => 9292,
    :Host        => default_host,
    :AccessLog   => [],
    :config      => "config.ru"
  }
end

```

上述两个方法中处理了一些对象本身定义的一些参数，比如默认的端口号 9292 以及默认的 config 文件，config 文件也就是 `rackup` 命令接受的一个文件参数，文件中的内容就是用来配置一个 Rack 服务器的代码，在默认情况下为 config.ru，也就是如果文件名是 config.ru，我们不需要向 `rackup` 命令传任何参数，它会自动找当前目录的该文件：

```
$ rackup
[2017-10-27 09:00:34] INFO  WEBrick 1.3.1
[2017-10-27 09:00:34] INFO  ruby 2.3.3 (2016-11-21) [x86_64-darwin16]
[2017-10-27 09:00:34] INFO  WEBrick::HTTPServer#start: pid=96302 port=9292

```

访问相同的 URL 能得到完全一致的结果，在这里就不再次展示了，有兴趣的读者可以亲自尝试一下。

### 『包装』应用

当我们执行了 `.initialize` 方法初始化了一个新的实例之后，接下来就会进入 `#start` 实例方法尝试启动一个 webserver 处理 config.ru 中定义的应用了：

```
From: lib/rack/server.rb @ line 258:
Owner: Rack::Server

def start &blk
  # ...

  wrapped_app
  # ..

  server.run wrapped_app, options, &blk
end

```

我们已经从上述方法中删除了很多对于本文来说不重要的代码实现，所以上述方法中最重要的部分就是 `#wrapped_app` 方法，以及另一个 `#server` 方法，首先来看 `#wrapped_app` 方法的实现。

```
From: lib/rack/server.rb @ line 353:
Owner: Rack::Server

def wrapped_app
  @wrapped_app ||= build_app app
end

```

上述方法有两部分组成，分别是 `#app` 和 `#build_app` 两个实例方法，其中 `#app` 方法的调用栈比较复杂：

![server-app-call-stack](https://img.draveness.me/2017-10-29-server-app-call-stack.png)

整个方法在最终会执行 `Builder.new_from_string` 通过 Ruby 中元编程中经常使用的 `eval` 方法，将输入文件中的全部内容与两端字符串拼接起来，并直接执行这段代码：

```
From: lib/rack/builder.rb @ line 48:
Owner: Rack::Builder

def self.new_from_string(builder_script, file="(rackup)")
  eval "Rack::Builder.new {\n" + builder_script + "\n}.to_app",
    TOPLEVEL_BINDING, file, 0
end

```

在 `eval` 方法中执行代码的作用其实就是如下所示的：

```
Rack::Builder.new {
  use StatusLogger
  use BodyTransformer, count: 3
  run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }
}.to_app

```

我们先暂时不管这段代码是如何执行的，我们只需要知道上述代码存储了所有的中间件以及 Proc 对象，最后通过 `#to_app` 方法返回一个 Rack 应用。

在这之后会使用 `#build_app` 方法将所有的中间件都包括在 Rack 应用周围，因为所有的中间件也都是一个响应 `#call` 方法，返回三元组的对象，其实也就是一个遵循协议的 App，唯一的区别就是中间件中会调用初始化时传入的 Rack App：

```
From: lib/rack/server.rb @ line 343:
Owner: Rack::Server

def build_app(app)
  middleware[options[:environment]].reverse_each do |middleware|
    middleware = middleware.call(self) if middleware.respond_to?(:call)
    next unless middleware
    klass, *args = middleware
    app = klass.new(app, *args)
  end
  app
end

```

经过上述方法，我们在一个 Rack 应用周围一层一层包装上了所有的中间件，最后调用的中间件在整个调用栈中的最外层，当包装后的应用接受来自外界的请求时，会按照如下的方式进行调用：

![wrapped-app](https://img.draveness.me/2017-10-29-wrapped-app.png)

所有的请求都会先经过中间件，每一个中间件都会在 `#call` 方法内部调用另一个中间件或者应用，在接收到应用的返回之后会分别对响应进行处理最后由最先定义的中间件返回。

### 中间件的实现

在 Rack 中，中间件是由两部分的代码共同处理的，分别是 `Rack::Builder` 和 `Rack::Server` 两个类，前者包含所有的能够在 config.ru 文件中使用的 DSL 方法，当我们使用 `eval` 执行 config.ru 文件中的代码时，会先初始化一个 `Builder` 的实例，然后执行 `instance_eval` 运行代码块中的所有内容：

```
From: lib/rack/builder.rb @ line 53:
Owner: Rack::Builder

def initialize(default_app = nil, &block)
  @use, @map, @run, @warmup = [], nil, default_app, nil
  instance_eval(&block) if block_given?
end

```

在这时，config.ru 文件中的代码就会在当前实例的环境下执行，文件中的 `#use` 和 `#run` 方法在调用时就会执行 `Builder` 的实例方法，我们可以先看一下 `#use` 方法是如何实现的：

```
From: lib/rack/builder.rb @ line 81:
Owner: Rack::Builder

def use(middleware, *args, &block)
  @use << proc { |app| middleware.new(app, *args, &block) }
end

```

上述方法会将传入的参数组合成一个接受 `app` 作为入参的 `Proc` 对象，然后加入到 `@use` 数组中存储起来，在这里并没有发生任何其他的事情，另一个 `#run` 方法的实现其实就更简单了：

```
From: lib/rack/builder.rb @ line 103:
Owner: Rack::Builder

def run(app)
  @run = app
end

```

它只是将传入的 `app` 对象存储到持有的 `@run` 实例变量中，如果我们想要获取当前的 `Builder` 生成的应用，只需要通过 `#to_app` 方法：

```
From: lib/rack/builder.rb @ line 144:
Owner: Rack::Builder

def to_app
  fail "missing run or map statement" unless @run
  @use.reverse.inject(@run) { |a,e| e[a] }
end

```

上述方法将所有传入 `#use` 和 `#run` 命令的应用和中间件进行了组合，通过 `#inject` 方法达到了如下所示的效果：

```
# config.ru
use MiddleWare1
use MiddleWare2
run RackApp

# equals to
MiddleWare1.new(MiddleWare2.new(RackApp)))

```

`Builder` 类其实简单来看就做了这件事情，将一种非常难以阅读的代码，变成比较清晰可读的 DSL，最终返回了一个中间件（也可以说是应用）对象，虽然在 `Builder` 中也包含其他的 DSL 语法元素，但是在这里都没有介绍。

上一小节提到的 `#build_app` 方法其实也只是根据当前的环境选择合适的中间件继续包裹到这个链式的调用中：

```
From: lib/rack/server.rb @ line 343:
Owner: Rack::Server

def build_app(app)
  middleware[options[:environment]].reverse_each do |middleware|
    middleware = middleware.call(self) if middleware.respond_to?(:call)
    next unless middleware
    klass, *args = middleware
    app = klass.new(app, *args)
  end
  app
end

```

在这里的 `#middleware` 方法可以被子类覆写，如果不覆写该方法会根据环境的不同选择不同的中间件数组包裹当前的应用：

```
From: lib/rack/server.rb @ line 229:
Owner: #<Class:Rack::Server>

def default_middleware_by_environment
  m = Hash.new {|h,k| h[k] = []}
  m["deployment"] = [
    [Rack::ContentLength],
    [Rack::Chunked],
    logging_middleware,
    [Rack::TempfileReaper]
  ]
  m["development"] = [
    [Rack::ContentLength],
    [Rack::Chunked],
    logging_middleware,
    [Rack::ShowExceptions],
    [Rack::Lint],
    [Rack::TempfileReaper]
  ]
  m
end

```

`.default_middleware_by_environment` 中就包含了不同环境下应该使用的中间件，`#build_app` 会视情况选择中间件加载。

### webserver 的选择

在 `Server#start` 方法中，我们已经通过 `#wrapped_app` 方法将应用和中间件打包到了一起，然后分别执行 `#server`和 `Server#run` 方法选择并运行 webserver，先来看 webserver 是如何选择的：

```
From: lib/rack/server.rb @ line 300:
Owner: Rack::Server

def server
  @_server ||= Rack::Handler.get(options[:server])
  unless @_server
    @_server = Rack::Handler.default
  end
  @_server
end

```

如果我们在运行 `rackup` 命令时传入了 `server` 选项，例如 `rackup -s WEBrick`，就会直接使用传入的 webserver，否则就会使用默认的 Rack 处理器：

```
From: lib/rack/handler.rb @ line 46:
Owner: #<Class:Rack::Handler>

def self.default
  # Guess.
  if ENV.include?("PHP_FCGI_CHILDREN")
    Rack::Handler::FastCGI
  elsif ENV.include?(REQUEST_METHOD)
    Rack::Handler::CGI
  elsif ENV.include?("RACK_HANDLER")
    self.get(ENV["RACK_HANDLER"])
  else
    pick ['puma', 'thin', 'webrick']
  end
end

```

在这个方法中，调用 `.pick` 其实最终也会落到 `.get` 方法上，在 `.pick` 中我们通过遍历传入的数组**尝试**对其进行加载：

```
From: lib/rack/handler.rb @ line 34:
Owner: #<Class:Rack::Handler>

def self.pick(server_names)
  server_names = Array(server_names)
  server_names.each do |server_name|
    begin
      return get(server_name.to_s)
    rescue LoadError, NameError
    end
  end

  raise LoadError, "Couldn't find handler for: #{server_names.join(', ')}."
end

```

`.get` 方法是用于加载 webserver 对应处理器的方法，方法中会通过一定的命名规范从对应的文件目录下加载相应的常量：

```
From: lib/rack/handler.rb @ line 11:
Owner: #<Class:Rack::Handler>

def self.get(server)
  return unless server
  server = server.to_s

  unless @handlers.include? server
    load_error = try_require('rack/handler', server)
  end

  if klass = @handlers[server]
    klass.split("::").inject(Object) { |o, x| o.const_get(x) }
  else
    const_get(server, false)
  end

rescue NameError => name_error
  raise load_error || name_error
end

```

一部分常量是预先定义在 handler.rb 文件中的，另一部分是由各个 webserver 的开发者自己定义或者遵循一定的命名规范加载的：

```
register 'cgi', 'Rack::Handler::CGI'
register 'fastcgi', 'Rack::Handler::FastCGI'
register 'webrick', 'Rack::Handler::WEBrick'
register 'lsws', 'Rack::Handler::LSWS'
register 'scgi', 'Rack::Handler::SCGI'
register 'thin', 'Rack::Handler::Thin'

```

在默认的情况下，如果不在启动服务时指定服务器就会按照 puma、thin 和 webrick 的顺序依次尝试加载响应的处理器。

### webserver 的启动

当 Rack 已经使用中间件对应用进行包装并且选择了对应的 webserver 之后，我们就可以将处理好的应用作为参数传入 `WEBrick.run` 方法了：

```
module Rack
  module Handler
    class WEBrick < ::WEBrick::HTTPServlet::AbstractServlet
      def self.run(app, options={})
        environment  = ENV['RACK_ENV'] || 'development'
        default_host = environment == 'development' ? 'localhost' : nil

        options[:BindAddress] = options.delete(:Host) || default_host
        options[:Port] ||= 8080
        @server = ::WEBrick::HTTPServer.new(options)
        @server.mount "/", Rack::Handler::WEBrick, app
        yield @server  if block_given?
        @server.start
      end
    end
  end
end

```

所有遵循 Rack 协议的 webserver 都会实现上述 `.run` 方法接受 `app`、`options` 和一个 block 作为参数运行一个进程来处理所有的来自用户的 HTTP 请求，在这里就是每个 webserver 自己需要解决的了，它其实并不属于 Rack 负责的部门，但是 Rack 实现了一些常见 webserver 的 handler，比如 CGI、Thin 和 WEBrick 等等，这些 handler 的实现原理都不会包含在这篇文章中。

## Rails 和 Rack

在了解了 Rack 的实现之后，其实我们可以发现 Rails 应用就是一堆 Rack 中间件和一个 Rack 应用的集合，在任意的工程中我们执行 `rake middleware` 的命令都可以得到以下的输出：

```
$ rake middleware
use Rack::Sendfile
use ActionDispatch::Static
use ActionDispatch::Executor
use ActiveSupport::Cache::Strategy::LocalCache::Middleware
use Rack::Runtime
use ActionDispatch::RequestId
use ActionDispatch::RemoteIp
use Rails::Rack::Logger
use ActionDispatch::ShowExceptions
use ActionDispatch::DebugExceptions
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::Migration::CheckPending
use Rack::Head
use Rack::ConditionalGet
use Rack::ETag
run ApplicationName::Application.routes

```

在这里包含了很多使用 `use` 加载的 Rack 中间件，当然在最后也包含一个 Rack 应用，也就是 `ApplicationName::Application.routes`，这个对象其实是一个 `RouteSet` 实例，也就是说在 Rails 中所有的请求在经过中间件之后都会先有一个路由表来处理，路由会根据一定的规则将请求交给其他控制器处理：

![rails-application](https://img.draveness.me/2017-10-29-rails-application.png)

除此之外，`rake middleware` 命令的输出也告诉我们 Rack 其实为我们提供了很多非常方便的中间件比如 `Rack::Sendfile` 等可以减少我们在开发一个 webserver 时需要处理的事情。

## 总结

Rack 协议可以说占领了整个 Ruby 服务端的市场，无论是常见的服务器还是框架都遵循 Rack 协议进行了设计，而正因为 Rack 以及 Rack 协议的存在我们在使用 Rails 或者 Sinatra 开发 Web 应用时才可以对底层使用的 webserver 进行无缝的替换，在接下来的文章中会逐一介绍不同的 webserver 是如何对 HTTP 请求进行处理以及它们拥有怎样的 I/O 模型。

# WEBrick 的多线程模型

这篇文章会介绍在开发环境中最常用的应用容器 WEBrick 的实现原理，除了通过源代码分析之外，我们也会介绍它的 IO 模型以及一些特点。

在 GitHub 上，WEBrick 从 2003 年的六月份就开始开发了，有着十几年历史的 WEBrick 的实现非常简单，总共只有 4000 多行的代码：

```
$ loc_counter .
40 files processed
Total     6918 lines
Empty     990 lines
Comments  1927 lines
Code      4001 lines

```

## WEBrick 的实现

由于 WEBrick 是 Rack 中内置的处理器，所以与 Unicorn 和 Puma 这种第三方开发的 webserver 不同，WEBrick 的处理器是在 Rack 中实现的，而 WEBrick 的运行也都是从这个处理器的开始的。

```
module Rack
  module Handler
    class WEBrick < ::WEBrick::HTTPServlet::AbstractServlet
      def self.run(app, options={})
        environment  = ENV['RACK_ENV'] || 'development'
        default_host = environment == 'development' ? 'localhost' : nil

        options[:BindAddress] = options.delete(:Host) || default_host
        options[:Port] ||= 8080
        @server = ::WEBrick::HTTPServer.new(options)
        @server.mount "/", Rack::Handler::WEBrick, app
        yield @server  if block_given?
        @server.start
      end
    end
  end
end

```

我们在上一篇文章 [谈谈 Rack 协议与实现](https://draveness.me/rack) 中介绍 Rack 的实现原理时，最终调用了上述方法，从这里开始大部分的实现都与 WEBrick 有关了。

在这里，你可以看到方法会先处理传入的参数比如：地址、端口号等等，在这之后会使用 WEBrick 提供的 `HTTPServer` 来处理 HTTP 请求，调用 `mount` 在根路由上挂载应用和处理器 `Rack::Handler::WEBrick` 接受请求，最后执行 `#start` 方法启动服务器。

### 初始化服务器

`HTTPServer` 的初始化分为两个阶段，一部分是 `HTTPServer` 的初始化，另一部分调用父类的 `initialize` 方法，在自己构造器中，会配置当前服务器能够处理的 HTTP 版本并初始化新的 `MountTable` 实例：

```
From: lib/webrick/httpserver.rb @ line 46:
Owner: #<Class:WEBrick::HTTPServer>

def initialize(config={}, default=Config::HTTP)
  super(config, default)
  @http_version = HTTPVersion::convert(@config[:HTTPVersion])

  @mount_tab = MountTable.new
  if @config[:DocumentRoot]
    mount("/", HTTPServlet::FileHandler, @config[:DocumentRoot],
          @config[:DocumentRootOptions])
  end

  unless @config[:AccessLog]
    @config[:AccessLog] = [
      [ $stderr, AccessLog::COMMON_LOG_FORMAT ],
      [ $stderr, AccessLog::REFERER_LOG_FORMAT ]
    ]
  end

  @virtual_hosts = Array.new
end

```

在父类 `GenericServer` 中初始化了用于监听端口号的 Socket 连接：

```
From: lib/webrick/server.rb @ line 185:
Owner: #<Class:WEBrick::GenericServer>

def initialize(config={}, default=Config::General)
  @config = default.dup.update(config)
  @status = :Stop

  @listeners = []
  listen(@config[:BindAddress], @config[:Port])
  if @config[:Port] == 0
    @config[:Port] = @listeners[0].addr[1]
  end
end

```

每一个服务器都会在初始化的时候创建一系列的 `listener` 用于监听地址和端口号组成的元组，其内部调用了 `Utils` 模块中定义的方法：

```
From: lib/webrick/server.rb @ line 127:
Owner: #<Class:WEBrick::GenericServer>

def listen(address, port)
  @listeners += Utils::create_listeners(address, port)
end

From: lib/webrick/utils.rb @ line 61:
Owner: #<Class:WEBrick::Utils>

def create_listeners(address, port)
  sockets = Socket.tcp_server_sockets(address, port)
  sockets = sockets.map {|s|
    s.autoclose = false
    ts = TCPServer.for_fd(s.fileno)
    s.close
    ts
  }
  return sockets
end
module_function :create_listeners

```

在 `.create_listeners` 方法中调用了 `.tcp_server_sockets` 方法由于初始化一组 `Socket` 对象，最后得到一个数组的 `TCPServer` 实例。

### 挂载应用

在使用 `WEBrick` 启动服务的时候，第二步就是将处理器和 Rack 应用挂载到根路由下：

```
@server.mount "/", Rack::Handler::WEBrick, app

```

`#mount` 方法其实是一个比较简单的方法，因为我们在构造器中已经初始化了 `MountTable` 对象，所以这一步只是将传入的多个参数放到这个表中：

```
From: lib/webrick/httpserver.rb @ line 155:
Owner: WEBrick::HTTPServer

def mount(dir, servlet, *options)
  @mount_tab[dir] = [ servlet, options ]
end

```

`MountTable` 是一个包含从路由到 Rack 处理器一个 App 的映射表：

![mounttable-and-applications](https://img.draveness.me/2017-11-01-mounttable-and-applications.png)

当执行了 `MountTable` 的 `#compile` 方法时，上述的对象就会将表中的所有键按照加入的顺序逆序拼接成一个如下的正则表达式用来匹配传入的路由：

```
^(/|/admin|/user)(?=/|$)

```

上述正则表达式在使用时如果匹配到了指定的路由就会返回 `$&` 和 `$'` 两个部分，前者表示整个匹配的文本，后者表示匹配文本后面的字符串。

### 启动服务器

在 `Rack::Handler::WEBrick` 中的 `.run` 方法先初始化了服务器，将处理器和应用挂载到了根路由上，在最后执行 `#start` 方法启动服务器：

```
From: lib/webrick/server.rb @ line 152:
Owner: WEBrick::GenericServer

def start(&block)
  raise ServerError, "already started." if @status != :Stop

  @status = :Running
  begin
    while @status == :Running
      begin
        if svrs = IO.select([*@listeners], nil, nil, 2.0)
          svrs[0].each{ |svr|
            sock = accept_client(svr)
            start_thread(sock, &block)
          }
        end
      rescue Errno::EBADF, Errno::ENOTSOCK, IOError, StandardError => ex
      rescue Exception => ex
        raise
      end
    end
  ensure
    cleanup_listener
    @status = :Stop
  end
end

```

由于原方法的实现比较复杂不容易阅读，在这里对方法进行了简化，省略了向 logger 中输出内容、处理服务的关闭以及执行回调等功能。

我们可以理解为上述方法通过 `.select` 方法对一组 Socket 进行监听，当有消息需要处理时就依次执行 `#accept_client` 和 `#start_thread` 两个方法处理来自客户端的请求：

```
From: lib/webrick/server.rb @ line 254:
Owner: WEBrick::GenericServer

def accept_client(svr)
  sock = nil
  begin
    sock = svr.accept
    sock.sync = true
    Utils::set_non_blocking(sock)
  rescue Errno::ECONNRESET, Errno::ECONNABORTED,
         Errno::EPROTO, Errno::EINVAL
  rescue StandardError => ex
    msg = "#{ex.class}: #{ex.message}\n\t#{ex.backtrace[0]}"
    @logger.error msg
  end
  return sock
end

```

WEBrick 在 `#accept_client` 方法中执行了 `#accept` 方法来得到一个 TCP 客户端 Socket，同时会通过 `set_non_blocking` 将该 Socket 变成非阻塞的，最后在方法末尾返回创建的 Socket。

在 `#start_thread` 方法中会**开启一个新的线程**，并在新的线程中执行 `#run` 方法来处理请求：

```
From: lib/webrick/server.rb @ line 278:
Owner: WEBrick::GenericServer

def start_thread(sock, &block)
  Thread.start {
    begin
      Thread.current[:WEBrickSocket] = sock
      run(sock)
    rescue Errno::ENOTCONN, ServerError, Exception
    ensure
      Thread.current[:WEBrickSocket] = nil
      sock.close
    end
  }
end

```

### 处理请求

所有的请求都不会由 `GenericServer` 这个通用的服务器来处理，它只处理通用的逻辑，对于 HTTP 请求的处理都是在 `HTTPServer#run` 中完成的：

```
From: lib/webrick/httpserver.rb @ line 69:
Owner: WEBrick::HTTPServer

def run(sock)
  while true
    res = HTTPResponse.new(@config)
    req = HTTPRequest.new(@config)
    server = self
    begin
      timeout = @config[:RequestTimeout]
      while timeout > 0
        break if sock.to_io.wait_readable(0.5)
        break if @status != :Running
        timeout -= 0.5
      end
      raise HTTPStatus::EOFError if timeout <= 0 || @status != :Running
      raise HTTPStatus::EOFError if sock.eof?
      req.parse(sock)
      res.request_method = req.request_method
      res.request_uri = req.request_uri
      res.request_http_version = req.http_version
      self.service(req, res)
    rescue HTTPStatus::EOFError, HTTPStatus::RequestTimeout, HTTPStatus::Error => ex
      res.set_error(ex)
    rescue HTTPStatus::Status => ex
      res.status = ex.code
    rescue StandardError => ex
      res.set_error(ex, true)
    ensure
      res.send_response(sock) if req.request_line
    end
    break if @http_version < "1.1"
  end
end

```

对 HTTP 协议了解的读者应该能从上面的代码中看到很多与 HTTP 协议相关的东西，比如 HTTP 的版本号、方法、URL 等等，上述方法总共做了三件事情，等待监听的 Socket 变得可读，执行 `#parse` 方法解析 Socket 上的数据，通过 `#service` 方法完成处理请求的响应，首先是对 Socket 上的数据进行解析：

```
From: lib/webrick/httprequest.rb @ line 192:
Owner: WEBrick::HTTPRequest

def parse(socket=nil)
  @socket = socket
  begin
    @peeraddr = socket.respond_to?(:peeraddr) ? socket.peeraddr : []
    @addr = socket.respond_to?(:addr) ? socket.addr : []
  rescue Errno::ENOTCONN
    raise HTTPStatus::EOFError
  end

  read_request_line(socket)
  if @http_version.major > 0
    # ...
  end
  return if @request_method == "CONNECT"
  return if @unparsed_uri == "*"

  begin
    setup_forwarded_info
    @request_uri = parse_uri(@unparsed_uri)
    @path = HTTPUtils::unescape(@request_uri.path)
    @path = HTTPUtils::normalize_path(@path)
    @host = @request_uri.host
    @port = @request_uri.port
    @query_string = @request_uri.query
    @script_name = ""
    @path_info = @path.dup
  rescue
    raise HTTPStatus::BadRequest, "bad URI `#{@unparsed_uri}'."
  end

  if /close/io =~ self["connection"]
    # deal with keep alive
  end
end

```

由于 HTTP 协议本身就比较复杂，请求中包含的信息也非常多，所以在这里用于**解析** HTTP 请求的代码也很多，想要了解 WEBrick 是如何解析 HTTP 请求的可以看 httprequest.rb 文件中的代码，在处理了 HTTP 请求之后，就开始执行 `#service` 响应该 HTTP 请求了：

```
From: lib/webrick/httpserver.rb @ line 125:
Owner: WEBrick::HTTPServer

def service(req, res)
  servlet, options, script_name, path_info = search_servlet(req.path)
  raise HTTPStatus::NotFound, "`#{req.path}' not found." unless servlet
  req.script_name = script_name
  req.path_info = path_info
  si = servlet.get_instance(self, *options)
  si.service(req, res)
end

```

在这里我们会从上面提到的 `MountTable` 中找出在之前注册的处理器 handler 和 Rack 应用：

```
From: lib/webrick/httpserver.rb @ line 182:
Owner: WEBrick::HTTPServer

def search_servlet(path)
  script_name, path_info = @mount_tab.scan(path)
  servlet, options = @mount_tab[script_name]
  if servlet
    [ servlet, options, script_name, path_info ]
  end
end

```

得到了处理器 handler 之后，通过 `.get_instance` 方法创建一个新的实例，这个方法在大多数情况下等同于初始化方法 `.new`，随后调用了该处理器 `Rack::WEBrick::Handler` 的 `#service` 方法，该方法是在 rack 工程中定义的：

```
From: rack/lib/handler/webrick.rb @ line 57:
Owner: Rack::Handler::WEBrick

def service(req, res)
  res.rack = true
  env = req.meta_vars
  env.delete_if { |k, v| v.nil? }

  env.update(
    # ...
    RACK_URL_SCHEME   => ["yes", "on", "1"].include?(env[HTTPS]) ? "https" : "http",
    # ...
  )
  
  status, headers, body = @app.call(env)
  begin
    res.status = status.to_i
    headers.each { |k, vs|
      # ...
    }

    body.each { |part|
      res.body << part
    }
  ensure
    body.close  if body.respond_to? :close
  end
end

```

由于上述方法也涉及了非常多 HTTP 协议的实现细节所以很多过程都被省略了，在上述方法中，我们先构建了应用的输入 `env` 哈希变量，然后通过执行 `#call` 方法将控制权交给 Rack 应用，最后获得一个由 `status`、`headers` 和 `body` 组成的三元组；在接下来的代码中，分别对这三者进行处理，为这次请求『填充』一个完成的 HTTP 请求。

到这里，最后由 `WEBrick::HTTPServer#run` 方法中的 `ensure` block 来结束整个 HTTP 请求的处理：

```
From: lib/webrick/httpserver.rb @ line 69:
Owner: WEBrick::HTTPServer

def run(sock)
  while true
    res = HTTPResponse.new(@config)
    req = HTTPRequest.new(@config)
    server = self
    begin
      # ...
    ensure
      res.send_response(sock) if req.request_line
    end
    break if @http_version < "1.1"
  end
end

```

在 `#send_reponse` 方法中，分别执行了 `#send_header` 和 `#send_body` 方法向当前的 Socket 中发送 HTTP 响应中的数据：

```
From: lib/webrick/httpresponse @ line 205:
Owner: WEBrick::HTTPResponse

def send_response(socket)
  begin
    setup_header()
    send_header(socket)
    send_body(socket)
  rescue Errno::EPIPE, Errno::ECONNRESET, Errno::ENOTCONN => ex
    @logger.debug(ex)
    @keep_alive = false
  rescue Exception => ex
    @logger.error(ex)
    @keep_alive = false
  end
end

```

所有向 Socket 中写入数据的工作最终都会由 `#_write_data` 这个方法来处理，将数据全部写入 Socket 中：

```
From: lib/webrick/httpresponse @ line 464:
Owner: WEBrick::HTTPResponse

def _write_data(socket, data)
  socket << data
end

```

从解析 HTTP 请求、调用 Rack 应用、创建 Response 到最后向 Socket 中写回数据，WEBrick 处理 HTTP 请求的部分就结束了。

## I/O 模型

通过对 WEBrick 源代码的阅读，我们其实已经能够了解整个 webserver 的工作原理，当我们启动一个 WEBrick 服务时只会启动一个进程，该进程会在指定的 ip 和端口上使用 `.select` 监听来自用户的所有 HTTP 请求：

![webrick-io-mode](https://img.draveness.me/2017-11-01-webrick-io-model.png)

当 `.select` 接收到来自用户的请求时，会为每一个请求创建一个新的 `Thread` 并在新的线程中对 HTTP 请求进行处理。

由于 WEBrick 在运行时只会启动一个进程，并没有其他的守护进程，所以它不够健壮，不能在发生问题时重启持续对外界提供服务，再加上 WEBrick 确实历史比较久远，代码的风格也不是特别的优雅，还有普遍知道的内存泄漏以及 HTTP 解析的问题，所以在生产环境中很少被使用。

虽然 WEBrick 有一些性能问题，但是作为 Ruby 自带的默认 webserver，在开发阶段使用 WEBrick 提供服务还是没有什么问题的。

## 总结

WEBrick 是 Ruby 社区中老牌的 webserver，虽然至今也仍然被广泛了解和使用，但是在生产环境中开发者往往会使用更加稳定的 Unicorn 和 Puma 代替它，我们选择在这个系列的文章中介绍它很大原因就是 WEBrick 的源代码与实现足够简单，我们很快就能了解一个 webserver 到底具备那些功能，在接下来的文章中我们就可以分析更加复杂的 webserver、了解更复杂的 IO 模型与实现了。

# Thin 的事件驱动模型

在上一篇文章中我们已经介绍了 WEBrick 的实现，它的 handler 是写在 Rack 工程中的，而在这篇文章介绍的 webserver [thin](https://github.com/macournoyer/thin) 的 Rack 处理器也是写在 Rack 中的；与 WEBrick 相同，Thin 的实现也非常简单，官方对它的介绍是：

> A very fast & simple Ruby web server.

它将 [Mongrel](https://zedshaw.com/archive/ragel-state-charts/)、[EventMachine](https://github.com/eventmachine/eventmachine) 和 [Rack](http://rack.github.io/) 三者进行组合，在其中起到胶水的作用，所以在理解 Thin 的实现的过程中我们也需要分析 EventMachine 到底是如何工作的。

## Thin 的实现

在这一节中我们将从源代码来分析介绍 Thin 的实现原理，因为部分代码仍然是在 Rack 工程中实现的，所以我们要从 Rack 工程的代码开始理解 Thin 的实现。

### 从 Rack 开始

Thin 的处理器 `Rack::Handler::Thin` 与其他遵循 Rack 协议的 webserver 一样都实现了 `.run` 方法，接受 Rack 应用和 `options` 作为输入：

```
module Rack
  module Handler
    class Thin
      def self.run(app, options={})
        environment  = ENV['RACK_ENV'] || 'development'
        default_host = environment == 'development' ? 'localhost' : '0.0.0.0'

        host = options.delete(:Host) || default_host
        port = options.delete(:Port) || 8080
        args = [host, port, app, options]
        args.pop if ::Thin::VERSION::MAJOR < 1 && ::Thin::VERSION::MINOR < 8
        server = ::Thin::Server.new(*args)
        yield server if block_given?
        server.start
      end
    end
  end
end

```

上述方法仍然会从 `options` 中取出 ip 地址和端口号，然后初始化一个 `Thin::Server` 的实例后，执行 `#start` 方法在 8080 端口上监听来自用户的请求。

### 初始化服务

Thin 服务的初始化由以下的代码来处理，首先会处理在 `Rack::Handler::Thin.run` 中传入的几个参数 `host`、`port`、`app` 和 `options`，将 Rack 应用存储在临时变量中：

```
From: lib/thin/server.rb @ line 100:
Owner: Thin::Server

def initialize(*args, &block)
  host, port, options = DEFAULT_HOST, DEFAULT_PORT, {}

  args.each do |arg|
    case arg
    when 0.class, /^\d+$/ then port    = arg.to_i
    when String           then host    = arg
    when Hash             then options = arg
    else
      @app = arg if arg.respond_to?(:call)
    end
  end

  @backend = select_backend(host, port, options)
  @backend.server = self
  @backend.maximum_connections            = DEFAULT_MAXIMUM_CONNECTIONS
  @backend.maximum_persistent_connections = DEFAULT_MAXIMUM_PERSISTENT_CONNECTIONS
  @backend.timeout                        = options[:timeout] || DEFAULT_TIMEOUT

  @app = Rack::Builder.new(&block).to_app if block
end

```

在初始化服务的过程中，总共只做了三件事情，处理参数、选择并配置 `backend`，创建新的应用：

![thin-initialize-serve](https://img.draveness.me/2017-11-04-thin-initialize-server.png)

处理参数的过程自然不用多说，只是这里判断的方式并不是按照顺序处理的，而是按照参数的类型；在初始化器的最后，如果向初始化器传入了 block，那么就会使用 `Rack::Builder` 和 block 中的代码初始化一个新的 Rack 应用。

### 选择后端

在选择后端时 Thin 使用了 `#select_backend` 方法，这里使用 `case` 语句替代多个 `if`、`else`，也是一个我们可以使用的小技巧：

```
From: lib/thin/server.rb @ line 261:
Owner: Thin::Server

def select_backend(host, port, options)
  case
  when options.has_key?(:backend)
    raise ArgumentError, ":backend must be a class" unless options[:backend].is_a?(Class)
    options[:backend].new(host, port, options)
  when options.has_key?(:swiftiply)
    Backends::SwiftiplyClient.new(host, port, options)
  when host.include?('/')
    Backends::UnixServer.new(host)
  else
    Backends::TcpServer.new(host, port)
  end
end

```

在大多数时候，我们只会选择 `UnixServer` 和 `TcpServer` 两种后端中的一个，而后者又是两者中使用更为频繁的后端：

```
From: lib/thin/backends/tcp_server.rb @ line 8:
Owner: Thin::Backends::TcpServer

def initialize(host, port)
  @host = host
  @port = port
  super()
end

From: lib/thin/backends/base.rb @ line 47:
Owner: Thin::Backends::Base

def initialize
  @connections                    = {}
  @timeout                        = Server::DEFAULT_TIMEOUT
  @persistent_connection_count    = 0
  @maximum_connections            = Server::DEFAULT_MAXIMUM_CONNECTIONS
  @maximum_persistent_connections = Server::DEFAULT_MAXIMUM_PERSISTENT_CONNECTIONS
  @no_epoll                       = false
  @ssl                            = nil
  @threaded                       = nil
  @started_reactor                = false
end

```

初始化的过程中只是对属性设置默认值，比如 `host`、`port` 以及超时时间等等，并没有太多值得注意的代码。

### 启动服务

在启动服务时会直接调用 `TcpServer#start` 方法并在其中传入一个用于处理信号的 block：

```
From: lib/thin/server.rb @ line 152:
Owner: Thin::Server

def start
  raise ArgumentError, 'app required' unless @app
  
  log_info  "Thin web server (v#{VERSION::STRING} codename #{VERSION::CODENAME})"
  log_debug "Debugging ON"
  trace     "Tracing ON"
  
  log_info "Maximum connections set to #{@backend.maximum_connections}"
  log_info "Listening on #{@backend}, CTRL+C to stop"

  @backend.start { setup_signals if @setup_signals }
end

```

虽然这里的 `backend` 其实已经被选择成了 `TcpServer`，但是该子类并没有覆写 `#start` 方法，这里执行的方法其实是从父类继承的：

```
From: lib/thin/backends/base.rb @ line 60:
Owner: Thin::Backends::Base

def start
  @stopping = false
  starter   = proc do
    connect
    yield if block_given?
    @running = true
  end
  
  # Allow for early run up of eventmachine.
  if EventMachine.reactor_running?
    starter.call
  else
    @started_reactor = true
    EventMachine.run(&starter)
  end
end

```

上述方法在构建一个 `starter` block 之后，将该 block 传入 `EventMachine.run` 方法，随后执行的 `#connect` 会启动一个 `EventMachine` 的服务器用于处理用户的网络请求：

```
From: lib/thin/backends/tcp_server.rb @ line 15:
Owner: Thin::Backends::TcpServer

def connect
  @signature = EventMachine.start_server(@host, @port, Connection, &method(:initialize_connection))
  binary_name = EventMachine.get_sockname( @signature )
  port_name = Socket.unpack_sockaddr_in( binary_name )
  @port = port_name[0]
  @host = port_name[1]
  @signature
end

```

在 EventMachine 的文档中，`.start_server` 方法被描述成一个在指定的地址和端口上初始化 TCP 服务的方法，正如这里所展示的，它经常在 `.run` 方法的 block 中执行；该方法的参数 `Connection` 作为处理 TCP 请求的类，会实现不同的方法接受各种各样的回调，传入的 `initialize_connection` block 会在有请求需要处理时对 `Connection` 对象进行初始化：

> `Connection` 对象继承自 `EventMachine::Connection`，是 EventMachine 与外界的接口，在 EventMachine 中的大部分事件都会调用 `Connection` 的一个实例方法来传递数据和参数。

```
From: lib/thin/backends/base.rb @ line 145:
Owner: Thin::Backends::Base

def initialize_connection(connection)
  connection.backend                 = self
  connection.app                     = @server.app
  connection.comm_inactivity_timeout = @timeout
  connection.threaded                = @threaded
  connection.start_tls(@ssl_options) if @ssl

  if @persistent_connection_count < @maximum_persistent_connections
    connection.can_persist!
    @persistent_connection_count += 1
  end
  @connections[connection.__id__] = connection
end

```

### 处理请求的连接

`Connection` 类中有很多的方法 `#post_init`、`#receive_data` 方法等等都是由 EventMachine 在接收到请求时调用的，当 Thin 的服务接收到来自客户端的数据时就会调用 `#receive_data` 方法：

```
From: lib/thin/connection.rb @ line 36:
Owner: Thin::Connection

def receive_data(data)
  @idle = false
  trace data
  process if @request.parse(data)
rescue InvalidRequest => e
  log_error("Invalid request", e)
  post_process Response::BAD_REQUEST
end

```

在这里我们看到了与 WEBrick 在处理来自客户端的原始数据时使用的方法 `#parse`，它会解析客户端请求的原始数据并执行 `#process` 来处理 HTTP 请求：

```
From: lib/thin/connection.rb @ line 47:
Owner: Thin::Connection

def process
  if threaded?
    @request.threaded = true
    EventMachine.defer { post_process(pre_process) }
  else
    @request.threaded = false
    post_process(pre_process)
  end
end

```

如果当前的连接允许并行处理多个用户的请求，那么就会在 `EventMachine.defer` 的 block 中执行两个方法 `#pre_process` 和 `#post_process`：

```
From: lib/thin/connection.rb @ line 63:
Owner: Thin::Connection

def pre_process
  @request.remote_address = remote_address
  @request.async_callback = method(:post_process)

  response = AsyncResponse
  catch(:async) do
    response = @app.call(@request.env)
  end
  response
rescue Exception => e
  unexpected_error(e)
  can_persist? && @request.persistent? ? Response::PERSISTENT_ERROR : Response::ERROR
end

```

在 `#pre_process` 中没有做太多的事情，只是调用了 Rack 应用的 `#call` 方法，得到一个三元组 `response`，在这之后将这个数组传入 `#post_process` 方法：

```
From: lib/thin/connection.rb @ line 95:
Owner: Thin::Connection

def post_process(result)
  return unless result
  result = result.to_a
  return if result.first == AsyncResponse.first

  @response.status, @response.headers, @response.body = *result
  @response.each do |chunk|
    send_data chunk
  end
rescue Exception => e
  unexpected_error(e)
  close_connection
ensure
  if @response.body.respond_to?(:callback) && @response.body.respond_to?(:errback)
    @response.body.callback { terminate_request }
    @response.body.errback  { terminate_request }
  else
    terminate_request unless result && result.first == AsyncResponse.first
  end
end

```

`#post_response` 方法将传入的数组赋值给 `response` 的 `status`、`headers` 和 `body` 这三部分，在这之后通过 `#send_data` 方法将 HTTP 响应以块的形式写回 Socket；写回结束后可能会调用对应的 `callback` 并关闭持有的 `request` 和 `response` 两个实例变量。

> 上述方法中调用的 `#send_data` 继承自 `EventMachine::Connection` 类。

### 小结

到此为止，我们对于 Thin 是如何处理来自用户的 HTTP 请求的就比较清楚了，我们可以看到 Thin 本身并没有做一些类似解析 HTTP 数据包以及发送数据的问题，它使用了来自 Rack 和 EventMachine 两个开源框架中很多已有的代码逻辑，确实只做了一些胶水的事情。

对于 Rack 是如何工作的我们在前面的文章 [谈谈 Rack 协议与实现](https://draveness.me/rack) 中已经介绍过了；虽然我们看到了很多与 EventMachine 相关的代码，但是到这里我们仍然对 EventMachine 不是太了解。

## EventMachine 和 Reactor 模式

为了更好地理解 Thin 的工作原理，在这里我们会介绍一个 EventMachine 和 Reactor 模式。

EventMachine 其实是一个使用 Ruby 实现的事件驱动的并行框架，它使用 Reactor 模式提供了事件驱动的 IO 模型，如果你对 Node.js 有所了解的话，那么你一定对事件驱动这个词并不陌生，EventMachine 的出现主要是为了解决两个核心问题：

- 为生产环境提供更高的可伸缩性、更好的性能和稳定性；
- 为上层提供了一些能够减少高性能的网络编程复杂性的 API；

其实 EventMachine 的主要作用就是将所有同步的 IO 都变成异步的，调度都通过事件来进行，这样用于监听用户请求的进程不会被其他代码阻塞，能够同时为更多的客户端提供服务；在这一节中，我们需要了解一下在 Thin 中使用的 EventMachine 中几个常用方法的实现。

### 启动事件循环

EventMachine 其实就是一个事件循环（Event Loop），当我们想使用 EventMachine 来处理某些任务时就一定需要调用 `.run` 方法启动这个事件循环来接受外界触发的各种事件：

```
From: lib/eventmachine.rb @ line 149:
Owner: #<Class:EventMachine>

def self.run blk=nil, tail=nil, &block
  # ...
  begin
    @reactor_pid = Process.pid
    @reactor_running = true
    initialize_event_machine
    (b = blk || block) and add_timer(0, b)
    if @next_tick_queue && !@next_tick_queue.empty?
      add_timer(0) { signal_loopbreak }
    end
    @reactor_thread = Thread.current

    run_machine
  ensure
    until @tails.empty?
      @tails.pop.call
    end

    release_machine
    cleanup_machine
    @reactor_running = false
    @reactor_thread = nil
  end
end

```

在这里我们会使用 `.initialize_event_machine` 初始化当前的事件循环，其实也就是一个全局的 `Reactor` 的单例，最终会执行 `Reactor#initialize_for_run` 方法：

```
From: lib/em/pure_ruby.rb @ line 522:
Owner: EventMachine::Reactor

def initialize_for_run
  @running = false
  @stop_scheduled = false
  @selectables ||= {}; @selectables.clear
  @timers = SortedSet.new # []
  set_timer_quantum(0.1)
  @current_loop_time = Time.now
  @next_heartbeat = @current_loop_time + HeartbeatInterval
end

```

在启动事件循环的过程中，它还会将传入的 block 与一个 `interval` 为 0 的键组成键值对存到 `@timers` 字典中，所有加入的键值对都会在大约 `interval` 的时间过后执行一次 block。

随后执行的 `#run_machine` 在最后也会执行 `Reactor` 的 `#run` 方法，该方法中包含一个 loop 语句，也就是我们一直说的事件循环：

```
From: lib/em/pure_ruby.rb @ line 540:
Owner: EventMachine::Reactor

def run
  raise Error.new( "already running" ) if @running
  @running = true

  begin
    open_loopbreaker

    loop {
      @current_loop_time = Time.now

      break if @stop_scheduled
      run_timers
      break if @stop_scheduled
      crank_selectables
      break if @stop_scheduled
      run_heartbeats
    }
  ensure
    close_loopbreaker
    @selectables.each {|k, io| io.close}
    @selectables.clear

    @running = false
  end
end

```

在启动事件循环之间会在 `#open_loopbreaker` 中创建一个 `LoopbreakReader` 的实例绑定在 `127.0.0.1` 和随机的端口号组成的地址上，然后开始运行事件循环。

![reactor-eventloop](https://img.draveness.me/2017-11-04-reactor-eventloop.png)

在事件循环中，Reactor 总共需要执行三部分的任务，分别是执行定时器、处理 Socket 上的事件以及运行心跳方法。

无论是运行定时器还是执行心跳方法其实都非常简单，只要与当前时间进行比较，如果到了触发的时间就调用正确的方法或者回调，最后的 `#crank_selectables` 方法就是用于处理 Socket 上读写事件的方法了：

```
From: lib/em/pure_ruby.rb @ line 540:
Owner: EventMachine::Reactor

def crank_selectables
  readers = @selectables.values.select { |io| io.select_for_reading? }
  writers = @selectables.values.select { |io| io.select_for_writing? }

  s = select(readers, writers, nil, @timer_quantum)

  s and s[1] and s[1].each { |w| w.eventable_write }
  s and s[0] and s[0].each { |r| r.eventable_read }

  @selectables.delete_if {|k,io|
    if io.close_scheduled?
      io.close
      begin
        EventMachine::event_callback io.uuid, ConnectionUnbound, nil
      rescue ConnectionNotBound; end
      true
    end
  }
end

```

上述方法会在 Socket 变成可读或者可写时执行 `#eventable_write` 或 `#eventable_read` 执行事件的回调，我们暂时放下这两个方法，先来了解一下 EventMachine 是如何启动服务的。

### 启动服务

在启动服务的过程中，最重要的目的就是创建一个 Socket 并绑定在指定的 ip 和端口上，在实现这个目的的过程中，我们使用了以下的几个方法，首先是 `EventMachine.start_server`：

```
From: lib/eventmachine.rb @ line 516:
Owner: #<Class:EventMachine>

def self.start_server server, port=nil, handler=nil, *args, &block
  port = Integer(port)
  klass = klass_from_handler(Connection, handler, *args)

  s = if port
        start_tcp_server server, port
      else
        start_unix_server server
      end
  @acceptors[s] = [klass, args, block]
  s
end

```

该方法其实使我们在使用 EventMachine 时常见的接口，只要我们想要启动一个新的 TCP 或者 UNIX 服务器，就可以上述方法，在这里会根据端口号是否存在，选择执行 `.start_tcp_server` 或者 `.start_unix_server` 创建一个新的 Socket 并存储在 `@acceptors` 中：

```
From: lib/em/pure_ruby.rb @ line 184:
Owner: #<Class:EventMachine>

def self.start_tcp_server host, port
  (s = EvmaTCPServer.start_server host, port) or raise "no acceptor"
  s.uuid
end

```

`EventMachine.start_tcp_server` 在这里也只做了个『转发』方法的作用的，直接调用 `EvmaTCPServer.start_server` 创建一个新的 Socket 对象并绑定到传入的 `` 上：

```
From: lib/em/pure_ruby.rb @ line 1108:
Owner: #<Class:EventMachine::EvmaTCPServer>

def self.start_server host, port
  sd = Socket.new( Socket::AF_LOCAL, Socket::SOCK_STREAM, 0 )
  sd.setsockopt( Socket::SOL_SOCKET, Socket::SO_REUSEADDR, true )
  sd.bind( Socket.pack_sockaddr_in( port, host ))
  sd.listen( 50 ) # 5 is what you see in all the books. Ain't enough.
  EvmaTCPServer.new sd
end

```

方法的最后会创建一个新的 `EvmaTCPServer` 实例的过程中，我们需要通过 `#fcntl` 将 Socket 变成非阻塞式的：

```
From: lib/em/pure_ruby.rb @ line 687:
Owner: EventMachine::Selectable

def initialize io
  @io = io
  @uuid = UuidGenerator.generate
  @is_server = false
  @last_activity = Reactor.instance.current_loop_time

  m = @io.fcntl(Fcntl::F_GETFL, 0)
  @io.fcntl(Fcntl::F_SETFL, Fcntl::O_NONBLOCK | m)

  @close_scheduled = false
  @close_requested = false

  se = self; @io.instance_eval { @my_selectable = se }
  Reactor.instance.add_selectable @io
end

```

不只是 `EvmaTCPServer`，所有的 `Selectable` 子类在初始化的最后都会将新的 Socket 以 `uuid` 为键存储到 `Reactor`单例对象的 `@selectables` 字典中：

```
From: lib/em/pure_ruby.rb @ line 532:
Owner: EventMachine::Reactor

def add_selectable io
  @selectables[io.uuid] = io
end

```

在整个事件循环的大循环中，这里存入的所有 Socket 都会被 `#select` 方法监听，在响应的事件发生时交给合适的回调处理，作者在 [Redis 中的事件循环](https://draveness.me/redis-eventloop) 一文中也介绍过非常相似的处理过程。

![eventmachine-select](https://img.draveness.me/2017-11-04-eventmachine-select.png)

所有的 Socket 都会存储在一个 `@selectables` 的哈希中并由 `#select` 方法监听所有的读写事件，一旦相应的事件触发就会通过 `eventable_read` 或者 `eventable_write` 方法来响应该事件。

### 处理读写事件

所有的读写事件都是通过 `Selectable` 和它的子类来处理的，在 EventMachine 中，总共有以下的几种子类：

![selectable-and-subclasses](https://img.draveness.me/2017-11-04-selectable-and-subclasses.png)

所有处理服务端读写事件的都是 `Selectable` 的子类，也就是 `EvmaTCPServer` 和 `EvmaUNIXServer`，而所有处理客户端读写事件的都是 `StreamObject` 的子类 `EvmaTCPServer` 和 `EvmaUNIXClient`。

当我们初始化的绑定在 `` 上的 Socket 对象监听到了来自用户的 TCP 请求时，当前的 Socket 就会变得可读，事件循环中的 `#select` 方法就会调用 `EvmaTCPClient#eventable_read` 通知由一个请求需要处理：

```
From: lib/em/pure_ruby.rb @ line 1130:
Owner: EventMachine::EvmaTCPServer

def eventable_read
  begin
    10.times {
      descriptor, peername = io.accept_nonblock
      sd = EvmaTCPClient.new descriptor
      sd.is_server = true
      EventMachine::event_callback uuid, ConnectionAccepted, sd.uuid
    }
  rescue Errno::EWOULDBLOCK, Errno::EAGAIN
  end
end

```

在这里会尝试多次 `#accept_non_block` 当前的 Socket 并会创建一个 TCP 的客户端对象 `EvmaTCPClient`，同时通过 `.event_callback` 方法发送 `ConnectionAccepted` 消息。

`EventMachine::event_callback` 就像是一个用于处理所有事件的中心方法，所有的回调都要通过这个中继器进行调度，在实现上就是一个庞大的 `if`、`else` 语句，里面处理了 EventMachine 中可能出现的 10 种状态和操作：

![event-callback](https://img.draveness.me/2017-11-04-event-callback.png)

大多数事件在触发时，都会从 `@conns` 中取出相应的 `Connection` 对象，最后执行合适的方法来处理，而这里触发的 `ConnectionAccepted` 事件是通过以下的代码来处理的：

```
From: lib/eventmachine.rb @ line 1462:
Owner: #<Class:EventMachine>

def self.event_callback conn_binding, opcode, data
  if opcode == # ...
    # ...
  elsif opcode == ConnectionAccepted
    accep, args, blk = @acceptors[conn_binding]
    raise NoHandlerForAcceptedConnection unless accep
    c = accep.new data, *args
    @conns[data] = c
    blk and blk.call(c)
    c
  else
    # ...
  end
end

```

上述的 `accep` 变量就是我们在 Thin 调用 `.start_server` 时传入的 `Connection` 类，在这里我们初始化了一个新的实例，同时以 Socket 的 `uuid` 作为键存到 `@conns` 中。

在这之后 `#select` 方法就会监听更多 Socket 上的事件了，当这个 “accept” 后创建的 Socket 接收到数据时，就会触发下面的 `#eventable_read` 方法：

```
From: lib/em/pure_ruby.rb @ line 1130:
Owner: EventMachine::StreamObject

def eventable_read
  @last_activity = Reactor.instance.current_loop_time
  begin
    if io.respond_to?(:read_nonblock)
      10.times {
        data = io.read_nonblock(4096)
        EventMachine::event_callback uuid, ConnectionData, data
      }
    else
      data = io.sysread(4096)
      EventMachine::event_callback uuid, ConnectionData, data
    end
  rescue Errno::EAGAIN, Errno::EWOULDBLOCK, SSLConnectionWaitReadable
  rescue Errno::ECONNRESET, Errno::ECONNREFUSED, EOFError, Errno::EPIPE, OpenSSL::SSL::SSLError
    @close_scheduled = true
    EventMachine::event_callback uuid, ConnectionUnbound, nil
  end
end

```

方法会从 Socket 中读取数据并通过 `.event_callback` 发送 `ConnectionData` 事件：

```
From: lib/eventmachine.rb @ line 1462:
Owner: #<Class:EventMachine>

def self.event_callback conn_binding, opcode, data
  if opcode == # ...
    # ...
  elsif opcode == ConnectionData
    c = @conns[conn_binding] or raise ConnectionNotBound, "received data #{data} for unknown signature: #{conn_binding}"
    c.receive_data data
  else
    # ...
  end
end

```

从上述方法对 `ConnectionData` 事件的处理就可以看到通过传入 Socket 的 `uuid` 和数据，就可以找到上面初始化的 `Connection` 对象，`#receive_data` 方法就能够将数据传递到上层，让用户在自定义的 `Connection` 中实现自己的处理逻辑，这也就是 Thin 需要覆写 `#receive_data` 方法来接受数据的原因了。

当 Thin 以及 Rack 应用已经接收到了来自用户的请求、完成处理并返回之后经过一系列复杂的调用栈就会执行 `Connection#send_data` 方法：

```
From: lib/em/connection.rb @ line 324:
Owner: EventMachine::Connection

def send_data data
  data = data.to_s
  size = data.bytesize if data.respond_to?(:bytesize)
  size ||= data.size
  EventMachine::send_data @signature, data, size
end

From: lib/em/pure_ruby.rb @ line 172:
Owner: #<Class:EventMachine>

def self.send_data target, data, datalength
  selectable = Reactor.instance.get_selectable( target ) or raise "unknown send_data target"
  selectable.send_data data
end

From: lib/em/pure_ruby.rb @ line 851:
Owner: EventMachine::StreamObject

def send_data data
  unless @close_scheduled or @close_requested or !data or data.length <= 0
    @outbound_q << data.to_s
  end
end

```

经过一系列同名方法的调用，在调用栈末尾的 `StreamObject#send_data` 中，将所有需要写入的数据全部加入 `@outbound_q` 中，这其实就是一个待写入数据的队列。

当 Socket 变得可写之后，就会由 `#select` 方法触发 `#eventable_write` 将 `@outbound_q` 队列中的数据通过 `#write_nonblock` 或者 `syswrite` 写入 Socket，也就是将请求返回给客户端。

```
From: lib/em/pure_ruby.rb @ line 823:
Owner: EventMachine::StreamObject

def eventable_write
  @last_activity = Reactor.instance.current_loop_time
  while data = @outbound_q.shift do
    begin
      data = data.to_s
      w = if io.respond_to?(:write_nonblock)
            io.write_nonblock data
          else
            io.syswrite data
          end

      if w < data.length
        @outbound_q.unshift data[w..-1]
        break
      end
    rescue Errno::EAGAIN, SSLConnectionWaitReadable, SSLConnectionWaitWritable
      @outbound_q.unshift data
      break
    rescue EOFError, Errno::ECONNRESET, Errno::ECONNREFUSED, Errno::EPIPE, OpenSSL::SSL::SSLError
      @close_scheduled = true
      @outbound_q.clear
    end
  end
end

```

### 关闭 Socket

当数据写入时发生了 `EOFError` 或者其他错误时就会将 `close_scheduled` 标记为 `true`，在随后的事件循环中会关闭 Socket 并发送 `ConnectionUnbound` 事件：

```
From: lib/em/pure_ruby.rb @ line 540:
Owner: EventMachine::Reactor

def crank_selectables
  # ...

  @selectables.delete_if {|k,io|
    if io.close_scheduled?
      io.close
      begin
        EventMachine::event_callback io.uuid, ConnectionUnbound, nil
      rescue ConnectionNotBound; end
      true
    end
  }
end

```

`.event_callback` 在处理 `ConnectionUnbound` 事件时会在 `@conns` 中将结束的 `Connection` 剔除：

```
def self.event_callback conn_binding, opcode, data
  if opcode == ConnectionUnbound
    if c = @conns.delete( conn_binding )
      c.unbind
      io = c.instance_variable_get(:@io)
      begin
        io.close
      rescue Errno::EBADF, IOError
      end
    elsif c = @acceptors.delete( conn_binding )
    else
      raise ConnectionNotBound, "received ConnectionUnbound for an unknown signature: #{conn_binding}"
    end
  elsif opcode = 1
    #...
  end
end

```

在这之后会调用 `Connection` 的 `#unbind` 方法，再次执行 `#close` 确保 Socket 连接已经断掉了。

### 小结

EventMachine 在处理用户的请求时，会通过一个事件循环和一个中心化的事件处理中心 `.event_callback` 来响应所有的事件，你可以看到在使用 EventMachine 时所有的响应都是异步的，尤其是对 Socket 的读写，所有外部的输入在 EventMachine 看来都是一个事件，它们会被 EventMachine 选择合适的处理器进行转发。

## I/O 模型

Thin 本身其实没有实现任何的 I/O 模型，它通过对 EventMachine 进行封装，使用了其事件驱动的特点，为上层提供了处理并发 I/O 的 Reactor 模型，在不同的阶段有着不同的工作流程，在启动 Thin 的服务时，Thin 会直接通过 `.start_server` 创建一个 Socket 监听一个 `` 组成的元组：

![thin-start-server](https://img.draveness.me/2017-11-04-thin-start-server.png)

当服务启动之后，就可以接受来自客户端的 HTTP 请求了，处理 HTTP 请求总共需要三个模块的合作，分别是 EventMachine、Thin 以及 Rack 应用：

![thin-handle-request](https://img.draveness.me/2017-11-04-thin-handle-request.png)

在上图中省略了 Rack 的处理部分，不过对于其他部分的展示还是比较详细的，EventMachine 负责对 TCP Socket 进行监听，在发生事件时通过 `.event_callback` 进行处理，将消息转发给位于 Thin 中的 `Connection`，该类以及模块负责处理 HTTP 协议相关的内容，将整个请求包装成一个 `env` 对象，调用 `#call` 方法。

在这时就开始了返回响应的逻辑了，`#call` 方法会返回一个三元组，经过 Thin 中的 `#send_data` 最终将数据写入 `outbound_q` 队列中等待处理：

![thin-send-response](https://img.draveness.me/2017-11-04-thin-send-response.png)

EventMachine 会通过一个事件循环，使用 `#select` 监听当前 Socket 的可读写状态，并在合适的时候触发 `#eventable_write` 从 `outbound_q` 队列中读取数据写入 Socket，在写入结束后 Socket 就会被关闭，整个请求的响应也就结束了。

![thin-io-model](https://img.draveness.me/2017-11-04-thin-io-model.png)

Thin 使用了 EventMachine 作为底层处理 TCP 协议的框架，提供了事件驱动的 I/O 模型，也就是我们理解的 Reactor 模型，对于每一个 HTTP 请求都会创建一个对应的 `Connection` 对象，所有的事件都由 EventMachine 来派发，最大程度做到了 I/O 的读写都是异步的，不会阻塞当前的线程，这也是 Thin 以及 Node.js 能够并发处理大量请求的原因。

## 总结

Thin 作为一个 Ruby 社区中简单的 webserver，其实本身没有做太多的事情，只是使用了 EventMachine 提供的事件驱动的 I/O 模型，为上层提供了更加易用的 API，相比于其他同步处理请求的 webserver，Reactor 模式的优点就是 Thin 的优点，主程序只负责监听事件和分发事件，一旦涉及到 I/O 的工作都尽量使用回调的方式处理，当回调完成后再发送通知，这种方式能够减少进程的等待时间，时刻都在处理用户的请求和事件。

# Unicorn 的多进程模型

作为 Ruby 社区中老牌的 webserver，在今天也有很多开发者在生产环境使用 Unicorn 处理客户端的发出去的 HTTP 请求，与 WEBrick 和 Thin 不同，Unicorn 使用了完全不同的模型，提供了多进程模型批量处理来自客户端的请求。

![unicorn](https://img.draveness.me/2017-11-08-unicorn.jpeg)

Unicorn 为 Rails 应用提供并发的方式是使用 `fork` 创建多个 worker 线程，监听同一个 Socket 上的输入。

> 本文中使用的是 5.3.1 的 Unicorn，如果你使用了不同版本的 Unicorn，原理上的区别不会太大，只是在一些方法的实现上会有一些细微的不同。

## 实现原理

Unicorn 虽然也是一个遵循 Rack 协议的 Ruby webserver，但是因为它本身并没有提供 Rack 处理器，所以没有办法直接通过 `rackup -s Unicorn` 来启动 Unicorn 的进程。

```
$ unicorn -c unicorn.rb
I, [2017-11-06T08:05:03.082116 #33222]  INFO -- : listening on addr=0.0.0.0:8080 fd=10
I, [2017-11-06T08:05:03.082290 #33222]  INFO -- : worker=0 spawning...
I, [2017-11-06T08:05:03.083505 #33222]  INFO -- : worker=1 spawning...
I, [2017-11-06T08:05:03.083989 #33222]  INFO -- : master process ready
I, [2017-11-06T08:05:03.084610 #33223]  INFO -- : worker=0 spawned pid=33223
I, [2017-11-06T08:05:03.085100 #33223]  INFO -- : Refreshing Gem list
I, [2017-11-06T08:05:03.084902 #33224]  INFO -- : worker=1 spawned pid=33224
I, [2017-11-06T08:05:03.085457 #33224]  INFO -- : Refreshing Gem list
I, [2017-11-06T08:05:03.123611 #33224]  INFO -- : worker=1 ready
I, [2017-11-06T08:05:03.123670 #33223]  INFO -- : worker=0 ready

```

在使用 Unicorn 时，我们需要直接使用 `unicorn` 命令来启动一个 Unicorn 服务，在使用时可以通过 `-c` 传入一个配置文件，文件中的内容其实都是 Ruby 代码，每一个方法调用都是 Unicorn 的一条配置项：

```
$ cat unicorn.rb
worker_processes 2

```

### 可执行文件

`unicorn` 这个命令位于 `bin/unicorn` 中，在这个可执行文件中，大部分的代码都是对命令行参数的配置和说明，整个文件可以简化为以下的几行代码：

```
rackup_opts = # ...
app = Unicorn.builder(ARGV[0] || 'config.ru', op)
Unicorn::Launcher.daemonize!(options) if rackup_opts[:daemonize]
Unicorn::HttpServer.new(app, options).start.join

```

`unicorn` 命令会从 Rack 应用的标配 config.ru 文件或者传入的文件中加载代码构建一个新的 Rack 应用；初始化 Rack 应用后会使用 `.daemonize!` 方法将 unicorn 进程启动在后台运行；最后会创建并启动一个新的 `HttpServer` 的实例。

### 构建应用

读取 config.ru 文件并解析的过程其实就是直接使用了 Rack 的 `Builder` 模块，通过 `eval` 运行一段代码得到一个 Rack 应用：

```
From: lib/unicorn.rb @ line 39:
Owner: #<Class:Unicorn>

def self.builder(ru, op)
  raw = File.read(ru)
  inner_app = eval("Rack::Builder.new {(\n#{raw}\n)}.to_app", TOPLEVEL_BINDING, ru)

  middleware = {
    ContentLength: nil,
    Chunked: nil,
    CommonLogger: [ $stderr ],
    ShowExceptions: nil,
    Lint: nil,
    TempfileReaper: nil,
  }

  Rack::Builder.new do
    middleware.each do |m, args|
      use(Rack.const_get(m), *args) if Rack.const_defined?(m)
    end
    run inner_app
  end.to_app
end

```

在该方法中会执行两次 `Rack::Builder.new` 方法，第一次会运行 config.ru 中的代码，第二次会添加一些默认的中间件，最终会返回一个接受 `#call` 方法返回三元组的 Rack 应用。

### 守护进程

在默认情况下，Unicorn 的进程都是以前台进程的形式运行的，但是在生产环境我们往往需要在后台运行 Unicorn 进程，这也就是 `Unicorn::Launcher` 所做的工作。

```
From: lib/unicorn.rb @ line 23:
Owner: #<Class:Unicorn::Launcher>

def self.daemonize!(options)
  cfg = Unicorn::Configurator
  $stdin.reopen("/dev/null")

  unless ENV['UNICORN_FD']
    rd, wr = IO.pipe
    grandparent = $$
    if fork
      wr.close
    else
      rd.close
      Process.setsid
      exit if fork
    end

    if grandparent == $$
      master_pid = (rd.readpartial(16) rescue nil).to_i
      unless master_pid > 1
        warn "master failed to start, check stderr log for details"
        exit!(1)
      end
      exit 0
    else
      options[:ready_pipe] = wr
    end
  end
  cfg::DEFAULTS[:stderr_path] ||= "/dev/null"
  cfg::DEFAULTS[:stdout_path] ||= "/dev/null"
  cfg::RACKUP[:daemonized] = true
end

```

想要真正理解上述代码的工作，我们需要理解广义上的 daemonize 过程，在 Unix-like 的系统中，一个 [daemon](https://en.wikipedia.org/wiki/Daemon_(computing))（守护进程）是运行在后台不直接被用户操作的进程；一个进程想要变成守护进程通常需要做以下的事情：

1. 执行 `fork` 和 `exit` 来创建一个后台任务；
2. 从 tty 的控制中分离、创建一个新的 session 并成为新的 session 和进程组的管理者；
3. 将根目录 `/` 设置为当前进程的工作目录；
4. 将 umask 更新成 `0` 以提供自己的权限管理掩码；
5. 使用日志文件、控制台或者 `/dev/null` 设备作为标准输入、输出和错误；

在 `.daemonize!` 方法中我们总共使用 fork 创建了两个进程，整个过程涉及三个进程的协作，其中 grandparent 是启动 Unicorn 的进程一般指终端，parent 是用来启动 Unicorn 服务的进程，master 就是 Unicorn 服务中的主进程，三个进程有以下的关系：

![unicorn-daemonize](https://img.draveness.me/2017-11-08-unicorn-daemonize.png)

上述的三个进程中，grandparent 表示用于启动 Unicorn 进程的终端，parent 只是一个用于设置进程状态和掩码的中间进程，它在启动 Unicorn 的 master 进程后就会立刻退出。

在这里，我们会分三个部分分别介绍 grandparent、parent 和 master 究竟做了哪些事情；首先，对于 grandparent 进程来说，我们实际上运行了以下的代码：

```
def self.daemonize!(options)
  $stdin.reopen("/dev/null")
  rd, wr = IO.pipe
  wr.close

  # fork

  master_pid = (rd.readpartial(16) rescue nil).to_i
  unless master_pid > 1
    warn "master failed to start, check stderr log for details"
    exit!(1)
  end
end

```

通过 `IO.pipe` 方法创建了一对 Socket 节点，其中一个用于读，另一个用于写，在这里由于当前进程 grantparent 不需要写，所以直接将写的一端 `#close`，保留读的一端等待 Unicorn master 进程发送它的 `pid`，如果 master 没有成功启动就会报错，这也是 grandparent 进程的主要作用。

对于 parent 进程来说做的事情其实就更简单了，在 `fork` 之后会直接将读的一端执行 `#close`，这样无论是当前进程 parent 还是 parent fork 出来的进程都无法通过 `rd` 读取数据：

```
def self.daemonize!(options)
  $stdin.reopen("/dev/null")
  rd, wr = IO.pipe

  # fork

  rd.close
  Process.setsid
  exit if fork
end

```

在 parent 进程中，我们通过 `Process.setsid` 将当前的进程设置为新的 session 和进程组的管理者，从 tty 中分离；最后直接执行 `fork` 创建一个新的进程 master 并退出 parent 进程，parent 进程的作用其实就是为了启动新 Unicorn master 进程。

```
def self.daemonize!(options)
  cfg = Unicorn::Configurator
  $stdin.reopen("/dev/null")

  rd, wr = IO.pipe
  rd.close

  Process.setsid

  # fork

  options[:ready_pipe] = wr
  cfg::DEFAULTS[:stderr_path] ||= "/dev/null"
  cfg::DEFAULTS[:stdout_path] ||= "/dev/null"
  cfg::RACKUP[:daemonized] = true
end

```

新的进程 Unicorn master 就是一个不关联在任何 tty 的一个后台进程，不过到这里为止也仅仅创建另一个进程，Unicorn 还无法对外提供服务，我们将可读的 Socket `wr` 写入 `options` 中，在 webserver 成功启动后将通过 `IO.pipe` 创建的一对 Socket 将信息回传给 grandparent 进程通知服务启动的结果。

### 初始化服务

HTTP 服务在初始化时其实也没有做太多的事情，只是对 Rack 应用进行存储并初始化了一些实例变量：

```
From: lib/unicorn/http_server.rb @ line 69:
Owner: Unicorn::HttpServer

def initialize(app, options = {})
  @app = app
  @request = Unicorn::HttpRequest.new
  @reexec_pid = 0
  @ready_pipe = options.delete(:ready_pipe)
  @init_listeners = options[:listeners] ? options[:listeners].dup : []
  self.config = Unicorn::Configurator.new(options)
  self.listener_opts = {}

  @self_pipe = []
  @workers = {}
  @sig_queue = []
  @pid = nil

  config.commit!(self, :skip => [:listeners, :pid])
  @orig_app = app
  @queue_sigs = [:WINCH, :QUIT, :INT, :TERM, :USR1, :USR2, :HUP, :TTIN, :TTOU]
end

```

在 `.daemonize!` 方法中存储的 `ready_pipe` 在这时被当前的 `HttpServer` 对象持有，之后会通过这个管道上传数据。

### 启动服务

`HttpServer` 服务的启动一看就是这个 `#start` 实例方法控制的，在这个方法中总过做了两件比较重要的事情：

```
From: lib/unicorn/http_server.rb @ line 120:
Owner: Unicorn::HttpServer

def start
  @queue_sigs.each { |sig| trap(sig) { @sig_queue << sig; awaken_master } }
  trap(:CHLD) { awaken_master }

  self.pid = config[:pid]

  spawn_missing_workers
  self
end

```

第一件事情是将构造器中初始化的 `queue_sigs` 实例变量中的全部信号，通过 `trap` 为信号提供用于响应事件的代码。

第二件事情就是通过 `#spawn_missing_workers` 方法 `fork` 当前 master 进程创建一系列的 worker 进程：

```
From: lib/unicorn/http_server.rb @ line 531:
Owner: Unicorn::HttpServer

def spawn_missing_workers
  worker_nr = -1
  until (worker_nr += 1) == @worker_processes
    worker = Unicorn::Worker.new(worker_nr)
    before_fork.call(self, worker)

    pid = fork

    unless pid
      after_fork_internal
      worker_loop(worker)
      exit
    end

    @workers[pid] = worker
    worker.atfork_parent
  end
rescue => e
  @logger.error(e) rescue nil
  exit!
end

```

在这种调用了 `fork` 的方法中，我们还是将其一分为二来看，在这里就是 master 和 worker 进程，对于 master 进程来说：

```
From: lib/unicorn/http_server.rb @ line 531:
Owner: Unicorn::HttpServer

def spawn_missing_workers
  worker_nr = -1
  until (worker_nr += 1) == @worker_processes
    worker = Unicorn::Worker.new(worker_nr)
    before_fork.call(self, worker)

    pid = fork
    
    # ...

    @workers[pid] = worker
  end
rescue => e
  @logger.error(e) rescue nil
  exit!
end

```

通过一个 until 循环，master 进程能够创建 `worker_processes` 个 worker 进程，在每个循环中，上述方法都会创建一个 `Unicorn::Worker` 对象并在 `fork` 之后，将子进程的 `pid` 和 `worker` 以键值对的形式存到 `workers` 这个实例变量中。

`before_fork` 中存储的 block 是我们非常熟悉的，其实就是向服务器的日志中追加内容：

```
DEFAULTS = {
  # ...
  :after_fork => lambda { |server, worker|
    server.logger.info("worker=#{worker.nr} spawned pid=#{$$}")
  },
  :before_fork => lambda { |server, worker|
    server.logger.info("worker=#{worker.nr} spawning...")
  },
  :before_exec => lambda { |server|
    server.logger.info("forked child re-executing...")
  },
  :after_worker_exit => lambda { |server, worker, status|
    m = "reaped #{status.inspect} worker=#{worker.nr rescue 'unknown'}"
    if status.success?
      server.logger.info(m)
    else
      server.logger.error(m)
    end
  },
  :after_worker_ready => lambda { |server, worker|
    server.logger.info("worker=#{worker.nr} ready")
  },
  # ...
}

```

所有日志相关的输出大都在 `Unicorn::Configurator` 类中作为常量定义起来，并在初始化时作为缺省值赋值到 `HttpServer` 相应的实例变量上。而对于真正处理 HTTP 请求的 worker 进程来说，就会进入更加复杂的逻辑了：

```
def spawn_missing_workers
  worker_nr = -1
  until (worker_nr += 1) == @worker_processes
    worker = Unicorn::Worker.new(worker_nr)
    before_fork.call(self, worker)

    # fork

    after_fork_internal
    worker_loop(worker)
    exit
  end
rescue => e
  @logger.error(e) rescue nil
  exit!
end

```

在这里调用了两个实例方法，其中一个是 `#after_fork_internal`，另一个是 `#worker_loop` 方法，前者用于处理一些 `fork` 之后收尾的逻辑，比如关闭仅在 master 进程中使用的 `self_pipe`：

```
def after_fork_internal
  @self_pipe.each(&:close).clear
  @ready_pipe.close if @ready_pipe
  Unicorn::Configurator::RACKUP.clear
  @ready_pipe = @init_listeners = @before_exec = @before_fork = nil
end

```

而后者就是 worker 持续监听 Socket 输入并处理请求的循环了。

### 循环

当我们开始运行 worker 中的循环时，就开始监听 Socket 上的事件，整个过程还是比较直观的：

```
From: lib/unicorn/http_server.rb @ line 681:
Owner: Unicorn::HttpServer

def worker_loop(worker)
  ppid = @master_pid
  readers = init_worker_process(worker)
  ready = readers.dup
  @after_worker_ready.call(self, worker)

  begin
    tmp = ready.dup
    while sock = tmp.shift
      if client = sock.kgio_tryaccept
        process_client(client)
      end
    end

    unless nr == 0
      tmp = ready.dup
      redo
    end

    ppid == Process.ppid or return

    ret = IO.select(readers, nil, nil, @timeout) and ready = ret[0]
  rescue => e
    # ...
  end while readers[0]
end

```

如果当前 Socket 上有等待处理的 HTTP 请求，就会执行 `#process_client` 方法队请求进行处理，在这里调用了 Rack 应用的 `#call` 方法得到了三元组：

```
From: lib/unicorn/http_server.rb @ line 605:
Owner: Unicorn::HttpServer

def process_client(client)
  status, headers, body = @app.call(env = @request.read(client))

  begin
    return if @request.hijacked?

    @request.headers? or headers = nil
    http_response_write(client, status, headers, body,
                        @request.response_start_sent)
  ensure
    body.respond_to?(:close) and body.close
  end

  unless client.closed?
    client.shutdown
    client.close
  end
rescue => e
  handle_error(client, e)
end

```

请求的解析是通过 `Request#read` 处理的，而向 Socket 写 HTTP 响应是通过 `#http_response_write` 方法来完成的，在这里有关 HTTP 请求的解析和响应的处理都属于一些不重要的实现细节，在这里也就不展开介绍了；当我们已经响应了用户的请求就可以将当前 Socket 直接关掉，断掉这个 TCP 连接了。

## 调度

我们在上面已经通过多次 `fork` 启动了用于管理 Unicorn worker 进程的 master 以及多个 worker 进程，由于 Unicorn webserver 涉及了多个进程，所以需要进程之间进行调度。

在 Unix 中，进程的调度往往都是通过信号来进行的，`HttpServer#join` 就在 Unicorn 的 master 进程上监听外界发送的各种信号，不过在监听信号之前，要通过 `ready_pipe` 通知 grandparent 进程当前 master 进程已经启动完毕：

```
From: lib/unicorn/http_server.rb @ line 267:
Owner: Unicorn::HttpServer

def join
  respawn = true
  last_check = time_now

  proc_name 'master'
  logger.info "master process ready" # test_exec.rb relies on this message
  if @ready_pipe
    begin
      @ready_pipe.syswrite($$.to_s)
    rescue => e
      logger.warn("grandparent died too soon?: #{e.message} (#{e.class})")
    end
    @ready_pipe = @ready_pipe.close rescue nil
  end

  # ...
end

```

当 grandparent 进程，也就是执行 Unicorn 命令的进程接收到命令退出之后，就可以继续做其他的操作了，而 master 进程会进入一个 while 循环持续监听外界发送的信号：

```
def join
  # ...

  begin
    reap_all_workers
    case @sig_queue.shift
    # ...
    when :WINCH
      respawn = false
      soft_kill_each_worker(:QUIT)
      self.worker_processes = 0
    when :TTIN
      respawn = true
      self.worker_processes += 1
    when :TTOU
      self.worker_processes -= 1 if self.worker_processes > 0
    when # ...
    end
  rescue => e
    Unicorn.log_error(@logger, "master loop error", e)
  end while true

  # ...
end

```

这一部分的几个信号都会改变当前 Unicorn worker 的进程数，无论是 `TTIN`、`TTOU` 还是 `WINCH` 信号最终都修改了 `worker_processes` 变量，其中 `#soft_kill_each_worker` 方法向所有的 Unicorn worker 进程发送了 `QUIT` 信号。

除了一些用于改变当前 worker 数量的信号，Unicorn 的 master 进程还监听了一些用于终止 master 进程或者更新配置文件的信号。

```
def join
  # ...

  begin
    reap_all_workers
    case @sig_queue.shift
    # ...
    when :QUIT
      break
    when :TERM, :INT
      stop(false)
      break
    when :HUP
      respawn = true
      if config.config_file
        load_config!
      else # exec binary and exit if there's no config file
        reexec
      end
    when # ...
    end
  rescue => e
    Unicorn.log_error(@logger, "master loop error", e)
  end while true

  # ...
end

```

无论是 `QUIT` 信号还是 `TERM`、`INT` 最终都执行了 `#stop` 方法，选择使用不同的信号干掉当前 master 管理的 worker 进程：

```
From: lib/unicorn/http_server.rb @ line 339:
Owner: Unicorn::HttpServer

def stop(graceful = true)
  self.listeners = []
  limit = time_now + timeout
  until @workers.empty? || time_now > limit
    if graceful
      soft_kill_each_worker(:QUIT)
    else
      kill_each_worker(:TERM)
    end
    sleep(0.1)
    reap_all_workers
  end
  kill_each_worker(:KILL)
end

```

上述方法其实非常容易理解，它会根据传入的参数选择强制终止或者正常停止所有的 worker 进程，这样整个 Unicorn 服务才真正停止并不再为外界提供服务了。

当我们向 master 发送 `TTIN` 或者 `TTOU` 信号时只是改变了实例变量 `worker_process` 的值，还并没有 `fork` 出新的进程，这些操作都是在 `nil` 条件中完成的：

```
def join
  # ...

  begin
    reap_all_workers
    case @sig_queue.shift
    when nil
      if (last_check + @timeout) >= (last_check = time_now)
        sleep_time = murder_lazy_workers
      else
        sleep_time = @timeout/2.0 + 1
      end
      maintain_worker_count if respawn
      master_sleep(sleep_time)
    when # ...
    end
  rescue => e
    Unicorn.log_error(@logger, "master loop error", e)
  end while true

  # ...
end

```

当 `@sig_queue.shift` 返回 `nil` 时也就代表当前没有需要处理的信号，如果需要创建新的进程或者停掉进程就会通过 `#maintain_worker_count` 方法，之后 master 进程会陷入睡眠直到被再次唤醒。

```
From: lib/unicorn/http_server.rb @ line 561:
Owner: Unicorn::HttpServer

def maintain_worker_count
  (off = @workers.size - worker_processes) == 0 and return
  off < 0 and return spawn_missing_workers
  @workers.each_value { |w| w.nr >= worker_processes and w.soft_kill(:QUIT) }
end

```

通过创建缺失的进程并关闭多余的进程，我们能够实时的保证整个 Unicorn 服务中的进程数与期望的配置完全相同。

在 Unicorn 的服务中，不仅 master 进程能够接收到来自用户或者其他进程的各种信号，worker 进程也能通过以下的方式将接受到的信号交给 master 处理： 

```
From: lib/unicorn/http_server.rb @ line 120:
Owner: Unicorn::HttpServer

def start
  # ...
  @queue_sigs.each { |sig| trap(sig) { @sig_queue << sig; awaken_master } }
  trap(:CHLD) { awaken_master }
end

From: lib/unicorn/http_server.rb @ line 391:
Owner: Unicorn::HttpServer

def awaken_master
  return if $$ != @master_pid
  @self_pipe[1].kgio_trywrite('.')
end

```

所以即使向 worker 进程发送 `TTIN` 或者 `TTOU` 等信号也能够改变整个 Unicorn 服务中 worker 进程的个数。

## 多进程模型

总的来说，Unicorn 作为 Web 服务使用了多进程的模型，通过一个 master 进程来管理多个 worker 进程，其中 master 进程不负责处理客户端的 HTTP 请求，多个 worker 进程监听同一组 Socket：

![unicorn-io-mode](https://img.draveness.me/2017-11-08-unicorn-io-model.png)

一组 worker 进程在监听 Socket 时，如果发现当前的 Socket 有等待处理的请求时就会在当前的进程中直接通过 `#process_client` 方法处理，整个过程会阻塞当前的进程，而多进程阻塞 I/O 的方式没有办法接受慢客户端造成的性能损失，只能通过反向代理 nginx 才可以解决这个问题。

![unicorn-multi-processes](https://img.draveness.me/2017-11-08-unicorn-multi-processes.png)

在 Unicorn 中，worker 之间的负载均衡是由操作系统解决的，所有的 worker 是通过 `.select` 方法等待共享 Socket 上的请求，一旦出现可用的 worker，就可以立即进行处理，避开了其他负载均衡算法没有考虑到请求处理时间的问题。

## 总结

Unicorn 的源代码其实是作者读过的可读性最差的 Ruby 代码了，很多 Ruby 代码的风格写得跟 C 差不多，看起来也比较头疼，可能是需要处理很多边界条件以及信号，涉及较多底层的进程问题；虽然代码风格上看起来确实让人头疼，不过实现还是值得一看的，重要的代码大都包含在 unicorn.rb 和 http_server.rb 两个文件中，阅读时也不需要改变太多的上下文。

相比于 WEBrick 的单进程多线程的 I/O 模型，Unicorn 的多进程模型有很多优势，一是能够充分利用多核 CPU 的性能，其次能够通过 master 来管理并监控 Unicorn 中包含的一组 worker 并提供了零宕机部署的功能，除此之外，多进程的 I/O 模型还不在乎当前的应用是否是线程安全的，所以不会出现线程竞争等问题，不过 Unicorn 由于 `fork` 了大量的 worker 进程，如果长时间的在 Unicorn 上运行内存泄露的应用会非常耗费内存资源，可以考虑使用 [unicorn-worker-killer](https://github.com/kzk/unicorn-worker-killer) 来自动重启。



# Puma 的并发模型与实现

这篇文章已经是整个 Rack 系列文章的第五篇了，在前面的文章中我们见到了多线程模型、多进程模型以及事件驱动的 I/O 模型，对于几种常见的 webserver 已经很了解了，其实无论是 Ruby 还是其他社区对于 webserver 的实现也就是这么几种方式：多线程、多线程和 Reactor。

![puma-logo](https://img.draveness.me/2017-11-10-puma-logo.png)

在这篇文章中要介绍的 Puma 只是混合了两种 I/O 模型，同时使用多进程和多线程来提高应用的并行能力。

> 文中使用的 Puma 版本是 v3.10.0，如果你使用了不同版本的 Puma，原理上的区别不会太大，只是在一些方法的实现上会有一些细微的不同。

## Rack 默认处理器

Puma 是目前 Rack 中优先级最高的默认 webserver，如果直接使用 `rackup` 命令并且当前机器上安装了 `puma`，那么 Rack 会自动选择 Puma 作为当前处理 HTTP 请求的服务器：

```
def self.default
  pick ['puma', 'thin', 'webrick']
end

$ rackup
Puma starting in single mode...
* Version 3.10.0 (ruby 2.3.3-p222), codename: Russell's Teapot
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://localhost:9292
Use Ctrl-C to stop

```

通过在 `Rack::Handler` 下创建一个新的 `module Puma` 再实现类方法 `.run`，我们就可以直接将启动的过程转交给 `Puma::Launcher` 处理：

```
module Rack
  module Handler
    module Puma
      def self.run(app, options = {})
        conf   = self.config(app, options)
        events = options.delete(:Silent) ? ::Puma::Events.strings : ::Puma::Events.stdio
        launcher = ::Puma::Launcher.new(conf, :events => events)

        yield launcher if block_given?
        begin
          launcher.run
        rescue Interrupt
          puts "* Gracefully stopping, waiting for requests to finish"
          launcher.stop
          puts "* Goodbye!"
        end
      end
    end
  end
end

```

## 启动器 Launcher

Puma 中的启动器确实没有做太多的工作，大部分的代码其实都是在做配置，从 `ENV` 和上下文的环境中读取参数，而整个初始化方法中需要注意的地方也只有不同 `@runner` 的初始化了：

```
From: lib/puma/launcher.rb @ line 44:
Owner: Puma::Launcher

def initialize(conf, launcher_args={})
  @runner        = nil
  @events        = launcher_args[:events] || Events::DEFAULT
  @argv          = launcher_args[:argv] || []
  @config        = conf
  @config.load

  Dir.chdir(@restart_dir)

  if clustered?
    @events.formatter = Events::PidFormatter.new
    @options[:logger] = @events

    @runner = Cluster.new(self, @events)
  else
    @runner = Single.new(self, @events)
  end

  @status = :run
end

```

在 `#initialize` 方法中，`@runner` 的初始化是根据当前配置中的 worker 数决定的，如果当前的 `worker > 0`，那么就会选择 `Cluster` 作为 `@runner`，否则就会选择 `Single`，在初始化结束之后会执行 `Launcher#run` 方法启动当前的 Puma 进程：

```
From: lib/puma/launcher.rb @ line 165:
Owner: Puma::Launcher

def run
  previous_env = ENV.to_h

  setup_signals
  @runner.run

  case @status
  when :halt
    log "* Stopping immediately!"
  when :run, :stop
    graceful_stop
  when :restart
    log "* Restarting..."
    ENV.replace(previous_env)
    @runner.before_restart
    restart!
  when :exit
    # nothing
  end
end

```

在这个简单的 `#run` 方法中，Puma 通过 `#setup_singals` 设置了一些信号的响应过程，在这之后执行 `Runner#run`启动 Puma 的服务。

## 启动服务

根据配置文件中不同的配置项，Puma 在启动时有两种不同的选择，一种是当前的 worker 数为 0，这时会通过 `Single` 启动单机模式的 Puma 进程，另一种情况是 worker 数大于 0，它使用 `Cluster` 的 runner 启动一组 Puma 进程。

![single-cluster](https://img.draveness.me/2017-11-10-single-cluster.png)

在这一节中文章将会简单介绍不同的 runner 是如何启动 Puma 进程的。

### 单机模式

Puma 单机模式的启动通过 `Single` 类来处理，而定义这个类的文件 single.rb 中其实并没有多少代码，我们从中就可以看到单机模式下 Puma 的启动其实并不复杂：

```
From: lib/puma/single.rb @ line 40:
Owner: Puma::Single

def run
  output_header "single"

  if daemon?
    log "* Daemonizing..."
    Process.daemon(true)
    redirect_io
  end

  load_and_bind
  @launcher.write_state
  @server = server = start_server

  begin
    server.run.join
  rescue Interrupt
    # Swallow it
  end
end

```

如果我们启动了后台模式，就会通过 Puma 为 Process 模块扩展的方法 `.daemon` 在后台启动新的 Puma 进程，启动的过程其实和 Unicorn 中的差不多：

```
From: lib/puma/daemon_ext.rb @ line 12:
Owner: #<Class:Process>

def self.daemon(nochdir=false, noclose=false)
  exit if fork
  Process.setsid
  exit if fork

  Dir.chdir "/" unless nochdir

  if !noclose
    STDIN.reopen File.open("/dev/null", "r")
    null_out = File.open "/dev/null", "w"
    STDOUT.reopen null_out
    STDERR.reopen null_out
  end

  0
end

```

在 Puma 中通过两次 `fork` 同时将当前进程从终端中分离出来，最终就可以得到一个独立的 Puma 进程，你可以通过下面的图片简单理解这个过程：

![puma-daemonize](https://img.draveness.me/2017-11-10-puma-daemonize.png)

当我们在后台启动了一个 Puma 的 master 进程之后就可以开始启动 Puma 的服务器了，也就是 `Puma::Server`的实例：

```
From: lib/puma/runner.rb @ line 151:
Owner: Puma::Runner

def start_server
  min_t = @options[:min_threads]
  max_t = @options[:max_threads]

  server = Puma::Server.new app, @launcher.events, @options
  server.min_threads = min_t
  server.max_threads = max_t
  server.inherit_binder @launcher.binder

  if @options[:mode] == :tcp
    server.tcp_mode!
  end

  unless development?
    server.leak_stack_on_error = false
  end

  server
end

```

这里有很多不是特别重要的代码，需要注意的是 `Server` 初始化的过程以及最大、最小线程数的设置，这些信息都是通过命令行或者配置文件传入的，例如 `puma -t 8:32` 表示当前的最小线程数为 8、最大线程数为 32 个，Puma 会根据当前的流量自动调节同一个进程中的线程个数。

服务在启动时会创建一个线程池 `ThreadPool` 并传入一个用于处理请求的 block，这个方法的实现其实非常长，这里省略了很多代码；

```
From: lib/puma/server.rb @ line 255:
Owner: Puma::Server

def run(background=true)
  queue_requests = @queue_requests

  @thread_pool = ThreadPool.new(@min_threads,
                                @max_threads,
                                IOBuffer) do |client, buffer|
    process_now = false

    begin
      if queue_requests
        process_now = client.eagerly_finish
      else
        client.finish
        process_now = true
      end
    rescue MiniSSL::SSLError, HttpParserError => e
      # ...
    rescue ConnectionError
      client.close
    else
      if process_now
        process_client client, buffer
      else
        client.set_timeout @first_data_timeout
        @reactor.add client
      end
    end
  end

  if queue_requests
    @reactor = Reactor.new self, @thread_pool
    @reactor.run_in_thread
  end

  @thread = Thread.new { handle_servers }
  @thread
end

```

上述代码创建了一个新的 `Reactor` 对象并在一个新的线程中执行 `#handle_servers` 接受客户端的请求，文章会在后面介绍请求的处理。

### 集群模式

如果在启动 puma 进程时使用 `-w` 参数，例如下面的命令：

```
$ puma -w 3
[20904] Puma starting in cluster mode...
[20904] * Version 3.10.0 (ruby 2.3.3-p222), codename: Russell's Teapot
[20904] * Min threads: 0, max threads: 16
[20904] * Environment: development
[20904] * Process workers: 3
[20904] * Phased restart available
[20904] * Listening on tcp://0.0.0.0:9292
[20904] Use Ctrl-C to stop
[20904] - Worker 2 (pid: 20907) booted, phase: 0
[20904] - Worker 1 (pid: 20906) booted, phase: 0
[20904] - Worker 0 (pid: 20905) booted, phase: 0

$ ps aux | grep puma
draveness        20909   0.0  0.0  4296440    952 s001  S+   10:23AM   0:00.01 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn puma
draveness        20907   0.0  0.1  4358888  12128 s003  S+   10:23AM   0:00.07 puma: cluster worker 2: 20904 [Desktop]
draveness        20906   0.0  0.1  4358888  12148 s003  S+   10:23AM   0:00.07 puma: cluster worker 1: 20904 [Desktop]
draveness        20905   0.0  0.1  4358888  12196 s003  S+   10:23AM   0:00.07 puma: cluster worker 0: 20904 [Desktop]
draveness        20904   0.0  0.2  4346784  25632 s003  S+   10:23AM   0:00.67 puma 3.10.0 (tcp://0.0.0.0:9292) [Desktop]

```

上述命令就会启动一个 Puma 的 master 进程和三个 worker 进程，Puma 集群模式就是通过 `Puma::Cluster` 类来启动的，而启动集群的方法 `#run` 仍然是一个非常长的方法，在这里仍然省去了很多的代码：

```
From: lib/puma/cluster.rb @ line 386:
Owner: Puma::Cluster

def run
  @status = :run

  output_header "cluster"
  log "* Process workers: #{@options[:workers]}"

  read, @wakeup = Puma::Util.pipe

  Process.daemon(true)
  spawn_workers

  begin
    while @status == :run
      begin
        res = IO.select([read], nil, nil, WORKER_CHECK_INTERVAL)

        if res
          req = read.read_nonblock(1)
          result = read.gets
          pid = result.to_i

          if w = @workers.find { |x| x.pid == pid }
            case req
            when "b"
              w.boot!
            when "t"
              w.dead!
            when "p"
              w.ping!(result.sub(/^\d+/,'').chomp)
            end
          else
            log "! Out-of-sync worker list, no #{pid} worker"
          end
        end

      rescue Interrupt
        @status = :stop
      end
    end

    stop_workers unless @status == :halt
  ensure
    read.close
    @wakeup.close
  end
end

```

在使用 `#spawn_workers` 之后，当前 master 进程就开始通过 Socket 监听所有来自worker 的消息，例如当前的状态以及心跳检查等等。

`#spawn_workers` 方法会通过 fork 创建当前集群中缺少的 worker 数，在新的进程中执行 `#worker` 方法并将 worker 保存在 master 的 `@workers` 数组中：

```
From: lib/puma/cluster.rb @ line 116:
Owner: Puma::Cluster

def spawn_workers
  diff = @options[:workers] - @workers.size
  return if diff < 1

  master = Process.pid

  diff.times do
    idx = next_worker_index
    pid = fork { worker(idx, master) }

    debug "Spawned worker: #{pid}"
    @workers << Worker.new(idx, pid, @phase, @options)
  end
end

```

在 fork 出的新进程中，`#worker` 方法与单机模式中一样都创建了新的 `Server` 实例，调用 `#run` 和 `#join` 方法启动服务：

```
From: lib/puma/cluster.rb @ line 231:
Owner: Puma::Cluster

def worker(index, master)
  title  = "puma: cluster worker #{index}: #{master}"
  $0 = title

  server = start_server
  server.run.join
end

```

与 Unicorn 完全相同，Puma 使用了一个 master 进程来管理所有的 worker 进程：

![puma-cluster-mode](https://img.draveness.me/2017-11-10-puma-cluster-mode.png)

虽然 Puma 集群中的所有节点也都是由 master 管理的，但是所有的事件和信号会由各个接受信号的进程处理的，只有在特定事件发生时会通知主进程。

## 处理请求

在 Puma 中所有的请求都是通过 `Server` 和 `ThreadPool` 协作来响应的，我们在 `#handler_servers` 方法中通过 `IO.select` 监听一组套接字上的读写事件：

```
From: lib/puma/server.rb @ line 334:
Owner: Puma::Server

def handle_servers
  begin
    sockets = @binder.ios
    pool = @thread_pool

    while @status == :run
      begin
        ios = IO.select sockets
        ios.first.each do |sock|
          begin
            if io = sock.accept_nonblock
              client = Client.new io, @binder.env(sock)
              pool << client
              pool.wait_until_not_full
            end
          rescue Errno::ECONNABORTED
            io.close rescue nil
          end
        rescue Object => e
          @events.unknown_error self, e, "Listen loop"
        end
      end
    rescue Exception => e
      # ...
  end
end

```

当有读写事件发生时会非阻塞的接受 Socket，创建新的 `Client` 对象最后加入到线程池中交给线程池来处理接下来的请求。

```
From: lib/puma/thread_pool.rb @ line 140:
Owner: Puma::ThreadPool

def <<(work)
  @mutex.synchronize do
    if @shutdown
      raise "Unable to add work while shutting down"
    end

    @todo << work

    if @waiting < @todo.size and @spawned < @max
      spawn_thread
    end

    @not_empty.signal
  end
end

```

`ThreadPool` 覆写了 `#<<` 方法，在这个方法中它将 `Client` 对象加入到 `@todo` 数组中，通过对比几个参数选择是否创建一个新的线程来处理当前队列中的任务。

重新回到 `ThreadPool` 的初始化方法 `#initialize` 中，线程池在初始化时就会创建最低数量的线程保证当前的 worker 进程中有足够的工作线程能够处理客户端的请求：

```
From: lib/puma/thread_pool.rb @ line 21:
Owner: Puma::ThreadPool

def initialize(min, max, *extra, &block)
  @mutex = Mutex.new

  @todo = []

  @spawned = 0
  @waiting = 0

  @min = Integer(min)
  @max = Integer(max)
  @block = block
  @extra = extra

  @workers = []

  @mutex.synchronize do
    @min.times { spawn_thread }
  end
end

```

每一个线程都是通过 `Thread.new` 创建的，我们会在这个线程执行的过程中执行传入的 block：

```
From: lib/puma/thread_pool.rb @ line 21:
Owner: Puma::ThreadPool

def spawn_thread
  @spawned += 1

  th = Thread.new(@spawned) do |spawned|
    todo  = @todo
    block = @block
    mutex = @mutex

    extra = @extra.map { |i| i.new }

    while true
      work = nil

      continue = true

      mutex.synchronize do
        work = todo.shift
      end

      begin
        block.call(work, *extra)
      rescue Exception => e
        STDERR.puts "Error reached top of thread-pool: #{e.message} (#{e.class})"
      end
    end

    mutex.synchronize do
      @spawned -= 1
      @workers.delete th
    end
  end

  @workers << th
  th
end

```

在每一个工作完成之后，也会在一个互斥锁内部使用 `#delete` 方法将当前线程从数组中删除，在这里执行的 block 中将客户端对象 `Client` 加入了 `Reactor` 中等待之后的处理。

```
@thread_pool = ThreadPool.new(@min_threads,
                              @max_threads,
                              IOBuffer) do |client, buffer|
  begin
    client.finish
  rescue MiniSSL::SSLError => e
    # ...
  else
    process_client client, buffer
  end
end

```

如过当前任务不需要立即处理，就会向 `Reactor` 加入任务等待一段时间，否则就会立即由 `#process_client` 方法进行处理，其中调用了 `#handle_request` 方法尝试处理当前的网络请求：

```
From: lib/puma/server.rb @ line 439:
Owner: Puma::Server

def process_client(client, buffer)
  begin
    while true
      case handle_request(client, buffer)
      when false
        return
      when true
        return unless @queue_requests
        buffer.reset
        unless client.reset(@status == :run)
          client.set_timeout @persistent_timeout
          @reactor.add client
          return
        end
      end
    end
  rescue StandardError => e
    # ...
  ensure
    # ...
  end
end

```

用于处理网络请求的方法 `#handle_request` 足足有 200 多行，代码中处理非常多的实现细节，在这里实在是不想一行一行代码看过去，也就简单梳理一下这段代码的脉络了：

```
From: lib/puma/server.rb @ line 574:
Owner: Puma::Server

def handle_request(req, lines)
  env = req.env
  client = req.io

  # ...

  begin
    status, headers, res_body = @app.call(env)

    headers.each do |k, vs|
      # ...
    end

    fast_write client, lines.to_s
    res_body.each do |part|
      fast_write client, part
      client.flush
    end
  ensure
    body.close
  end
end

```

我们在这里直接将这段代码压缩至 20 行左右，你可以看到与其他的 webserver 完全相同，这里也调用了 Rack 应用的 `#call` 方法获得了一个三元组，然后通过 `#fast_write` 将请求写回客户端的 Socket 结束这个 HTTP 请求。

## 并发模型

到目前为止，我们已经对 Puma 是如何处理 HTTP 请求的有一个比较清晰的认识了，对于每一个 HTTP 请求都会由操作系统选择不同的进程来处理，这部分的负载均衡完全是由 OS 层来做的，当请求被分配给某一个进程时，当前进程会根据持有的线程数选择是否对请求进行处理，在这时可能会创建新的 `Thread` 对象来处理这个请求，也可能会把当前请求暂时扔到 `Reactor` 中进行等待。

![puma-concurrency-model](https://img.draveness.me/2017-11-10-puma-concurrency-model.png)

`Reactor` 主要是为了提高 Puma 服务的性能存在的产物，它能够让当前的 worker 接受所有请求并将它们以队列的形式传入处理器中；如果当前的系统中存在慢客户端，那么也会占用处理请求的资源，不过由于 Puma 是多进程多线程模型的，所以影响没有那么严重，但是我们也经常会通过反向代理来解决慢客户端的问题。

## 总结

相比于多进程单线程的 Unicorn，Puma 提供了更灵活的配置功能，每一个进程的线程数都能在一定范围内进行收缩，目前也是绝大多数的 Ruby 项目使用的 webserver，从不同 webserver 的发展我们其实可以看出混合方式的并发模型虽然实现更加复杂，但是确实能够提供更高的性能和容错。

Puma 项目使用了 Rubocop 来规范项目中的代码风格，相比其他的 webserver 来说确实有更好的阅读体验，只是偶尔出现的长方法会让代码在理解时出现一些问题。

# Ruby Web 服务器的并发模型与性能

这是整个 Rack 系列文章的最后一篇了，在之前其实也尝试写过很多系列文章，但是到最后都因为各种原因放弃了，最近由于自己对 Ruby 的 webserver 非常感兴趣，所以看了下社区中常见 webserver 的实现原理，包括 WEBrick、Thin、Unicorn 和 Puma，虽然在 Ruby 社区中也有一些其他的 webserver 有着比较优异的性能，但是在这有限的文章中也没有办法全都介绍一遍。

![webservers](https://img.draveness.me/2017-11-17-webservers.png)

在这篇文章中，作者想对 Ruby 社区中不同 webserver 的实现原理和并发模型进行简单的介绍，总结一下前面几篇文章中的内容。

> 文中所有的压力测试都是在内存 16GB、8 CPU、2.6 GHz Intel Core i7 的 macOS 上运行的，如果你想要复现这里的测试可能不会得到完全相同的结果。

## WEBrick

WEBrick 是 Ruby 社区中非常古老的 Web 服务器，从 2000 年到现在已经有了将近 20 年的历史了，虽然 WEBrick 有着非常多的问题，但是迄今为止 WEBrick 也是开发环境中最常用的 Ruby 服务器；它使用了最为简单、直接的并发模型，运行一个 WEBrick 服务器只会在后台启动一个进程，默认监听来自 9292 端口的请求。

![webrick-concurrency-model](https://img.draveness.me/2017-11-17-webrick-concurrency-model.png)

当 WEBrick 通过 `.select` 方法监听到来自客户端的请求之后，会为每一个请求创建一个单独 `Thread` 并在新的线程中处理 HTTP 请求。

```
run Proc.new { |env| ['200', {'Content-Type' => 'text/plain'}, ['get rack\'d']] }

```

如果我们如果创建一个最简单的 Rack 应用，直接返回所有的 HTTP 响应，那么使用下面的命令对 WEBrick 的服务器进行测试会得到如下的结果：

```
Concurrency Level:      100
Time taken for tests:   22.519 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      2160000 bytes
HTML transferred:       200000 bytes
Requests per second:    444.07 [#/sec] (mean)
Time per request:       225.189 [ms] (mean)
Time per request:       2.252 [ms] (mean, across all concurrent requests)
Transfer rate:          93.67 [Kbytes/sec] received

```

在处理 ApacheBench 发出的 10000 个 HTTP 请求时，WEBrick 对于每个请求平均消耗了 225.189ms，每秒处理了 444.07 个请求；除此之外，在处理请求的过程中 WEBrick 进程的 CPU 占用率很快达到了 100%，通过这个测试我们就可以看出为什么不应该在生产环境中使用 WEBrick 作为 Ruby 的应用服务器，在业务逻辑和代码更加复杂的情况下，WEBrick 的性能想必也不会达到期望。

## Thin

在 2006 和 2007 两年，Ruby 社区中发布了两个至今都非常重要的开源项目，其中一个是 Mongrel，它提供了标准的 HTTP 接口，同时多语言的支持也使得 Mongrel 在当时非常流行，另一个项目就是 Rack 了，它在 Web 应用和 Web 服务器之间建立了一套统一的 [标准](https://www.google.com/search?client=safari&rls=en&q=rack+spec&ie=UTF-8&oe=UTF-8)，规定了两者的协作方式，所有的应用只要遵循 Rack 协议就能够随时替换底层的应用服务器。

![rack-protoco](https://img.draveness.me/2017-11-17-rack-protocol.png)

随后，在 2009 年出现的 Thin 就站在了巨人的肩膀上，同时遵循了 Rack 协议并使用了 Mongrel 中的解析器，而它也是 Ruby 社区中第一个使用 Reactor 模型的 Web 服务器。

![thin-concurrency-model](https://img.draveness.me/2017-11-17-thin-concurrency-model.png)

Thin 使用 Reactor 模型处理客户端的 HTTP 请求，每一个请求都会交由 EventMachine，通过内部对事件的分发，最终执行相应的回调，这种事件驱动的 IO 模型与 node.js 非常相似，使用单进程单线程的并发模型却能够快速处理 HTTP 请求；在这里，我们仍然使用 ApacheBench 以及同样的负载对 Thin 的性能进行简单的测试。

```
Concurrency Level:      100
Time taken for tests:   4.221 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      880000 bytes
HTML transferred:       100000 bytes
Requests per second:    2368.90 [#/sec] (mean)
Time per request:       42.214 [ms] (mean)
Time per request:       0.422 [ms] (mean, across all concurrent requests)
Transfer rate:          203.58 [Kbytes/sec] received

```

对于一个相同的 HTTP 请求，Thin 的吞吐量大约是 WEBrick 的四倍，每秒能够处理 2368.90 个请求，同时处理的速度也大幅降低到了 42.214ms；在压力测试的过程中虽然 CPU 占用率有所上升但是在处理的过程中完全没有超过 90%，可以说 Thin 的性能碾压了 WEBrick，这可能也是开发者都不会在生产环境中使用 WEBrick 的最重要原因。

但是同样作为单进程运行的 Thin，由于没有 master 进程的存在，哪怕当前进程由于各种各样奇怪的原因被操作系统杀掉，我们也不会收到任何的通知，只能手动重启应用服务器。

## Unicorn

与 Thin 同年发布的 Unicorn 虽然也是 Mongrel 项目的一个 fork，但是使用了完全不同的并发模型，每Unicorn 内部通过多次 `fork` 创建多个 worker 进程，所有的 worker 进程也都由一个 master 进程管理和控制：

![unicorn-master-workers](https://img.draveness.me/2017-11-17-unicorn-master-workers.png)

由于 master 进程的存在，当 worker 进程被意外杀掉后会被 master 进程重启，能够保证持续对外界提供服务，多个进程的 worker 也能够很好地压榨多核 CPU 的性能，尽可能地提高请求的处理速度。

![unicorn-concurrency-model](https://img.draveness.me/2017-11-17-unicorn-concurrency-model.png)

一组由 master 管理的 Unicorn worker 会监听绑定的两个 Socket，所有来自客户端的请求都会通过操作系统内部的负载均衡进行调度，将请求分配到不同的 worker 进程上进行处理。

不过由于 Unicorn 虽然使用了多进程的并发模型，但是每个 worker 进程在处理请求时都是用了阻塞 I/O 的方式，所以如果客户端非常慢就会大大影响 Unicorn 的性能，不过这个问题就可以通过反向代理来 nginx 解决。

![unicorn-multi-processes](https://img.draveness.me/2017-11-17-unicorn-multi-processes.png)

在配置 Unicorn 的 worker 数时，为了最大化的利用 CPU 资源，往往会将进程数设置为 CPU 的数量，同样我们使用 ApacheBench 以及相同的负载测试一个使用 8 核 CPU 的 Unicorn 服务的处理效率：

```
Concurrency Level:      100
Time taken for tests:   2.401 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1110000 bytes
HTML transferred:       100000 bytes
Requests per second:    4164.31 [#/sec] (mean)
Time per request:       24.014 [ms] (mean)
Time per request:       0.240 [ms] (mean, across all concurrent requests)
Transfer rate:          451.41 [Kbytes/sec] received

```

经过简单的压力测试，当前的一组 Unicorn 服务每秒能够处理 4000 多个请求，每个请求也只消耗了 24ms 的时间，比起使用单进程的 Thin 确实有着比较多的提升，但是并没有数量级的差距。

除此之外，Unicorn 由于其多进程的实现方式会占用大量的内存，在并行的处理大量请求时你可以看到内存的使用量有比较明显的上升。

## Puma

距离 Ruby 社区的第一个 webserver WEBrick 发布的 11 年之后的 2011 年，Puma 正式发布了，它与 Thin 和 Unicorn 一样都从 Mongrel 中继承了 HTTP 协议的解析器，不仅如此它还基于 Rack 协议重新对底层进行了实现。

![puma-cluster-mode](https://img.draveness.me/2017-11-17-puma-cluster-mode.png)

与 Unicorn 不同的是，Puma 是用了多进程加多线程模型，它可以同时在 fork 出来的多个 worker 中创建多个线程来处理请求；不仅如此 Puma 还实现了用于提高并发速度的 Reactor 模块和线程池能够在提升吞吐量的同时，降低内存的消耗。

![puma-concurrency-mode](https://img.draveness.me/2017-11-17-puma-concurrency-model.png)

但是由于 MRI 的存在，往往都需要使用 JRuby 才能最大化 Puma 服务器的性能，但是即便如此，使用 MRI 的 Puma 的吞吐量也能够轻松达到 Unicorn 的两倍。

```
Concurrency Level:      100
Time taken for tests:   1.057 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      750000 bytes
HTML transferred:       100000 bytes
Requests per second:    9458.08 [#/sec] (mean)
Time per request:       10.573 [ms] (mean)
Time per request:       0.106 [ms] (mean, across all concurrent requests)
Transfer rate:          692.73 [Kbytes/sec] received

```

在这里我们创建了 8 个 Puma 的 worker，每个 worker 中都包含 16~32 个用于处理用户请求的线程，每秒中处理的请求数接近 10000，处理时间也仅为 10.573ms，多进程、多线程以及 Reactor 模式的协作确实能够非常明显的增加 Web 服务器的工作性能和吞吐量。

在 Puma 的 [官方网站](http://puma.io/) 中，有一张不同 Web 服务器内存消耗的对比图：

![memory-usage-comparision](https://img.draveness.me/2017-11-17-memory-usage-comparision.png)

我们可以看到，与 Unicorn 相比 Puma 的内存使用量几乎可以忽略不计，它明显解决了多个 worker 占用大量内存的问题；不过使用了多线程模型的 Puma 需要开发者在应用中保证不同的线程不会出现竞争条件的问题，Unicorn 的多进程模型就不需要开发者思考这样的事情。

## 对比

上述四种不同的 Web 服务器其实有着比较明显的性能差异，在使用同一个最简单的 Web 应用时，不同的服务器表现出了差异巨大的吞吐量：

![ruby-webservers](https://img.draveness.me/2017-11-17-ruby-webservers.jpeg)

Puma 和 Unicorn 两者之间可能还没有明显的数量级差距，1 倍的吞吐量差距也可能很容易被环境因素抹平了，但是 WEBrick 可以说是绝对无法与其他三者匹敌的。

上述的不同服务器其实有着截然不同的 I/O 并发模型，因为 MRI 中 GIL 的存在我们很难利用多核 CPU 的计算资源，所以大多数多线程模型在 MRI 上的性能可能只比单线程略好，达不到完全碾压的效果，但是 JRuby 或者 Rubinius 的使用确实能够利用多核 CPU 的计算资源，从而增加多线程模型的并发效率。

![jruby](https://img.draveness.me/2017-11-17-jruby.png)

传统的 I/O 模型就是在每次接收到客户端的请求时 fork 出一个新的进程来处理当前的请求或者在服务器启动时就启动多个进程，每一个进程在同一时间只能处理一个请求，所以这种并发模型的吞吐量有限，在今天已经几乎看不到使用 **accept & fork** 这种方式处理请求的服务器了。

目前最为流行的方式还是混合多种 I/O 模型，同时使用多进程和多线程压榨 CPU 计算资源，例如 Phusion Passenger 或者 Puma 都支持在单进程和多进程、单线程和多线程之前来回切换，配置的不同会创建不同的并发模型，可以说是 Web 服务器中最好的选择了。

最后要说的 Thin 其实使用了非常不同的 I/O 模型，也就是事件驱动模型，这种模型在 Ruby 社区其实并没有那么热门，主要是因为 Rails 框架以及 Ruby 社区中的大部分项目并没有按照 Reactor 模型的方式进行设计，默认的文件 I/O 也都是阻塞的，而 Ruby 本身也可以利用多进程和多线程的计算资源，没有必要使用事件驱动的方式最大化并发量。

![nodejs-logo](https://img.draveness.me/2017-11-17-nodejs-logo.jpg)

Node.js 就完全不同了。Javascript 作为一个所有操作都会阻塞主线程的语言，更加需要事件驱动模型让主线程只负责接受 HTTP 请求，其余的脏活累活都交给线程池来做了，结果的返回都通过回调的形式通知主线程，这样才能提高吞吐量。

## 总结

在这个系列的文章中，我们先后介绍了 Rack 的实现原理以及 Rack 协议，还有四种 webserver 包括 WEBrick、Thin、Unicorn 和 Puma 的实现，除了这四种应用服务器之外，Ruby 社区中还有其他的应用服务器，例如：Rainbows 和 Phusion Passenger，它们都有各自的实现以及优缺点。

从当前的情况来看，还是更推荐开发者使用 Puma 或者 Phusion Passenger 作为应用的服务器，这样能获得最佳的效果。



















