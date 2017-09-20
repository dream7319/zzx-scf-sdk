# zzx-scf-sdk
# 中子星供应链金融平台API文档

约定：中子星简称zzx，第三方平台简称3rd

## 一，接口格式
### 1.1  接口协议和格式
中子星和其它平台之间的通信均采用https协议：

协议  | 方法  | 格式 | 编码 |
---- |------|----  |------|
HTTPS|POST  |JSON|  UTF-8|	

### 1.2  接口地址
* 中子星接口地址：<br/>
生产地址：待定<br/>
测试地址：待定<br/>
* 第三方接口地址：xx

### 1.3 消息格式
#### 1.3.1 请求格式
若未做特殊说明，本文档所有请求接口，在消息头部均需传以下5个参数：

序号	| 名称  | 必填	| 类型	|参数说明|
---- |------|----  |------|-----|
1	|method	|Y	|String	|方法名称  |   (参与签名)|
2	|ver	|Y	|String	|协议接口版本号，|"1.0"   |(参与签名)|
3	|channelId|	Y	|String	|平台编号（由中子星统一分配)。| (参与签名)|
4	|signType	|Y	|String	|RSA2（SHA256WithRSA）|   (参与签名)|
5	|sign	|Y	|String	|签名，具体方法参见下面的安全签名机制   (不参与签名)|
6	|params	|Y	|String	|请求消息体，以JSON格式传输(消息体中每个非空字段(null:不参与,"":参与)均参与签名)|

<em><font color=red size=3>如果一个字段为null，Json中不包含该字段。</font></em>

#### 1.3.2 应答消息格式	
若未做特殊说明，本文档所有应答接口头部均需传以下5个参数：

序号	| 名称  | 必填	| 类型	|参数说明|
---- |------|----  |------|-----|
1	|statusCode	|Y	|int	|"消息是否成功接受并处理结果标记，statusCode = 200时表示成功处理, statusCode != 200时表示处理失败" (不参与签名)|
2	|errMsg	|N	|String	|"失败原因， 仅当statusCode != 200时有效"(不参与签名)|
3	|method	|Y	|String	|方法名称，同请求   (参与签名)|
4	|ver	|Y	|String	|协议接口版本号，"1.0"，同请求。   (参与签名)|
5	|channelId|	Y	|String	|"平台编号（由中子星统一分配)。此字段为必选字段，内容即为分配给该平台的ID|
6	|signType	|Y|	String	|RSA2（SHA256WithRSA）   (参与签名)|
7	|sign	|Y	|String	|RSA加密签名，见安全签名机制 (不参与签名)|
8	|params	|Y	|String|	应答消息体，以JSON格式传输。 (消息体中每个非空字段均参与签名(null:不参与,"":参与),如果没有数据,此字段可以不传递)|

#### 1.3.3 请求和应答样例	

请求
```
{"method":"upsertCustomer","ver":"1.0","channelId":"99","signType":"RSA2","params":{"idNo":"21010219910405503X","userName":"测试姓名","contactTel":"13800138000","registerTime":"2016-09-21 10:27:34"},"sign":"..."}
```
应答
```
{"method":"upsertCustomer","ver":"1.0","channelId":"99","params":{"idNo":"21010219910405503X"},"signType":"RSA2","sign":"略","statusCode":200}
```


### 1.4 签名	

#### 1.4.1 说明
* 在RSA签名时，需要私钥和公钥一起参与签名。私钥与公钥可以自行通过OPENSSL来生成。<br>
    参考RSA生成的公钥和私钥：<br>
    1、openssl genrsa -out rsa_private_key.pem 1024<br>
    2、openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem<br>
    3、openssl pkcs8 -topk8 -in rsa_private_key.pem -out pkcs8_rsa_private_key.pem -nocrypt<br>
    即：rsa_public_key.pem公钥、pkcs8_rsa_private_key.pem私钥<br>
* 中子星平台把生成出的公钥与第三方的公钥做交换。
* 在签名时，中子星平台使用自己的私钥产生签名字符串。收到通知或应答时，使用第三方的公钥进行校验。 

#### 1.4.2 请求时产生签名
* 在发送请求前，先根据请求的消息内容，产生待签名字符串后，把待签名字符串与自己的私钥进行RSA加密得到签名结果字符串。

