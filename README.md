# redirect
'use strict'

var url = require('url')
var isUrl = /^https?:/

function Redirect (request) {
  this.request = request
  this.followRedirect = true
  this.followRedirects = true
  this.followAllRedirects = false
  this.followOriginalHttpMethod = false
  this.allowRedirect = function () { return true }
  this.maxRedirects = 10
  this.redirects = []
  this.redirectsFollowed = 0
  this.removeRefererHeader = false
}

Redirect.prototype.onRequest = function (options) {
  var self = this

  if (options.maxRedirects !== undefined) {
    self.maxRedirects = options.maxRedirects
  }
  if (typeof options.followRedirect === 'function') {
    self.allowRedirect = options.followRedirect
  }
  if (options.followRedirect !== undefined) {
    self.followRedirects = !!options.followRedirect
  }
  if (options.followAllRedirects !== undefined) {
    self.followAllRedirects = options.followAllRedirects
  }
  if (self.followRedirects || self.followAllRedirects) {
    self.redirects = self.redirects || []
  }
  if (options.removeRefererHeader !== undefined) {
    self.removeRefererHeader = options.removeRefererHeader
  }
  if (options.followOriginalHttpMethod !== undefined) {
    self.followOriginalHttpMethod = options.followOriginalHttpMethod
  }
}

Redirect.prototype.redirectTo = function (response) {
  var self = this
  var request = self.request

  var redirectTo = null
  if (response.statusCode >= 300 && response.statusCode < 400 && response.caseless.has('location')) {
    var location = response.caseless.get('location')
    request.debug('redirect', location)

    if (self.followAllRedirects) {
      redirectTo = location
    } else if (self.followRedirects) {
      switch (request.method) {
        case 'PATCH':
        case 'PUT':
        case 'POST':
        case 'DELETE':
          // Do not follow redirects
          break
        default:
          redirectTo = location
          break
      }
    }
  } else if (response.statusCode === 401) {
    var authHeader = request._auth.onResponse(response)
    if (authHeader) {
      request.setHeader('authorization', authHeader)
      redirectTo = request.uri
    }
  }
  return redirectTo
}

Redirect.prototype.onResponse = function (response) {
  var self = this
  var request = self.request

  var redirectTo = self.redirectTo(response)
  if (!redirectTo || !self.allowRedirect.call(request, response)) {
    return false
  }

  request.debug('redirect to', redirectTo)

  // ignore any potential response body.  it cannot possibly be useful
  // to us at this point.
  // response.resume should be defined, but check anyway before calling. Workaround for browserify.
  if (response.resume) {
    response.resume()
  }

  if (self.redirectsFollowed >= self.maxRedirects) {
    request.emit('error', new Error('Exceeded maxRedirects. Probably stuck in a redirect loop ' + request.uri.href))
    return false
  }
  self.redirectsFollowed += 1

  if (!isUrl.test(redirectTo)) {
    redirectTo = url.resolve(request.uri.href, redirectTo)
  }

  var uriPrev = request.uri
  request.uri = url.parse(redirectTo)

  // handle the case where we change protocol from https to http or vice versa
  if (request.uri.protocol !== uriPrev.protocol) {
    delete request.agent
  }

  self.redirects.push({ statusCode: response.statusCode, redirectUri: redirectTo })

  if (self.followAllRedirects && request.method !== 'HEAD' &&
    response.statusCode !== 401 && response.statusCode !== 307) {
    request.method = self.followOriginalHttpMethod ? request.method : 'GET'
  }
  // request.method = 'GET' // Force all redirects to use GET || commented out fixes #215
  delete request.src
  delete request.req
  delete request._started
  if (response.statusCode !== 401 && response.statusCode !== 307) {
    // Remove parameters from the previous response, unless this is the second request
    // for a server that requires digest authentication.
    delete request.body
    delete request._form
    if (request.headers) {
      request.removeHeader('host')
      request.removeHeader('content-type')
      request.removeHeader('content-length')
      if (request.uri.hostname !== request.originalHost.split(':')[0]) {
        // Remove authorization if changing hostnames (but not if just
        // changing ports or protocols).  This matches the behavior of curl:
        // https://github.com/bagder/curl/blob/6beb0eee/lib/http.c#L710
        request.removeHeader('authorization')
      }
    }
  }

  if (!self.removeRefererHeader) {
    request.setHeader('referer', uriPrev.href)
  }

  request.emit('redirect')

  request.init()

  return true
}

exports.Redirect = Redirect





解析





redirect 重定向
followRedirect 跟随重定向
 request 请求

首先定义一个路径的函数
指针函数 this
followRedirect跟随重定向 是true
followRedirects跟随重定向s 是true
followAllRedirects跟随全部重定向是false
followOriginalHttpMethod跟随起初的 http方法是 false
allowRedirect允许重定向 函数 返回 true
maxRedirects最大的重定向 =10
redirects重定向为[]
redirectsFollowed重定向跟随 = 0
removeRefererHeader移除引用头部= false

重定向.原型.要求 = 构造函数 （options）{
 定义 var (self) = this函数

如果 （options.最大重定向 ！== undefined）{
则(self).最大重定向 = optinons.最大重定向}

如果 （typeof(类型) options. 跟随重定向函数 === 'function'）{
(self) 允许重定向 = options.跟随重定向}

如果（options.跟随重定向 ！== undefined）{
self.跟随重定向 =！！options.跟随重定向}

如果（options.跟随重定向s ！== undefined）{
self.跟随重定向s =options.跟随重定向s}

如果（self.跟随重定向s || self. 跟随重定向s）{
self.重定向= self.重定向 ||[] }

如果

如果

}



