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
若想获取来自服务器的*原始套接字响应*，那么可以访问`r.raw`。前提是在初始请求中设置了`stream=True`。

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

例如，在前一个示例中没有指定 content-type:

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
- 在能判断内容长度的情况下，*header* 的*Content-Length*会被改写。

简单来说Requests不会基于定制header的具体情况改变自己的行为。只不过在最后的请求中，所有的header信息都会被传递进去。

注意: 所有的header值必须是string、bytestring 或者 unicode。尽管传递 unicode header 也是允许的，但不建议这样做。

### 更加复杂的POST请求
对于发送数据类似HTML表单的情况, 只需简单地传递一个字典给`data`参数。Requests在发出请求时会自动编码为表单形式：
```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}

>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```

`data`参数也可以是一个元组列表, 在表单中多个元素使用同一key的时候，这种方式尤其有效：
```python
>>> payload = (('key1', 'value1'), ('key1', 'value2'))
>>> r = requests.post('http://httpbin.org/post', data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key1": [
      "value1",
      "value2"
    ]
  },
  ...
}
```

如果发送的数据并非编码为表单形式的。例如如果传递一个string，那么数据会被直接发布出去。

例如，Github API v3 接受编码为 JSON 的 POST/PATCH 数据：
```python
>>> import json

>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, data=json.dumps(payload))
```

除了可以自行对dict进行编码，还可以使用json参数直接传递，然后它就会被自动编码。这是 2.4.2 版的新加功能：
```python
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, json=payload)
```

### POST一个多部分编码(Multipart-Encoded)的文件

Requests 使得上传多部分编码文件变得很简单：
```python
>>> url = 'http://httpbin.org/post'
>>> files = {'file': open('report.xls', 'rb')}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
```

可以显式地设置文件名，文件类型和请求头：
```python
>>> url = 'http://httpbin.org/post'
>>> files = {'file': ('report.xls', open('report.xls', 'rb'), 'application/vnd.ms-excel', {'Expires': '0'})}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
```

可以以文件形式发送字符串:
```python
>>> url = 'http://httpbin.org/post'
>>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "some,data,to,send\\nanother,row,to,send\\n"
  },
  ...
}
```

