# [System Design](https://baike.baidu.com/item/系统设计)
    　　系统设计是根据系统分析的结果，运用系统科学的思想和方法，设计出能最大限度满足所要求的目标 (或目的) 的新系统的过程。
    进行系统设计时，必须把所要设计的对象系统和围绕该对象系统的环境共同考虑，前者称为内部系统，后者称为外部系统，它们之间存在着
    相互支持和相互制约的关系，内部系统和外部系统结合起来称作总体系统。因此，在系统设计时必须采用内部设计与外部设计相结合的原则，
    从总体系统的功能、输入、输出、环境、程序、人的因素、物的媒介各方面综合考虑，设计出整体最优的系统。进行系统设计应当采用分解、
    综合与反馈的工作方法。不论多大的复杂系统，首先要分解为若干子系统或要素，分解可从结构要素、功能要求、时间序列、空间配置等方面
    进行，并将其特征和性能标准化，综合成最优子系统，然后将最优子系统进行总体设计，从而得到最优系统。在这一过程中，从设计计划开始
    到设计出满意系统为止，都要进行分阶段及总体综合评价，并以此对各项工作进行修改和完善。整个设计阶段是一个综合性反馈过程。系统
    设计内容，包括确定系统功能、设计方针和方法，产生理想系统并作出草案，通过收集信息对草案作出修正产生可选设计方案，将系统分解为
    若干子系统，进行子系统和总系统的详细设计并进行评价，对系统方案进行论证并作出性能效果预测。

`验证URL有效性`
---
    在获取请求时需要做Urldecode处理，否则可能会验证不成功
    假设接收地址设置为：http://api.3dept.com/，将向该地址发送如下验证请求：
<strong>请求方式：GET</strong><br>
<strong>请求地址：</strong>http://api.3dept.com/?msg_signature=DFA845SDS&timestamp=1234&nonce=123412323&echostr=ENCRYPT_STR<br>
<table><thead><tr><th>参数</th><th>*</th><th>说明</th></tr></thead>
<tbody>
<tr>
<td>msg_signature</td>
<td>是</td>
<td>加密签名，msg_signature结合了token、请求中的timestamp、nonce参数、加密的消息体</td>
</tr>
<tr>
<td>timestamp</td>
<td>是</td>
<td>时间戳</td>
</tr>
<tr>
<td>nonce</td>
<td>是</td>
<td>随机数</td>
</tr>
<tr>
<td>echostr</td>
<td>是</td>
<td>加密字符串需要解密得到消息内容明文，解密后有random、msg_len、msg、receiveid字段，其中msg即为消息内容明文</td>
</tr>
</tbody></table>
    
`解密得到消息内容明文`
---
* 解密得到msg的过程
    *  对密文`Base64`解码
        > aes_msg=Base64_Decode(msg_encrypt)
    *  使用`AESKey`做`AES-256-CBC`解密
        > rand_msg=AES_Decrypt(aes_msg)
    *  去掉`rand_msg`头部的`16`个随机字节和`4`个字节`msg_len`，截取`msg_len`长度部分为`msg`，剩下的为尾部`receiveid`
    *  验证解密后的`receiveid`、`msg_len`。注意，`receiveid`在不同场景含义不同。
* 举例说明
    *  假设在服务商管理端为某个套件有如下配置参数
        > corpId = "wx5823bf96d3bd56c7"<br>
          token = "QDG6eK"<br>
          encodingAesKey = "jWmYm7qr5nMoAUwZRjGtBxmz3KA1tkAj3ykkR6q2B2C"<br>
    *  收到来自服务商管理端的回调为：
        > POST /cgi-bin/wxpush?msg_signature=477715d11cdb4164915debcba66cb864d751f3e6&timestamp=1409659813&nonce=1372623149 HTTP/1.1<br>
          Host: qy.weixin.qq.com<br>
          Content-Length: 613<br>
