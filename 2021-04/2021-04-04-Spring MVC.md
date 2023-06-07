## Spring MVC

### Spring MVC 工作原理
[相关链接](https://www.cnblogs.com/leskang/p/6101368.html)
1. 客户端（浏览器）发送请求，直接请求到 DispatcherServlet。
1. DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。
1. 解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。
1. HandlerAdapter 会根据 Handler 来调用真正的处理器来处理请求，并处理相应的业务逻辑。
1. 处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
1. ViewResolver 会根据逻辑 View 查找实际的 View。
1. DispaterServlet 把返回的 Model 传给 View（视图渲染）。
1. 把 View 返回给请求者（浏览器）


https://www.cnblogs.com/xiohao/p/9096659.html#:~:text=HandlerAdapter%E5%AD%97%E9%9D%A2%E4%B8%8A%E7%9A%84%E6%84%8F%E6%80%9D%E5%B0%B1%E6%98%AF%E5%A4%84%E7%90%86%E9%80%82%E9%85%8D%E5%99%A8%EF%BC%8C%E5%AE%83%E7%9A%84%E4%BD%9C%E7%94%A8%E7%94%A8%E4%B8%80%E5%8F%A5%E8%AF%9D%E6%A6%82%E6%8B%AC%E5%B0%B1%E6%98%AF%E8%B0%83%E7%94%A8%E5%85%B7%E4%BD%93%E7%9A%84%E6%96%B9%E6%B3%95%E5%AF%B9%E7%94%A8%E6%88%B7%E5%8F%91%E6%9D%A5%E7%9A%84%E8%AF%B7%E6%B1%82%E6%9D%A5%E8%BF%9B%E8%A1%8C%E5%A4%84%E7%90%86%E3%80%82,%E5%BD%93handlerMapping%E8%8E%B7%E5%8F%96%E5%88%B0%E6%89%A7%E8%A1%8C%E8%AF%B7%E6%B1%82%E7%9A%84controller%E6%97%B6%EF%BC%8CDispatcherServlte%E4%BC%9A%E6%A0%B9%E6%8D%AEcontroller%E5%AF%B9%E5%BA%94%E7%9A%84controller%E7%B1%BB%E5%9E%8B%E6%9D%A5%E8%B0%83%E7%94%A8%E7%9B%B8%E5%BA%94%E7%9A%84HandlerAdapter%E6%9D%A5%E8%BF%9B%E8%A1%8C%E5%A4%84%E7%90%86%E3%80%82