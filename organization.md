方法名：loanApply

调用方向：3rd->zzx

参数列表：

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
amount | 贷款申请金额（销售合同金额合计）单位：元  | double |Y|Y|
productId | 金融产品ID  | string(128)|Y|N|
orders | 订单集合(list)  | list|Y|N|
*  金融产品ID确定了这次贷款申请的利率，期限等金融要素.对接时中子星会给对应的ID信息

* order实体说明

参数名 | 说明 | 类型 | 必须 | 唯一|
----|------|----|------|----|
orderId | 编号  | string(128)|Y|Y|
name | 姓名  | string(128)|Y|N|
organizationId | 单位编号  | string(128)|Y|N|
organization | 单位名称  | string(128)|Y|N|
mobile | 手机号  | 11 |Y|N|
cardno | 身份证号  | int|Y|N|
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
fileSubject | loanApply-单位编号  | string(128)|Y|N|
filename | 文件名称(aa.png)  | string(200)|Y|N|
data | 内容 base64  | string |Y|N|
* fileSubject的格式为：[loanApply-单位编号]，比如单位是: 111,则是loanApply-111

返回值：

* statusCode = 200即为成功，非200看errMsg字段