&lt;xml&gt;<br>
&lt;ToUserName&gt;&lt;![CDATA[ax5823bf96d3bd56c7]]&gt;&lt;/ToUserName&gt;<br>
&lt;Encrypt&gt;&lt;![CDATA[RypEvHKD8QQKFhvQ6QleEB4J58tiPdvo+rtK1I9qca6aM/wvqnLSV5zEPeusUiX5L5X/0lWfrf0QADHHhGd3QczcdCUpj911L3vg3W/sYYvuJTs3TUUkSUXxaccAS0qhxchrRYt66wiSpGLYL42aM6A8dTT+6k4aSknmPj48kzJs8qLjvd4Xgpue06DOdnLxAUHzM6+kDZ+HMZfJYuR+LtwGc2hgf5gsijff0ekUNXZiqATP7PF5mZxZ3Izoun1s4zG4LUMnvw2r+KqCKIw+3IQH03v+BCA9nMELNqbSf6tiWSrXJB3LAVGUcallcrw8V2t9EL4EhzJWrQUax5wLVMNS0+rUPA3k22Ncx4XXZS9o0MBH27Bo6BpNelZpS+/uh9KsNlY6bHCmJU9p8g7m3fVKn28H3KDYA5Pl/T8Z1ptDAVe0lXdQ2YoyyH2uyPIGHBZZIs2pDBS8R07+qN+E7Q==]]&gt;&lt;/Encrypt&gt;<br>
&lt;AgentID&gt;&lt;![CDATA[218]]&gt;&lt;/AgentID&gt;<br>
&lt;/xml&gt;

 第一步：准备相关参数
~~~
AESKey = Base64_Decode(EncodingAESKey + "=")
signature = "477715d11cdb4164915debcba66cb864d751f3e6";
timestamps = "1409659813";
nonce = "1372623149";
msg_encrypt = "RypEvHKD8QQKFhvQ6QleEB4J58tiPdvo+rtK1I9qca6aM/wvqnLSV5zEPeusUiX5L5X/0lWfrf0QADHHhGd3QczcdCUpj911L3vg3W/sYYvuJTs3TUUkSUXxaccAS0qhxchrRYt66wiSpGLYL42aM6A8dTT+6k4aSknmPj48kzJs8qLjvd4Xgpue06DOdnLxAUHzM6+kDZ+HMZfJYuR+LtwGc2hgf5gsijff0ekUNXZiqATP7PF5mZxZ3Izoun1s4zG4LUMnvw2r+KqCKIw+3IQH03v+BCA9nMELNqbSf6tiWSrXJB3LAVGUcallcrw8V2t9EL4EhzJWrQUax5wLVMNS0+rUPA3k22Ncx4XXZS9o0MBH27Bo6BpNelZpS+/uh9KsNlY6bHCmJU9p8g7m3fVKn28H3KDYA5Pl/T8Z1ptDAVe0lXdQ2YoyyH2uyPIGHBZZIs2pDBS8R07+qN+E7Q==";
~~~
 第二步：校验签名