#### 1.4.3 得到通知或应答时验证签名  
* 得到通知或应答后，根据收到的内容，按照规则产生待签名字符串，然后把待签名字符串、公钥、返回参数中的签名结果参数sign的值三者一同放入RSA签名函数中进行非对称的签名运算，来判断签名是否正确。

#### 1.4.4 待签名字符串的产生方法 
* 直接把请求数据中的所有元素(除sign本身)按照“key值=value值”的格式拼接起来，并且把这些拼接以后的元素以“&”字符再连接起来（顺序按首字母升序排列，值为空的不参与签名，params字符串里面的内容不需要排序），所连接起来的这段字符串即是请求时的待签名字符串。 

* 例子

待签名字符串：
```
channelId=2&method=getLoanDetailInfo&params={"loanDate":"2016-12-09","commissions":60,"loanAmount":2000,"balance":2080.53,"refunds":[{"periodNumber":1,"dueDate":"20170108","dueAmount":693.51,"status":3},{"periodNumber":2,"dueDate":"20170207","dueAmount":693.51,"status":3},{"periodNumber":3,"dueDate":"20170309","dueAmount":693.51,"status":3}]}&signType=RSA2&ver=1.0
```

签名后
```
{"method":"getLoanDetailInfo","ver":"1.0","channelId":"2","signType":"RSA2","sign":"AiNnysWeGcCmxAEnONxJhlvImIJgnlo6qAqf5+EgHmCSpVTGfydcR6bW0bdAve4Yj3bRm7hx36/2NtnbQiZRm/PD8S0JIt6lTp0LRzFU0ZcCy5RMSc6iM08Cm6KpiljTjkohCsOy0V6Ux2Hnu3xSSG3gLCBDAUVqWLPeADlFn8s\u003d","params":{"loanDate":"2016-12-09","commissions":60.0,"loanAmount":2000.0,"balance":2080.53,"refunds":[{"periodNumber":1,"dueDate":"20170108","dueAmount":693.51,"status":3},{"periodNumber":2,"dueDate":"20170207","dueAmount":693.51,"status":3},{"periodNumber":3,"dueDate":"20170309","dueAmount":693.51,"status":3}]}}
```

## 三, 接口说明

### 4.1, 申请贷款
用多个订单来申请融资，中子星会根据风控标准筛选出合适的订单
方法名：loanApply

调用方向：3rd->zzx

参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
amount | 贷款申请金额（销售合同金额合计）单位：元  | double |Y|Y|
productId | 金融产品ID(中子星提供)  | string(128)|Y|N|
orders | 订单集合(list)  | list|Y|N|
*  金融产品ID确定了这次贷款申请的利率，期限等金融要素.对接时中子星会给对应的ID信息

* order实体说明

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
orderId | 编号  | string(128)|Y|Y|
name | 姓名  | string(128)|Y|N|
organizationId | 单位编号  | string(128)|Y|N|
organization | 单位名称  | string(128)|Y|N|
mobile | 手机号  | string(11) |Y|N|
cardno | 身份证号  | String(19) |Y|N|
divideRate | 分成比例 | float |Y|N|
level | 套餐档次 | string|Y|Y|
orderDate | 订购时间 |string|Y|N|
packageDuration|协议期(套餐时长)| string|Y|N|
type|机器型号| string|Y|N|



返回值：
* statusCode = 200即为成功，非200看errMsg字段
* statusCode = 200 时 在响应体的params字段里面有 ｛loanId : xxxx},返回这次贷款申请的ID ,string类型

#### 4.2, 上传申请单位资料

方法名：uploadAttachment

调用方向：3rd->zzx

参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
id | 贷款编号(中子星返回的贷款编号)  | string(128)|Y|Y|
filename | 文件名称(aa.png)  | string(200)|Y|N|
data | 内容 base64  | string |Y|N|

返回值：

* statusCode = 200即为成功，非200看errMsg字段

#### 4.3, 确认贷款方案

方法名：loanApplySubmit
贷款申请的补充资料提交完毕后，确认申请

调用方向：3rd->zzx


参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
loanId | 贷款编号  | string(128)|Y|Y|

返回值：
* statusCode = 200即为成功，非200看errMsg字段


### 5, 贷款申请结果通知
中子星请求平台

方法名：loanApplyResultNotify

调用方向：zzx->3rd

