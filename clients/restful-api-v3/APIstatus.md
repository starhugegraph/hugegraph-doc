## 4.16.API状态码

状态代码分为以下五个类别。

| 类别 | 描述 |
| :----------------------- | :------- |
| 1xx：信息 | 通信传输协议级信息。 |
| 2xx：成功 | 表示客户端的请求已成功接受。 |
| 3xx：重定向 | 表示客户端必须执行一些其他操作才能完成其请求。 |
| 4xx：客户端错误 | 此类错误状态代码指向客户端。 |
| 5xx：服务器错误 | 服务器负责这些错误状态代码。 |


```json
200 （OK）
```
它表示REST API成功执行了客户端请求的任何操作，并且2xx系列中没有更具体的代码是合适的。与204状态代码不同，200响应应包括响应主体。响应返回的信息取决于请求中使用的方法，例如：
- 在响应中发送GET对应于所请求资源的实体;
- HEAD对应于所请求资源的实体头字段在响应中发送而没有任何消息体;
- POST一个描述或包含行动结果的实体;
- 跟踪包含终端服务器接收的请求消息的实体。
```json
201 （创建）
```
只要在集合内创建资源，REST API就会使用201状态代码进行响应。由于某些控制器动作，有时可能会创建新资源，在这种情况下，201也是适当的响应。新创建的资源可以由响应实体中返回的URI引用，其中由Location头字段给出的资源的最具体URI。原始服务器必须在返回201状态代码之前创建资源。如果无法立即执行操作，服务器应该响应202（已接受）响应。
```json
202 （已接受）
```
202响应通常用于需要很长时间才能处理的操作。它表示该请求已被接受处理，但处理尚未完成。该请求可能会或可能不会最终被执行，甚至可能在处理时不允许。其目的是允许服务器接受对某些其他进程的请求（可能是每天只运行一次的面向批处理的进程），而不要求用户代理与服务器的连接持续到进程完成为止。使用此响应返回的实体应该包括请求的当前状态的指示以及指向状态监视器的指针（作业队列位置）或某个用户何时可以期望满足请求的估计。
```json
204 （无内容）
```
在204个状态的代码通常是响应发送到一个```PUT```，```POST```或```DELETE```当REST API拒绝发送回在响应消息的主体的任何状态消息或表示请求。API还可以结合```GET```请求发送204以指示所请求的资源存在，但是没有要包括在正文中的状态表示。如果客户端是用户代理，它不应该从导致请求发送的文档视图中更改它的文档视图。此响应主要用于允许在不导致更改用户代理的活动文档视图的情况下进行操作的输入，尽管任何新的或更新的元信息应该应用于当前在用户代理的活动视图中的文档。204响应绝不能包含消息体，因此总是在头字段之后的第一个空行终止。
```json
301 （永久移动）
```
301状态代码表示REST API的资源模型已经过重新设计，并且已为客户端请求的资源分配了新的永久URI。REST API应在响应的Location头中指定新的URI，并且所有将来的请求都应该定向到给定的URI。您很难在API中使用此响应代码，因为您始终可以使用API版本控制新API，同时保留旧API。
```json
302（找到）
```
HTTP响应状态代码302 Found是执行URL重定向的常用方式。具有此状态代码的HTTP响应还将在位置标头字段中提供URL。通过具有该代码的响应来邀请用户代理（例如，web浏览器），以对位置字段中指定的新URL做出第二个（否则相同的）请求。许多Web浏览器以违反此标准的方式实现此代码，将新请求的请求类型更改为GET，而不管原始请求中使用的类型（例如POST）。RFC 1945和RFC 2068指定不允许客户端更改重定向请求上的方法。已经为希望明确清楚客户端期望哪种反应的服务器添加了状态代码303和307。
```json
303 （见其他）
```
303响应表示控制器资源已完成其工作，但它不是发送可能不需要的响应主体，而是向客户端发送响应资源的URI。这可以是临时状态消息的URI，也可以是某些已存在的，更永久的资源的URI。一般而言，303状态代码允许REST API发送对资源的引用，而不强制客户端下载其状态。相反，客户端可以向Location头的值发送GET请求。303响应绝不能被缓存，但对第二个（重定向）请求的响应可能是可缓存的。
```json
304 （未修改）
```
此状态代码类似于204（“无内容”），因为响应正文必须为空。关键的区别在于，当在正文中没有要发送的内容时使用204，而在自请求头If-Modified-Since或If-None-Match指定的版本以来未对资源进行修改时使用304。在这种情况下，不需要重新传输资源，因为客户端仍然具有先前下载的副本。使用它可以在服务器和客户端上节省带宽和重新处理，因为与服务器重新处理的整个页面相比，必须仅发送和接收标头数据，然后使用服务器和客户端的更多带宽再次发送。
```json
307 （临时重定向）
```
307响应表明REST API不会处理客户端的请求。相反，客户端应该将请求重新提交到响应消息的Location头指定的URI。但是，将来的请求仍应使用原始URI。REST API可以使用此状态代码为客户端请求的资源分配临时URI。例如，307响应可用于将客户端请求转移到另一个主机。临时URI应该由响应中的Location字段给出。除非请求方法是```HEAD```，否则响应的实体应该包含一个带有指向新URI的超链接的短超文本注释。如果收到307状态代码以响应除```GET```or之外的请求```HEAD```，则用户代理不得自动重定向请求，除非用户可以确认，因为这可能会改变发出请求的条件。
```json
400 （不良请求）
```
400是通用客户端错误状态，在没有其他4xx错误代码适用时使用。错误可能类似于格式错误的请求语法，无效的请求消息参数或欺骗性请求路由等。客户端不应该在没有修改的情况下重复请求。
```json
401 （未经授权）
```
401错误响应表示客户端尝试在受保护资源上运行而未提供适当的授权。它可能提供了错误的凭据或根本没有。响应必须包含WWW-Authenticate头字段，其中包含适用于所请求资源的质询。客户端可以使用合适的Authorization头字段重复请求。如果请求已包含授权凭据，则401响应表示已拒绝授权这些凭据。如果401响应包含与先前响应相同的挑战，并且用户代理已经尝试过至少一次认证，则应该向用户呈现响应中给出的实体，因为该实体可能包括相关的诊断信息。
```json
403 （禁止）
```
403错误响应表明客户端的请求是正确形成的，但REST API拒绝遵守它，即用户没有资源的必要权限。403响应不是客户端凭证不足的情况; 这将是401（“未经授权”）。身份验证无济于事，请求不应重复。与401 Unauthorized响应不同，身份验证不会产生任何影响。
```json
404 （未找到）
```
404错误状态代码表示REST API无法将客户端的URI映射到资源，但可能在将来可用。客户的后续请求是允许的。没有说明该病症是暂时的还是永久性的。如果服务器通过一些内部可配置的机制知道旧资源永久不可用且没有转发地址，则应该使用410（Gone）状态代码。当服务器不希望确切地说明请求被拒绝的原因，或者没有其他响应适用时，通常会使用此状态代码。
```json
405 （不允许的方法）
```
API响应405错误以指示客户端尝试使用资源不允许的HTTP方法。例如，只读资源只能支持GET和HEAD，而控制器资源可能只允许GET和POST，但不支持PUT或DELETE。405响应必须包含Allow标头，该标头列出资源支持的HTTP方法。例如：允许：GET，POST
```json
406 （不可接受）
```
406错误响应表示API无法生成任何客户端的首选媒体类型，如Accept请求标头所示。例如，```application/xml```如果API仅愿意将数据格式化，则客户端对格式化的数据的请求将接收406响应```application/json```。如果响应可能是不可接受的，则用户代理应该暂时停止接收更多数据并查询用户以决定进一步的操作。
```json
412 （前提条件失败）
```
412错误响应表明客户端在其请求标头中指定了一个或多个前提条件，有效地告诉REST API仅在满足某些条件时才执行其请求。412响应表示不满足这些条件，因此API不发送请求，而是发送此状态代码。
```json
415 （不支持的媒体类型）
```
415错误响应表明API无法处理客户端提供的媒体类型，如Content-Type请求标头所示。例如，```application/xml```如果API仅愿意处理格式化为的数据，则包括格式化的数据的客户机请求将接收415响应```application/json```。例如，客户端将图像上载为```image / svg + xml```，但服务器要求图像使用不同的格式。
```json
500 （内部服务器错误）
```
500是通用REST API错误响应。大多数Web框架在执行一些引发异常的请求处理程序代码时会自动响应此响应状态代码。500错误绝不是客户端的错误，因此客户端重试触发此响应的完全相同的请求是合理的，并希望获得不同的响应。API响应是一般错误消息，在遇到意外情况且没有更合适的消息时给出。
```json
501 （未实施）
```
服务器要么无法识别请求方法，要么无法满足请求。通常，这意味着将来的可用性（例如，Web服务API的新功能）。

