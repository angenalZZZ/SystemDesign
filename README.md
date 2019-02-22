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
* 解密得到msg的过程：
    *  对密文`Base64`解码
        > aes_msg=Base64_Decode(msg_encrypt)
    *  使用`AESKey`做`AES-256-CBC`解密
        > rand_msg=AES_Decrypt(aes_msg)
    *  去掉`rand_msg`头部的`16`个随机字节和`4`个字节的`msg_len`，截取`msg_len`长度的部分即为`msg`，剩下的为尾部的`receiveid`
    *  验证解密后的`receiveid`、`msg_len`。注意，`receiveid`在不同场景含义不同。