重定向.原型.重定向 to = 函数（response 响应）{
var self =this 函数
定义 var requst=self.request

var redirectTo = null

if (response.statusCode >= 300 response.statusCode < 400 response.caseless.has('location'))
都成立时为真{

定义loccation = response.caseless.get('location')  get函数获取

请求.调试 （'redirect', location）


if(self.followAllRedirects){

则 redirectTo = location(位置){

 else if（self.followRedirects）{
switch (request.method 方法){
        case 'PATCH':
        case 'PUT':
        case 'POST':
        case 'DELETE':

      break

       default:
          redirectTo = location
          break

}查找看是否相同

else if (response.statusCode === 401) {

var authHeader = request._auth.onResponse(response)

if (authHeader){

request.setHeader('authorization', authHeader)
      redirectTo = request.uri
}
return redirectTo(返回redirectTo)


（重定向.原型.响应)Redirect.prototype.onResponse = function (response) {
   var self =this (等于this函数)
   var request = self.request

 var redirectTo = self.redirectTo(response)
if (!重定向To ||  self.allowRedirect.call(呼叫)(request, response)){
 return false   返回 false
}

 request.debug ('redirect to', redirectTo) 调试

//忽略任何潜在的反应体。它不可能有用。
//在这一点上esponse.resume应定义，但反正检查之前调用。解决方法browserify

if(response.resume 恢复){
    response.resume()  
}

if(self.redirectsFollowed >= self.maxRedirects){
   request.emit(发出)（'error' 错误  new Error（'Exceeded(超过) maxRedirects.Probably stuck in a redirect loop（可能停留在重定向循环中)'+ request.uri.href))
  return false 返回false
}
 self.redirectsFollowed += 1  这个函数+=1


if (!isUrl.test(测试)(redirectTo){
 redirectTo =url.resolve(决定)(request.uri.href, redirectTo)
}

var uriPrev = request.uri
request.uri = url.parse(解析)(redirectTo)

//处理我们将协议从HTTPS更改为http或反之亦然的情况
if (request.uri.protocol(协议) !== uriPrev.protocol){
    delete request.agent （删除request.agent ）
}

self.redirects.push({ statusCode: response.statusCode, redirectUri: redirectTo })
               推  

if(self.followAllRedirects && request.method !== 'HEAD' &&
    response.statusCode !== 401 && response.statusCode !== 307)

self.followAllRedirects  request.method !== 'HEAD' response.statusCode !== 401 
response.statusCode !== 307  都实现时为真 这四个表达式{

 request.method = self.followOriginalHttpMethod ? request.method : 'GET' (获取)
}
/ / request.method = 'get' /强制所有重定向使用得到  ||  评论了修复# 215

delete request.src  删除 src
 delete request.req  删除 req
delete request._started  删除_started

if(response.statusCode !== 401 且 response.statusCode !== 307){
/从前面的响应中删除参数，除非这是第二个请求
/对于需要进行身份验证的服务器

delete request.body  删除body 
delete request._form  删除_form

if(request.headers){
  request.removeHeader('host')  请求.移除头部 host
  request.removeHeader('content-type')   content-type
  request.removeHeader('content-length')  content-length
  
   if(request.uri.hostname !== request.originalHost.split(':')[0]) {
    //删除授权如果改变主机名（但如果只是
    //更改端口或协议）。这与旋度的行为相匹配：
    //https://github.com/bagder/curl/blob/6beb0eee/lib/http.c#L710
     request.removeHeader('authorization')  授权
}

 } 
   
   }

  if(!self.removeRefererHeader){
      request.setHeader('referer', uriPrev.href)  
     }

request.emit('redirect')  发出

request.init()  初始化

return true  返回 true

}

exports.Redirect = Redirect   接口Redirect =Redirect

readme

    followRedirect - follow HTTP 3xx responses as redirects (default: true). This property can also be implemented as function which gets response object as a single argument and should return true if redirects should continue or false otherwise.
    followAllRedirects - follow non-GET HTTP 3xx responses as redirects (default: false)
    followOriginalHttpMethod - by default we redirect to HTTP method GET. you can enable this property to redirect to the original HTTP method (default: false)
    maxRedirects - the maximum number of redirects to follow (default: 10)
    removeRefererHeader - removes the referer header when a redirect happens (default: false). Note: if true, referer header set in the initial request is preserved during redirect chain.

followredirect遵循HTTP3xx-重定向响应(默认:true)。该特征还可以被实现为功能的响应对象作为参数并返回true，如果应该重定向或继续，否则返回FALSE。348下列非-followallredirects-GETHTTP3xx重定向响应(默认:false)349-我们默认followoriginalhttpmethod重定向到HTTPGet方法。此属性使您可以重定向到原始HTTP方法(默认:false)350maxredirects-的最大数目遵循重定向(默认值:10)351-removerefererheaderReferer报头去除时发生的重定向（缺省值：false）。注：如果为真，Referer标头中设定的初始请求期间保留重定向链中。



redirect是服务器根据逻辑，发送一个状态码，告诉浏览器重新去请求那个地址，所以地址栏显示的是心的URL
效率低 不能共享数据 一般用于用户注销登录时返回主页面和跳转到其他的网站等

request 有关于客户端所发出的请求的对象，只要时有关客户端请求的信息，都可以由他取得








