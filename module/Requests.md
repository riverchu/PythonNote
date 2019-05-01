# Requests

## 相关文档
---
- [Github首页](https://github.com/kennethreitz/requests)
- [官方文档](http://docs.python-requests.org/)
- [中文文档](http://cn.python-requests.org/zh_CN/latest/)

## 快速上手

导入模块
```python
import requests
```

### 发送请求
获取网页:
```python
>>> r = requests.get('https://api.github.com/events') # r为Response对象
```

Requests简便的API意味着所有HTTP请求类型都是显而易见的。HTTP请求的POST, PUT, DELETE, HEAD以及OPTIONS都可以被指定.
```python
>>> r = requests.post('http://httpbin.org/post', data = {'key':'value'})
>>> r = requests.put('http://httpbin.org/put', data = {'key':'value'})
>>> r = requests.delete('http://httpbin.org/delete')
>>> r = requests.head('http://httpbin.org/get')
>>> r = requests.options('http://httpbin.org/get')
```

### 传递URL参数
Requests允许使用*params*关键字参数，以一个字符串字典来提供这些参数。举例来说，如果想传递*key1=value1*和*key2=value2*到httpbin.org/get, 可以使用如下代码：
```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.get("http://httpbin.org/get", params=payload)

>>> print(r.url) # 等效于 http://httpbin.org/get?key2=value2&key1=value1
```
注意: 字典里值为*None*的键都不会被添加到URL的查询字符串里。

可将一个列表作为值传入：
```python
>>> payload = {'key1': 'value1', 'key2': ['value2', 'value3']}
>>> r = requests.get('http://httpbin.org/get', params=payload)

>>> print(r.url) # http://httpbin.org/get?key1=value1&key2=value2&key2=value3
```

### 响应内容
读取响应内容
```python
>>> import requests
>>> r = requests.get('https://api.github.com/events')
>>> r.text
u'[{"repository":{"open_issues":0,"url":"https://github.com/...
```
Requests会自动解码来自服务器的内容。大多数unicode字符集都能被无缝解码。

请求发出后，Requests会基于HTTP头部对响应的编码作出有根据的推测。当访问`r.text`之时，Requests会使用其推测的文本编码。可以查看Requests使用的编码，并且可以使用`r.encoding`属性来改变它：
```python
>>> r.encoding
'utf-8'
>>> r.encoding = 'ISO-8859-1'
```
改变了编码之后, 每次访问`r.text`Request都会继续使用修改之后的新值. 如果需要指定特殊逻辑指定文本编码, 例如HTTP和XML自身可以指定编码, 可以使用`r.content`找到编码并设置`r.encoding`为相应的编码.

如果需要使用定制编码, 只要在`codecs`模块进行注册, 就可以指定`r.encoding`的值, 然后Requests会自动处理编码.

### 二进制响应内容
以字节的方式访问响应内容: `r.content`
```python
>>> r.content
b'[{"repository":{"open_issues":0,"url":"https://github.com/...
```

Requests会**自动解码**gzip和deflate传输编码的响应数据。

e.g. 以请求返回的二进制数据创建一张图片:
```python
>>> from PIL import Image
>>> from io import BytesIO

>>> i = Image.open(BytesIO(r.content))
```

### JSON响应内容
Requests中有内置的JSON解码器, 处理JSON数据：
```python
>>> r = requests.get('https://api.github.com/events')
>>> r.json() # [{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...
```
如果JSON解码失败,`r.json()`就会抛出一个异常。例如，响应内容是401(Unauthorized)，尝试访问`r.json()`将会抛出`ValueError: No JSON object could be decoded`异常。

需要注意的是，成功调用`r.json()`并**不**意味着响应的成功。有的服务器会在失败的响应中包含一个JSON对象（比如 HTTP 500 的错误细节）。这种JSON会被解码返回。要检查请求是否成功，可使用`r.raise_for_status()`或者检查`r.status_code`是否和期望相同。

### 原始响应内容
若想获取来自服务器的原始*套接字响应*，那么可以访问`r.raw`。前提是在初始请求中设置了`stream=True`。
e.g.
```python
>>> r = requests.get('https://api.github.com/events', stream=True)
>>> r.raw
<requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
>>> r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'
```

一般情况下，应该将文本流保存到文件：
```python
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)
```
使用`Response.iter_content`将会处理大量直接使用`Response.raw`不得不处理的。当流下载时，上面是优先推荐的获取内容方式。Note that chunk_size can be freely adjusted to a number that may better fit your use cases.

### 定制请求头
为请求添加HTTP头部，要传递dict给headers参数。

例如，在前一个示例中我们没有指定 content-type:

e.g. 指定*content-type*
```python
>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}

>>> r = requests.get(url, headers=headers)
```
注意: 定制*header*的优先级低于某些特定的信息源，例如：

- 如果在`.netrc`中设置了用户认证信息，使用`headers=`设置的授权就不会生效。而如果设置了`auth=`参数，`.netrc`的设置就无效了。
- 如果被重定向到别的主机，授权*header*就会被删除。
- 代理授权*header*会被*URL*中提供的代理身份覆盖掉。
- 在我们能判断内容长度的情况下，*header* 的*Content-Length*会被改写。

简单来说Requests不会基于定制header的具体情况改变自己的行为。只不过在最后的请求中，所有的header信息都会被传递进去。

注意: 所有的header值必须是string、bytestring 或者 unicode。尽管传递 unicode header 也是允许的，但不建议这样做。

### 更加复杂的POST请求
### POST一个多部分编码(Multipart-Encoded)的文件
### 响应状态码
### 响应头
### Cookie
### 重定向与请求历史
### 超时
### 错误与异常

## 高级用法

会话对象
请求与响应对象
准备的请求 （Prepared Request）
SSL 证书验证
客户端证书
CA 证书
响应体内容工作流
保持活动状态（持久连接）
流式上传
块编码请求
POST 多个分块编码的文件
事件挂钩
自定义身份验证
流式请求
代理
SOCKS
合规性
编码方式
HTTP动词
定制动词
响应头链接字段
传输适配器
示例: 指定的 SSL 版本
阻塞和非阻塞
Header 排序
超时（timeout）
