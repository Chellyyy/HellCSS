# 同源策略 Same Origin Policy

日常开发中最常与网络打交道，那关于浏览器的同源策略和跨域相关的知识是该整理一下了。
首先需要明确的是，同源策略是浏览器的安全策略，由于存在这个策略，我们才需要对各种跨域需求进行处理。

同源策略的主要目的是为了保护用户的信息安全。

## 什么是同源

同源的含义其实比较好理解，实际上就是三点

- 协议相同
- 域名相同
- 端口相同

| url                                                | 说明                | 是否同源 |
| :------------------------------------------------- | :------------------ | :------- |
| http://www.test.com https://www.test.com           | 同一域名，不同协议  | 不同源   |
| http://www.test.com https://www.test1.com          | 不同域名            | 不同源   |
| http://www.test.com:8080 https://www.test.com:8081 | 同一域名，不同端口  | 不同源   |
| https://www.test.com http://192.168.1.1            | 域名和域名对应的 ip | 不同源   |

## 同源限制了什么

1、Cookie、localStorage 和 IndexDB 无法读取  
2、DOM 无法获得  
3、AJAX 请求不能发送

## 规避方法

### 1、读取 Cookie

cookie 的读取与其他属性有些不同。因为 cookie 的可见性是由 domain 属性和 path 属性决定的，所以他其实不受协议和端口的限制。只要是同一个 domain 和可见 path 下的 cookie，都能读取到。

#### 可见 path

- / 只能读取 / 下的 cookie
- /test 可以读取 / 和 /test 下的 cookie

由于 cookie 的读取本身具有局限性，所以跨域后的规避可以这样操作

- 前端读取不同 domain 的 cookie，可以把通过设置 document.domain 来读取相应 cookie。当然，你设置的 domain 必须是你当前 domain 的上一级域名，否则会报如下的错误。  
  `Uncaught DOMException: Failed to set the 'domain' property on 'Document': 'test' is not a suffix of 'localhost'.`
- 前端和服务器端设置 cookie 时指定可读的 domian 和 path，这个 domain 也遵循上面的规则，只能设置当前 domain 或 domain 上一级的域名，否则会设置失败。本地测试的现象是没有报错，但是也没有设置成功。

### 2、读取 localStorage、IndexedDB 和 DOM

localStorage、IndexedDB 和 DOM 是实实在在受同源策略限制的，协议、域名和端口任意一项不相同就无法读取。  
其实对于他们的读取可以简化成不同源的两个页面如何通信。  
本身 localStorage 和 IndexedDB 的出现就是为了让我们能够更好地存储数据，而获取 DOM 大多情况也是为了获取 DOM 中的各种数据。泛化到业务场景中，也就是传递数据的事儿了。

目前可以通过以下几种方式来进行通信。

- window.name  
  使用 window.name 的原理是同源 iframe 可以读取 contentWindow。具体操作如下  
  1、a 页面先载入一个不同源的 iframe 页面 b  
  2、在 b 页面 中修改 window.name 为需要的传递的数据  
  3、a 页面中修改这个 iframe 的 src 为同源的页面 c  
  4、a 页面获取之前设置的 window.name  
  按照这样的方式进行操作就能获取到子页面中的数据。如果子页面想要获取父页面中的数据，可以将 1、3 步骤换一下，2 步骤改成父页面直接修改 iframe.contentWindow.name 为需要传递的数据即可。  
  因为是通过 name 属性来传递参数，所以可传递的数据量很大，基本就是字符串长度的最大值。

  ```a 页面操作
  //记录 iframe onload 事件的加载次数
  var state = 0;
  var iframe = document.createElement("iframe");
  // 加载跨域页面
  iframe.src = "http://sub.test.com/test/test.html";
  // onload 事件会触发2次，第1次加载跨域页，在跨域页中将所需要的数据赋给 window.name
  iframe.onload = function () {
    if (state === 1) {
      // 第2次 onload 成功后，读取同域 window.name 中数据
      var data = iframe.contentWindow.name;
    } else if (state === 0) {
      // 第1次onload(跨域页)成功后，切换到同域代理页面
      iframe.contentWindow.location = "./testB.html";
      state = 1;
    }
  };
  document.body.appendChild(iframe);
  ```

- fragment identifier 片段标识符  
  fragment identifier 其实就是 url # 后面的部分。常写 SPA 应用的小伙伴应该会对他很熟悉，因为 hash 模式的 router 就是基于他实现的。修改嵌入的 iframe 的 src 为 url#需要传递的数据，iframe 页面就能通过监听 hashchange 事件获得传递的数据。如果是子页面向父页面传递数据要多加一步，得先让父页面把自己当前的 url 传递给子页面，然后子页面去修改父页面的 href。  
  使用这个方法传递数据的限制暂时还未测试，猜测应该会和浏览器限制 url 的长度有关。
  ```
  //父->子 a页面
  var iframe = document.createElement("iframe");
  // 加载跨域页面
  iframe.src = url;
  setTimeout(function () {
    iframe.src = url + "#a=111";
  }, 1000);
  document.body.appendChild(iframe);

  //父->子 b页面
  window.onhashchange = function () {
    alert(window.location.hash)
  }
  ```

  ```
  //子->父 a页面
  var iframe = document.createElement("iframe");
  // 加载跨域页面
  iframe.src = url;
  setTimeout(function () {
    var selfurl = document.location.href;
    iframe.src = url + "#" + selfurl;
  }, 1000);
  document.body.appendChild(iframe);
  window.onhashchange = function () {
    alert("from son" + window.location.hash);
  };

  //子->父 b页面
  window.onhashchange = function () {
    let target = window.location.hash.slice(1);
    window.parent.parent.location.href = target + "#111aaaa"
  }
  ```

