# 前言

在ASP.NET Core中最大的更改之一是对Http请求管道的更改，在ASP.NET中我们了解`HttpHandler`和`HttpModule`但是到现在这些已经被替换为`中间件`那么下面我们来看一下他们的不同处。

# HttpHandler

Handlers处理基于扩展的特定请求，HttpHandlers作为进行运行，同时做到对ASP.NET响应请求。他是一个实现`System.Web.IHttphandler`接口的类。任何实现IHttpHandler接口的类都可以作为Http请求处理响应的目标程序。
它提供了对文件特定的扩展名处理传入请求，
ASP.NET框架提供了一些默认的Http处理程序，最常见的处理程序是处理`.aspx`文件。下面提供了一些默认的处理程序。


Handler | Extension | Description
---|---|---
Page Handler | .aspx | handle normal WebPages
User Control Handler | .ascx | handle Web user control pages
Web Service Handler  | .asmx | handle Web service pages
Trace Handler        | trace.axd | handle trace functionality

## 创建一个自定义HttpHandler

```csharp

public class CustomHttpHandler:IHttpHandler
{
    
    public bool IsReusable
    {
        //指定是否可以重用处理程序
        get {return true;}
    }
    
    public void ProcessRequest(HttpContext context)
    {
        //TODO
        throw new NotImplementedException();
    }
}

```
在web.config中添加配置项
```
<!--IIS6或者IIS7经典模式-->  
  <system.web>  
    <httpHandlers>  
      <add name="mycustomhandler" path="*.aspx" verb="*" type="CustomHttpHandler"/>  
    </httpHandlers>  
  </system.web>  
   
<!--IIS7集成模式-->  
  <system.webServer>  
    <handlers>  
       <add name="mycustomhandler" path="*.aspx" verb="*" type="CustomHttpHandler"/>  
    </handlers>  
  </system.webServer>  
```

## 异步HttpHandlers

异步的话需要继承`HttpTaskAsyncHandler`类，`HttpTaskAsyncHandler`类实现了`IHttpTaskAsyncHandler`和`IHttpHandler`接口
```
public class CustomHttpHandlerAsync:HttpTaskAsyncHandler
{
    
    public override Task ProcessRequestAsync(HttpContext context)
    {

        throw new NotImplementedException();
    }
}
```

# HttpModule

下面是来自MSDN

> Modules are called before and after the handler executes. Modules enable developers to intercept, participate in, or modify each individual request. Modules implement the IHttpModule interface, which is located in the System.Web namespace.
> 

`HttpModule`类似过滤器,它是一个基于事件的，在应用程序发起到结束的整个生命周期中访问事件


## 自定义一个HttpModule

```csharp
public class CustomModule : IHttpModule
    {
        public void Dispose()
        {
            throw new NotImplementedException();
        }

        public void Init(HttpApplication context)
        {
            context.BeginRequest += new EventHandler(BeginRequest);
            context.EndRequest += new EventHandler(EndRequest);
        }
        void BeginRequest(object sender, EventArgs e)
        {
            ((HttpApplication)sender).Context.Response.Write("请求处理前");
        }

        void EndRequest(object sender, EventArgs e)
        {
            ((HttpApplication)sender).Context.Response.Write("请求处理结束后");
        }
    }


```

web.config中配置

```
<!--IIS6或者IIS7经典模式-->  
<system.web>  
    <httpModules>  
      <add name="mycustommodule" type="CustomModule"/>  
    </httpModules>  
  </system.web>  
<!--IIS7集成模式-->  
<system.webServer>  
    <modules>  
      <add name="mycustommodule" type="CustomModule"/>  
    </modules>  
</system.webServer>  
```
# 中间件

中间件可以视为集成到Http请求管道中的小型应用程序组件，它是ASP.NET中HttpModule和HttpHandler的结合，它可以处理身份验证、日志请求记录等。

# 中间件和HttpModule的相似处

中间件和HttpMoudle都是可以处理每个请求，同时可以配置进行返回我们自己的定义。

# 中间件和httpModule之间的区别


HttpModule | 中间件
---|---
通过web.config或global.asax配置 | 在Startup文件中添加中间件
执行顺序无法控制，因为模块顺序主要是基于应用程序生命周期事件| 可以控制执行内容和执行顺序按照添加顺序执行。
请求和响应执行顺序保持不变 |响应中间件顺序与请求顺序相反
HttpModules可以附件特定应用程序事件的代码|中间件独立于这些事件

# 中间件示例

```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseHttpsRedirection();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
```

在如上代码片段中我们有一些中间件的添加，同时也有中间件的顺序。


# Reference
https://www.talkingdotnet.com/asp-net-core-middleware-is-different-from-httpmodule/

https://support.microsoft.com/en-us/help/307985/info-asp-net-http-modules-and-http-handlers-overview