如果使用multipart/form-data请求发送一个非常大的文件，需要将请求做成数据流。默认下requests不支持, 但第三方包 requests-toolbelt 支持([toolbelt文档](https://toolbelt.rtfd.org/))。

**警告**  
对于文件的传输应该使用二进制模式(binary mode)打开文件, 因为Requests会识图补全Content-Length header, 这个值会被设置为文件字节数, 若使用文本模式(text mode)打开文件, 可能会出错.

### 响应状态码

检测响应状态码：
```python
>>> r = requests.get('http://httpbin.org/get')
>>> r.status_code
200
```

Requests还附带了一个内置的状态码查询对象方便判断状态：
```python
>>> r.status_code == requests.codes.ok
True
```

如果发送了一个返回请求为错误状态(*4XX* 客户端错误，或者*5XX*服务器错误响应)，可以通过`Response.raise_for_status()`来抛出异常：
```python
>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code
404
>>> bad_r.raise_for_status()
Traceback (most recent call last):
  File "requests/models.py", line 832, in raise_for_status
    raise http_error
requests.exceptions.HTTPError: 404 Client Error
```

若r的`status_code`是200，则调用`raise_for_status()`不会抛出错误：
```python
>>> r.raise_for_status()
None
```

### 响应头

服务器响应头以字典形式存储:
```python
>>> r.headers
{
    'content-encoding': 'gzip',
    'transfer-encoding': 'chunked',
    'connection': 'close',
    'server': 'nginx/1.0.4',
    'x-runtime': '148ms',
    'etag': '"e1ca502697e5c9317743dc078f67693f"',
    'content-type': 'application/json'
}
```

但是这个字典比较特殊, 仅为HTTP头部而生的。根据[RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)， HTTP头部是大小写不敏感的。

因此，对于响应头的访问可以忽略大小写:
```python
>>> r.headers['Content-Type']
'application/json'

>>> r.headers.get('content-type')
'application/json'
```

该数据格式的另一个特殊点: 服务器可以多次接受同一header，每次都使用不同的值。但Requests会将它们合并，这样它们就可以用一个映射表示出来，参见 [RFC 7230](http://tools.ietf.org/html/rfc7230#section-3.2):

>A recipient MAY combine multiple header fields with the same field name into one "field-name: field-value" pair, without changing the semantics of the message, by appending each subsequent field value to the combined field value in order, separated by a comma.

接收者可以合并多个相同名称的header栏位，把它们合为一个"field-name: field-value"配对，将每个后续的栏位值依次追加到合并的栏位值中，用逗号隔开即可，这样做不会改变信息的语义。

### Cookie
查看cookies:
```python
>>> url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)

>>> r.cookies['example_cookie_name']
'example_cookie_value'
```

指定cookies参数:
```python
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
```

Cookie的返回对象为RequestsCookieJar，其行为和字典类似，但接口更为完整，适合跨域名跨路径使用。还可以把Cookie Jar传到Requests中:
```python
>>> jar = requests.cookies.RequestsCookieJar()
>>> jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
>>> jar.set('gross_cookie', 'blech', domain='httpbin.org', path='/elsewhere')
>>> url = 'http://httpbin.org/cookies'
>>> r = requests.get(url, cookies=jar)
>>> r.text
'{"cookies": {"tasty_cookie": "yum"}}'
```

### 重定向与请求历史

默认情况下，除了HEAD, Requests会自动处理所有重定向。

可以使用响应对象的 history 方法来追踪重定向。

Response.history是一个Response对象的列表，为了完成请求而创建了这些对象。这个对象列表按照从 *最老 -> 最近* 的请求进行排序。

例如，Github 将所有的 HTTP 请求重定向到 HTTPS：
```python
>>> r = requests.get('http://github.com')

>>> r.url
'https://github.com/'

>>> r.status_code
200

>>> r.history
[<Response [301]>]
```

如果使用的是GET、OPTIONS、POST、PUT、PATCH 或者 DELETE，可以通过 allow_redirects 参数禁用重定向处理：
```python
>>> r = requests.get('http://github.com', allow_redirects=False)
>>> r.status_code
301
>>> r.history
[]
```

如果使用了HEAD，也可以启用重定向:
```python
>>> r = requests.head('http://github.com', allow_redirects=True)
>>> r.url
'https://github.com/'
>>> r.history
[<Response [301]>]
```

### 超时

可以指定 requests 在经过以 timeout 参数设定的秒数时间之后停止等待响应。基本上所有的生产代码都应该使用这一参数。如果不使用，程序可能会永远失去响应：
```python
>>> requests.get('http://github.com', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)
```

**注意**  
timeout 仅对连接过程有效，与响应体的下载无关。 timeout 并不是整个下载响应的时间限制，而是如果服务器在 timeout 秒内没有应答，将会引发一个异常（更精确地说，是在 timeout 秒内没有从基础套接字上接收到任何字节的数据时）If no timeout is specified explicitly, requests do not time out.

### 错误与异常
遇到网络问题（如：DNS 查询失败、拒绝连接等）时，Requests会抛出一个**ConnectionError**异常。

如果HTTP请求返回了不成功的状态码，`Response.raise_for_status()`会抛出一个**HTTPError**异常。

若请求超时，则抛出一个**Timeout**异常。

若请求超过了设定的最大重定向次数，则会抛出一个**TooManyRedirects**异常。

所有Requests显式抛出的异常都继承自**requests.exceptions.RequestException**。

### 参数小结

参数|适用请求|作用
:-:|:-:|:-
params|ALL|补全url里的参数
stream|ALL|通过`r.raw`获取原始套接字
headers|ALL|指定请求headers内容
data|POST|将字典编码为表单/直接传输string参数
json|POST|将字典编码为json格式传输
files|POST|字典形式接收文件句柄,传递文件
cookies|ALL|设置cookies
allow_redirects|ALL|true or false指定是否允许重定向
timeout|ALL|设定套接字无反应超时时间

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
