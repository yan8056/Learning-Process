# Nginx

- Nginx解决跨域问题

  > - 首先：跨域问题的产生，是由于浏览器有同源策略（记录当前访问的ip和端口，访问其他资源时，如果还是这个ip和端口就允许访问）（同源：协议+域名+端口，均相同）
  >
  > - 其次：当在浏览器中输入Nginx服务的ip和端口进行服务访问时，Nginx会拦截已在Nginx中配置的资源地址，将请求转发到正确的服务上，对于浏览器来说，资源来自多方，但是仅需要使用Nginx的ip和端口，所以不会触发浏览器的同源策略保护
  > - 具体配置方法：
  >   - 可以在Nginx中配置地址重写或转发（rewrite），浏览器访问Nginx服务地址
  >   - 或者在Nginx中添加“允许跨域请求头”（增加了跨域允许请求头的方法不会被同源策略阻止），这种做法和不用Nginx而直接在后端项目中配置请求允许跨域是一样的，然后浏览器访问前端工程的服务地址，前端工程服务访问其他服务时候访问的是Nginx的服务地址（效果是：当浏览器从其他ip和端口的访问对象中访问Nginx时是不会触发同源策略保护的）——同源策略是浏览器功能，Nginx访问后端没有同源策略

- Nginx是一个反向代理服务器

  > - 正向代理：A想访问B，但是网络不通，C可以访问B。所以A进行一些特殊的配置向C发送请求，并指定目标为服务B。C会向服务B转交A的请求并将获得的回复返回给A。这里C就是一个正向代理服务器
  >   - 有两点需要关注：A明确知道目标服务地址；A需要主动进行特殊的配置
  >
  > - 反向代理：A想访问一个服务或者一个服务集群，A无法直接访问（可能因为网络不通，或者服务集群并没有向外界提供具体地址，或地址是变化的），存在第三方服务器C可以访问这个集群（A知道C的地址）。A可以通过访问服务C，由C进行转发，从而获取真实服务或服务集群的资源内容。这里C就是一个反向代理服务器
  >   - 有两点需要关注：A不需要知道目标服务或服务集群的实际地址，只需要知道C的地址；A不需要进行任何配置，只需要直接访问C+资源名，所有的拦截及转发配置都由C配置完成
  >   - 在A不知情的时候，对A来说，C就像真实服务器一样（毕竟A并不知道C做了转发，还是C真的是有业务内容的服务）
  >   - 反向代理由于不需要客户端明确了解真实服务地址，所以非常适合集群情况下使用

- Nginx进行负载均衡

  > - 所谓的负载均衡，其实就是由于单个服务无法支撑庞大的请求量，或者为了避免单服务部署造成的一些不稳定性，服务提供商改为部署一个巨大的服务集群。负载均衡负责将所有对这个服务的请求按照一定规则分配给集群中的具体服务实例处理（一个任务分发功能）
  > - Nginx在进行服务转发的时候，默认实现负载均衡，且提供了多种分发规则，服务提供商可以根据实际部署情况和业务情况按需配置负载均衡策略
  > - Nginx提供的负载均衡策略简要说明（灵活组合使用，据说只有你想不到的）
  >   - 轮询：按顺序，请求轮换被转发到每个服务实例（轮询时候默认权重都为1）
  >   - 权重（weight）：给每个服务实例配置权重，向权重大的服务实例转发更多的请求
  >   - 根据ip（ip_hash）：将请求发送到相同ip的服务器上（使前后端保持稳定连接，可以解决例如断点续传、session共享等问题）
  >   - 最少连接（least_conn）：将请求发送到连接数较少的服务器（实际算法是遍历计算conn/weight的值）
  >   - 响应时间（fair）：（需要第三方插件）将请求优先发送给响应时间短的服务器
  >   - 根据URL（url_hash）：（需要第三方插件）使每一个url（同一个资源请求）定向到同一个服务器
  >
  > - 究竟使用哪种负载均衡配置策略，还需要根据各个策略的详细适用情况进行选择决策