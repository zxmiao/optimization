 ## 性能优化的根本目的：      
    要思考的是用户使用网站的体验如何，而不是我们可以节省多少字节，只有准确感知用户的感受，我们才有必要谈毫秒、字节和请求数量等问题。

 ## 针对优化注意事项：       
    1. 防止过早优化：没必要在刚开始阶段就对一个细节进行放大型的优化，因为这样的成本很高，除了代码可读性方面的东西，甚至还可能会引入更多的bug，所以，针对这个问题，我们可以在上线和运营的时候进行监控，当快暴露到问题的时候，进行整体优化。
    2. 本末倒置的关注：网站内容是最重要的，应该查看页面的每个部分，看是否满足网站页面的主要目的，暂时不需要将额外的注意力全部放到一些不关乎本质的东西上。

## 对于性能的分析：        
    1. 使用浏览器的性能分析工具，得到性能分析图表，最著名的就是反向火焰图表，针对浏览器的加载和渲染一目了然。
    2. 投入使用之前缺乏压力测试和性能测试

### 性能优化（从用户输入网址到客户端展现，一步一步优化）
    1. 输入网址  					==> 告诉浏览器你要去哪里
    2. 浏览器查找DNS  				==> 网络世界是IP地址的世界，DNS就是ip地址的别名。从本地DNS到最顶级DNS一步一步的网上爬，直到命中需要访问的IP地址
            a. DNS预解析            使用CDN缓存，加快解析CDN寻找到目标地址（dns-prefetch）

    3. 客户端和服务器建立连接  		==>建立TCP的安全通道，3次握手
            a. CDN加速              使用内容分发网络，让用户更快的获取到所要内容
            b. 启用压缩             在http协议中，使用类似Gzip压缩的方案（对服务器资源不足的时候进行权衡）
            c. 使用HTTP/2协议       http2.0针对1.0优化了很多东西，包括异步连接复用，头压缩等等，使传输更快

    4. 浏览器发送http请求 			==> 默认长连接（复用一个tcp通道，短连接：每次连接完就销毁）
            a. 减少http请求         每个请求从创建到销毁都会消耗很多资源和时间，减少请求就可以相对来说更快展示内容
                    1).                 压缩合并js文件以及css文件
                    2）.                针对图片，可将图片进行合并然后下载，通过css Sprites切割展示（控制大小，太大的话反而适得其反）
            b. 使用http缓存             缓存原则：越多越好，越久越好。让客户端发送更少请求，直接从本地获取，加快性能。
            c. 减少cookie请求           针对非必要数据（静态资源）请求，进行跨域隔离，减少传输内容大小。
            d. 预加载请求               针对一些业务中场景可预加载的内容，提前加载，在之后的用户操作中更少的请求，更快的响应
            e. 选择get和post            在http定义的时候，get本质上就是获取数据，post是发送数据的。get可以在一个TCP报文完成请求，但是post先发header，再发送数据。so，考虑好请求选型。
            f. 缓存方案选型             递进式缓存更新（防止一次性丢失大量缓存，导致负载骤多）

    5. 服务器响应请求 				==> tomcat、IIS等服务器通过本地映射文件关系找到地址或者通过数据库查找到数据，处理完成返回给浏览器
            a. 后端框架选型           \
                                       ==>  更快的响应，前端更快的操作。
            b. 数据库选型和优化       /

    6. 浏览器接受响应 				==> 浏览器根据报文头里面的数据进行不同的响应处理
            a. 解耦第三方依赖          越多的第三方的不确定因素，会导致web的不稳定性和不确定性
            b. 避免404资源             请求资源不到浪费了从请求到接受的所有资源

    7. 浏览器渲染顺序 				==> a.HTML解析开始构建dom树
                                        b.外部脚本和样式表加载完毕
                                            a). 尽快加载css，首先将CSSOM对象渲染出来，然后进行页面渲染，否则导致页面闪屏，用户体验差
                                            b). css选择器是从右往左解析的，so类似#test a {color: #444},css解析器会查找所有a标签的祖先节点，所以效率不是那么高
                                            c). 在css的媒介查询中，最好不要直接和任何css规则直接相关。最好写到link标签中，告诉浏览器，只有在这个媒介下，加载指定这个css
                                        c.脚本在文档内解析并执行
                                            a）. 按需加载脚本，例如现在的webpack就可以打包和按需加载js脚本
                                            b）. 将脚本标记为异步，不阻塞页面渲染，获得最佳启动，保证无关主要的脚本不会阻塞页
                                            c).  慎重选型框架和类库，避免只是用类库和框架的一个功能或者函数，而引用整个文件。
                                        d.HTML DOM完全构造起来
                                            a).  DOM 的多个读操作（或多个写操作），应该放在一起。原则：统一读、统一写。
                                        e.图片和外部内容加载
                                            a). 对多媒体内容进行适当优化，包括恰当使用文件格式，文件处理、渐进式渲染等
                                            b). 避免空的src，空的src仍然会发送请求到服务器
                                            c). 避免在html内容中缩放图片，如果你需要使用小图，则直接使用小图
                                        f.网页完成加载
                                            a). 服务端渲染，特别针对首屏加载很重要的网站，可以考虑这个方案。后端渲染结束，前端接管展示。
            a）针对首屏展示优化
                    1）. 图片懒加载       针对展示只加载第一屏，等用户进行滚动的时候再进行加载。如果用户对下面内容不感兴趣，那么节省的请求。

            b）javascript优化
                    1）. 减少对dom节点的查询，因为每次都会重新去索引这个集合或者元素。或者查询一次缓存起来，以待接下来使用
                    2）. 进行js操作DOM的时候，考虑清楚页面的重绘和重排，因为这些操作相对来说十分损耗性能的。
                    3）. 避免使用eval和Function构造，因为解析器会将这些内容先转换成可执行代码，然后再进行接下去的操作。
                    4）. 减少作用域链的查找，如果一个闭包函数使用到全局作用域的数据，那么每次局部作用域都会一层一层爬到最高作用域取得数据。
                    5）. 数据访问，对非引用类型数据访问和局部变量的访问是最快的。所以如果对引用类型的成员（对象的属性或者数组的成员）访问超过一次，则缓存
                    6）. 将前端可能会使用的一些算法函数写的更优化，在时间和空间复杂度上寻找到一个最优方案。
                    7）. 去除重复加载同一模块脚本
                    8）. 智能事件处理，比如在一个div下有10个按钮，可以在冒泡过程中捕获这个事件源，然后注册

            c） css优化
                    1）. 删除无用规则
                    2）. 内联关键CSS
                    3）. 避免@imports和Base64
                    4）. 启用高性价比属性(如opacity over rgba())
                    5）. 避免重复性工作
                    6）. 不要一条条地改变样式，而要通过改变class，或者csstext属性，一次性地改变样式。
                    7）. 可将元素设为display: none（需要1次重排和重绘），然后N次操作，最后恢复显示
                    8）. position属性为absolute或fixed的元素，重排的开销会比较小，因为不用考虑它对其他元素的影响。

            d） 图片优化（网络请求中80%都是静态资源的请求）
                    1). 图片正确格式的选择
                    2). 图片尺寸的选择，在低分辨率等状况下考虑降级处理（考虑响应式图片）
                    3). 使用正确的工具进行优化（有损压缩、无损压缩）
                    4). 能用css处理和代理的，优先考虑css实现（阴影，滤镜等）
                    5). 正确使用data url，比如说多地使用的地方，不建议data url，可考虑缓存
                    6). 考虑图片的懒加载和元素可见加载方案
                    7). 图片的预加载，在正确的合理的设计节点进行图片的预加载


####所有性能优化总结为三个层面优化：物理层面的优化，设计层面的优化，代码层面的优化

注：设计层优化最主要的核心：衡量如何花费最少代价实现页面功能  

注：HTTP/2（超文本传输协议第2版，最初命名为HTTP 2.0）：
    是HTTP协议的的第二个主要版本，HTTP/2的目标包括异步连接复用，头压缩和请求反馈管线化并保留与HTTP 1.1的完全语义兼容。Google Chrome、Mozilla Firefox、Microsoft Edge和Opera已支持HTTP/2，并默认启用。Internet Explorer自IE 11开始支持HTTP/2，但仅限于Windows 10 Beta，并默认情况激活。