*  token、timestamp、nonce、msg_encrypt 这四个参数按照字典序排序
~~~
"1372623149"
"1409659813"
"QDG6eK"
"RypEvHKD8QQKFhvQ6QleEB4J58tiPdvo+rtK1I9qca6aM/wvqnLSV5zEPeusUiX5L5X/0lWfrf0QADHHhGd3QczcdCUpj911L3vg3W/sYYvuJTs3TUUkSUXxaccAS0qhxchrRYt66wiSpGLYL42aM6A8dTT+6k4aSknmPj48kzJs8qLjvd4Xgpue06DOdnLxAUHzM6+kDZ+HMZfJYuR+LtwGc2hgf5gsijff0ekUNXZiqATP7PF5mZxZ3Izoun1s4zG4LUMnvw2r+KqCKIw+3IQH03v+BCA9nMELNqbSf6tiWSrXJB3LAVGUcallcrw8V2t9EL4EhzJWrQUax5wLVMNS0+rUPA3k22Ncx4XXZS9o0MBH27Bo6BpNelZpS+/uh9KsNlY6bHCmJU9p8g7m3fVKn28H3KDYA5Pl/T8Z1ptDAVe0lXdQ2YoyyH2uyPIGHBZZIs2pDBS8R07+qN+E7Q=="
~~~
*  拼接为一个字符串
~~~
sort_str = "13726231491409659813QDG6eKRypEvHKD8QQKFhvQ6QleEB4J58tiPdvo+rtK1I9qca6aM/wvqnLSV5zEPeusUiX5L5X/0lWfrf0QADHHhGd3QczcdCUpj911L3vg3W/sYYvuJTs3TUUkSUXxaccAS0qhxchrRYt66wiSpGLYL42aM6A8dTT+6k4aSknmPj48kzJs8qLjvd4Xgpue06DOdnLxAUHzM6+kDZ+HMZfJYuR+LtwGc2hgf5gsijff0ekUNXZiqATP7PF5mZxZ3Izoun1s4zG4LUMnvw2r+KqCKIw+3IQH03v+BCA9nMELNqbSf6tiWSrXJB3LAVGUcallcrw8V2t9EL4EhzJWrQUax5wLVMNS0+rUPA3k22Ncx4XXZS9o0MBH27Bo6BpNelZpS+/uh9KsNlY6bHCmJU9p8g7m3fVKn28H3KDYA5Pl/T8Z1ptDAVe0lXdQ2YoyyH2uyPIGHBZZIs2pDBS8R07+qN+E7Q=="
~~~
*  对该字符串进行sha1计算得到签名
~~~
signature = sha1(sort_str) = "477715d11cdb4164915debcba66cb864d751f3e6"
~~~
*  对比从URL得到的签名，发现两者一致，签名通过，说明没被篡改，是安全的

 第三步： 解密消息
*  对密文base64解码
~~~
aes_msg = base64_decode(msg_encrypt)
~~~
*  使用AESKey做AES解密（注意，不是EncodingAESKey）
~~~
rand_msg = aes_decrypt(aes_msg, AESKey)
~~~
*  去掉rand_msg头部的16个随机字节和4个字节的msg_len，截取msg_len长度的部分即为msg，剩下的为尾部的receiveid
~~~python
content = rand_msg[16:]  # 去掉前16随机字节
msg_len = str_to_uint(content[0:4]) # 取出4字节的msg_len
msg = content[4:msg_len+4] # 截取msg_len 长度的msg
receiveid = content[xml_len+4:] = "wx5823bf96d3bd56c7" # 剩余字节为receiveid
~~~
*  解密后得到明文为：
~~~xml
<xml>
   <ToUserName><![CDATA[wx5823bf96d3bd56c7]]></ToUserName>
   <FromUserName><![CDATA[mycreate]]></FromUserName>
   <CreateTime>1409659813</CreateTime>
   <MsgType><![CDATA[text]]></MsgType>
   <Content><![CDATA[hello]]></Content>
   <MsgId>4561255354251345929</MsgId>
   <AgentID>218</AgentID>
</xml>
~~~


`访问频率限制`
---

* 基础频率
    *  每企业调用单个/api不可超过2000次/分，30000次/小时
    *  企业每ip调用单个/api不可超过20000次/分，600000次/小时
    *  第三方应用提供商每ip调用单个/api不可超过40000次/分，1200000次/小时
* 上传图片频率
    *  每个企业，每天最多可以上传100张永久图片，接口详情参见`上传图片`
    
----

`全局错误码`
---
<table>
<thead><tr><th>错误码</th><th>错误说明</th><th>排查方法</th></tr></thead>
<tbody>
<tr>
<td>-1</td>
<td>系统繁忙</td>
<td>服务器暂不可用，建议稍候重试。建议重试次数不超过3次。</td>
</tr>
<tr>
<td>0</td>
<td>请求成功</td>
<td>接口调用成功</td>
</tr>
<tr>
<td>40001</td>
<td>不合法的secret参数</td>
<td>secret在应用详情/通讯录管理助手可查看</td>
</tr>
<tr>
<td>40003</td>
<td>无效的UserID</td>
<td><a href="#10649/错误码：40003">查看帮助</a></td>
</tr>
<tr>
<td>40004</td>
<td>不合法的媒体文件类型</td>
<td>不满足系统文件要求。参考：<a href="#10112">上传的媒体文件限制</a></td>
</tr>
</tbody></table>

----