参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
loanId | 贷款编号  | string(128)|Y|Y|
result | 贷款是否批准（0，1)1,代表贷款批准 | int|Y|Y|
commissions | 费用  | double|Y|Y|
reason | 档贷款result＝0时，有拒绝贷款原因  | string(128)|Y|Y|
loanAmount | 批准的贷款金额合计，仅当result=1有效  | double|Y|Y|
loanTerm | 批准的贷款期限，仅当result=1有效  | int|Y|Y|
paymentOption |还款方式 1.到期一次性   | int|Y|Y|
orders | 订单编号集合(list)  | string|Y|Y|

orders时一个数组，里面每一个order参数如下：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
sourceOrderId | 来源平台中的订单编号，仅当result=1有效  | string(128)|Y|Y|
loanAmount | 该订单审批通过的贷款金额，仅当result=1有效。为0是，表示该订单未审核通过|double|Y|Y|

返回值：
* statusCode = 200即为成功，非200看errMsg字段

### 6, 贷款申请结果确认
平台请求中子星

方法名：loanContractConfirm

调用方向：3rd->zzx

参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
loanId | 贷款编号  | string(128)|Y|Y|
confirmation | 确认结果1：同意   2：取消   3：过期 | int|Y|Y|

返回值：
* statusCode = 200即为成功，非200看errMsg字段

### 7，贷款放款完成通知
中子星收到平台的确认消息即开始放款操作

方法名：paymentNotify

调用方向：zzx-3rd

参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
loanId | 贷款编号  | string(128)|Y|Y|
date | 发放日期(YYYYMMDD) | string|Y|Y|
amount | 发放金额 | double|Y|Y|
account | 放款帐号(list) | string|Y|Y|
refunds | 还款计划集合(list) | string|Y|Y|

* account说明

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
accountName | 放款账户  | string(128)|Y|Y|
bankCard | 账号 | string|Y|Y|
bankBranch | 开户支行 | string|Y|Y|

* refunds说明

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
periodNumber | 还款期数序号  | int|Y|Y|
dueDate | 应还款日期(YYYYMMDD) | string|Y|Y|
dueCapital | 应还本金 | double|Y|Y|
dueInterest | 应还利息 | double|Y|Y|
dueAmount | 应还金额总和 | double|Y|Y|

返回值：
* statusCode = 200即为成功，非200看errMsg字段

### 8，主动还款通知
    第三方平台财务汇款后通知

方法名：refundNotify

调用方向：3rd->zzx

参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
refundType | 还款类型：1：到期正常，还款2：提前还款，3：逾期还款  | int|Y|Y|
loanId | 贷款编号 | string|Y|Y|
amount | 还款总金额 | double|Y|Y|
periodNumber | 还款期数序号 | int|Y|Y|

返回值：
* statusCode = 200即为成功，非200看errMsg字段

### 9，还款结果通知

方法名：refundResultNotify

调用方向：zzx->3rd
参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
result | 还款扣款结果  1：成功    0：失败  | int|Y|Y|
reason | 还款，仅当result=0时有效  | string|Y|Y|
date | 还款日期 | string|Y|Y|
periodNumber | 还款期数序号 | int|Y|Y|
loanId | 贷款编号 | string|Y|Y|
amount | 还款总金额=本金+利息+手续费+罚息 | double|Y|Y|
refundCapital | 还款本金，仅当result=1时有效 | double|Y|Y|
refundInterest | 还款利息，仅当result=1时有效 | double|Y|Y|
refundCommission | 还款手续费，仅当result=1时有效 | double|Y|Y|
refundDefaultInterest | 还款罚息，仅当result=1时有效 | double|Y|Y|
refundFlag | 此期是否还款完毕，仅当result=1时有效1：已还完完毕    2：未还款完毕 | int|Y|Y|
refundType | 还款类型，1：到期还款，2：提前还款，3：追偿还款 | int|Y|Y|

返回值：
* statusCode = 200即为成功，非200看errMsg字段

### 10，客户还款提醒

方法名：customerRefundNotify

调用方向：zzx->3rd

参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
loanId | 贷款编号  | string|Y|Y|
refundCapital | 当期应还本金  | double|Y|Y|
refundInterest | 当期应还利息 | double|Y|Y|
refundCommission | 当期应还手续费  | double|Y|Y|
amount | 还款总金额=本金+利息+手续费+罚息 | double|Y|Y|
periodNumber | 还款期数  | int|Y|Y|
refundDefaultInterest | 还款罚息 | double|Y|Y|
overdueDays | 逾期天数   | int|Y|Y|

返回值：
* statusCode = 200即为成功，非200看errMsg字段


