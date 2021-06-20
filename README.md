Odoo接口开发

Odoo是通过Controller来（控制器）发相应的接口的，路由是通过装饰有的方法定义的route()

1、 先定义一个Controller类

在项目的文件夹controllers新建一个controller.py文件,一定一个Controller类，如下：

2、 使用route()定义路由

路由：Controller

odoo.http.route(route=None, **kw)

装饰器让所装饰的方法成为请求的处理器。该方法必须要是Controller子类的一部分

参数

route 

字符串或数组。决定哪种http请求匹配所装饰方法的路由部分。可以是单个字符串或字符串数组。参见 werkzeug的路由文档了解路由表达式的格式( http://werkzeug.pocoo.org/docs/routing/ )

type 

请求的类型，可为‘http’和‘json’

关于这两个类型，后面会有详细说明，这个坑比较深、、、

auth

 认证方法的类型，可为以下类型：
 
  user：
  
  用户必须认证且当前请求将使用用户的权限进行执行
  
  public：
  
  用户可认证也可不认证。如未认证，当前请求会使用共享Public用户进行执行
  
  none：
  
  即使用没有数据库，方法也一直是活跃的。主要由框架和认证模块使用。它们的请求代码对访问数据库没有任何作用，也没有表明当前数据库或当前用户的配置
  
methods

一个这个路由所应用的http方法的序列。如未指定，允许使用所有方法

cors

Access-Control-Allow-Origin cors 指令值

csrf(Boolean)

是否应为路由启用CSRF保护。默认为True,参见CSRF保护了解更多内容

如下：

3、 结果

4、 关于type等于http和json的说明

在请求接口时，若是指定了Content-Type=application/json，那type只能是json，另外接收的数据不是在接口参数里面，而是需要使用data = json.loads(request.httprequest.data)，来获取请求接口的body数据内容；其他的时候就都是使用http类型
这个坑还是有很多人不明白具体怎么回事，odoo默认是用了Content-Type=application/json，会将接口转至jsonrpc去处理，另外这里在请求type=json时，返回的数据最外面一层还会包上jsonrcp相关数据，如下：

Sucessful request::
  --> {"jsonrpc": "2.0",
       "method": "call",
       "params": {"context": {},
                  "arg1": "val1" },
       "id": null}
  <-- {"jsonrpc": "2.0",
       "result": { "res1": "val1" },
       "id": null}
若是想改变这种默认使用jsonrpc来处理Content-Type=application/json的情况，可以继承修改odoo的处理方法，odoo源码如下：

def get_request(self, httprequest):
//deduce type of request
    if httprequest.mimetype in ("application/json", "application/json-rpc"):
      return JsonRequest(httprequest)
    else:
      return HttpRequest(httprequest)
可以将application/json的方式也转为HttpRequest处理

5、 其他问题

1、 跨域问题

解决方法如下：
odoo官网给的参数解释： cors – The Access-Control-Allow-Origin cors directive value.可	  以在Controller接口上配置参数，如：@http.route("/", type='json', auth="none", csrf=False, method=["POST"], website=True, cors="*")

这样前端在进行接口调用的时候，就可以调通了，跨域问题就解决了。