- postMessage  
  严格意义上来说上面两种方法都是对跨域页面获取数据的破解，postMessage 才是正统的非同源页面之间传递数据的方法。
  > window.postMessage() 方法提供了一种受控机制来规避此限制，只要正确的使用，这种方法就很安全。
  
  postMessage 是新增的 API，使用前记得查看一下兼容性。
  使用上需要注意的就是调用 postMessage 和监听 message 事件的的主体是同一个。也就是说子页面向父页面发送消息时，需要获取 window.parent 去调用 postMessage。父页面向子页面发送消息时，可以直接获取 iframe，然后调用 postMessage。  
  > postMessage((message, targetOrigin, [transfer]); 
   message  
   将要发送到其他 window 的数据。它将会被结构化克隆算法序列化。这意味着你可以不受什么限制的将数据对象安全的传送给目标窗口而无需自己序列化。  
   targetOrigin  
   通过窗口的 origin 属性来指定哪些窗口能接收到消息事件，其值可以是字符串"*"（表示无限制）或者一个 URI。  
   在发送消息的时候，如果目标窗口的协议、主机地址或端口这三者的任意一项不匹配 targetOrigin 提供的值，那么消息就不会被发送；只有三者完全匹配，消息才会被发送。  
   这个机制用来控制消息可以发送到哪些窗口；例如，当用 postMessage 传送密码时，这个参数就显得尤为重要，必须保证它的值与这条包含密码的信息的预期接受者的 origin 属性完全一致，来防止密码被恶意的第三方截获。  
   如果你明确的知道消息应该发送到哪个窗口，那么请始终提供一个有确切值的 targetOrigin，而不是*。不提供确切的目标将导致数据泄露到任何对数据感兴趣的恶意站点。  
   transfer 可选  
   是一串和message 同时传递的 Transferable 对象. 这些对象的所有权将被转移给消息的接收方，而发送一方将不再保有所有权 
 
  下面这个示例是子页面向父页面发送消息。

  ```
  // a 页面
  window.addEventListener("message", function (event) {
    console.log(event);
  }, false);

  // b 页面
  var b = "bbb";
  window.parent.postMessage({
    b
  }, "*")

  ```
  

- 另外特别需要注意的是关于 DOM 的获取，如果只是两个不同子域的页面，将 document.domain 设置为同一主域就可以读取相应数据。

  ```
  //a 页面 main.test.com
  var a = "aaa";
  // domain 设置为主域
  document.domain = "test.com";
  var iframe = document.createElement("iframe");
  // 加载跨域页面
  iframe.src = "sub.test.com";
  document.body.appendChild(iframe);
  iframe.onload = function(){
    // 可以获取到变量
    var data = iframe.contentWindow.b
    // 可以获取到 DOM
    var dom = iframe.contentWindow.document.body;
  }

  //b 页面 sub.test.com
  var b = "bbb";
  // domain 设置为主域
  document.domain = "test.com";
  ```
### 3、AJAX 请求
AJAX 请求跨域日常使用比较多，常用的方法有以下几种  

- JSONP  
  这个方法是向服务器请求的时候，在 url 后面写上 callback 方法的名字，请求返回实际上是返回了一个 调用 callback 方法的 js 文件。需要返回的参数也就在调用的时候传进去了。  
  所以局限也很明确，只支持 get 方法。

- 使用代理服务器  
  所有的请求先发送给这个代理服务器，由这个代理服务器去请求实际的接口，再把需要的数据返回。  
  （这个方式就可以真切地体会到，同源策略只是浏览器的一种安全策略）

- 使用 CORS Cross-Origin Resource Sharing 跨域资源共享
  目前基本上所有的浏览器都支持 CORS，所以只需要服务器端进行处理。对于前端来说，请求和同源的 AJAX 请求是一致的。  
  前端发送请求的时候浏览器会自动带上 origin 字段，服务器端去判断这个 origin 是否是可接受的地址，在相应头中设置 Access-Control-Allow-Origin 字段的值。  
  这样前端就能正常获取到数据啦。  
  这里只对 CORS 做了一个简单介绍，详细的下次细说吧。

总结：  
- 读取跨域的 cookie 需要设置 path 和 domain  
- 不同源页面间通讯可以通过设置 window.name、location 的 hash 值和 postMessage 来实现。  
  不难发现的是，设置 hash 和 postMessage 都是通过设置一个监听函数来实现的，所以我们是异步获取数据的。  
  而设置 window.name 虽然是在 iframe.onload 事件中获取的，但是本质上是在等待 iframe 的加载，确保数据已经设置成功。  
  这里还有值得思考的地方。
