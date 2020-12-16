# ProcedureManagementDoc
流程管理系统API文档

## 1. 基本概念
+ **流程**：在这个语境中指的是一系列事先决定的事件的集合。事件有可能有先后顺序，也有可能无序发生或者同时发生。需要按照顺序完成的叫有序性流程，不需要按照顺序完成的叫做完备性流程。
+ **事件**：事件指的是在流程范围内需要完成的操作。
+ **数字证据**：包含操作痕迹、数字文档、数字协议，以及上述操作或数据的数字签名

## 2. 流程管理

* **所有API的`Method`均为`POST`**
* **所有API的`Content-Type`均为`application/json;charset=UTF-8`**
* **所有API的`返回类型`均为`application/json;charset=UTF-8`**

### 2.1 获取流程类型列表
+ URL:
    * `/api/procedure-type-list`
+ 参数：

| 参数名 | 解释 | 例 |
| --- | --- | --- |
| token | 机构的API账户标志 | 417fXkWTP85uE8LXZH9nqCZn3r3JjsTl |
| datetime| 调用时间（UTC时间） | 2020-01-22T09:12:43Z |
| signature | 机构的数字签名的Base64编码 | MIGIAkIB8HKnnrj5tMwEPVC... |

+ 签名字段：
    * token
    * datetime
+ 返回字段：

| 参数名 | 数据类型 | 说明 |
| --- | --- | --- |
| proTypeId | int | 流程类型的ID |
| proTypeName | string | 流程类型的名称。例如供应链金融中的“应收账款抵押”，“票据质押贷款”等 |
| proSteps | int | 流程总步骤数 |
| eventTypeIdArray | int数组 | 该流程包含的事件类型ID的数组 |
| comments | string | 备注或解释 |

### 2.2 获取流程实例列表
+ URL:
    * `/api/procedure-list`
+ 参数：

| 参数名 | 解释 | 例 |
| --- | --- | --- |
| proTypeId | 可选参数；用于找到所有属于proTypeId的流程实例 | 123 |
| from | 可选参数；用于找到从这个时间开始创建的流程实例 | 2020-01-22T09:12:43Z |
| to | 可选参数；用于找到到这个时间结束的流程实例 | 2020-01-22T09:12:43Z |
| token | 机构的API账户标志 | 417fXkWTP85uE8LXZH9nqCZn3r3JjsTl |
| datetime| 调用时间（UTC时间） | 2020-01-22T09:12:43Z |
| signature | 机构的数字签名的Base64编码 | MIGIAkIB8HKnnrj5tMwEPVC... |
+ 签名字段：
    * proTypeId
    * from
    * to
    * token
    * datetime
+ 返回字段：

| 参数名 | 数据类型 | 说明 |
| --- | --- | --- |
| proId | long | 流程实例的ID |
| proTypeId | int | 流程类型的ID |
| proName | string | 流程实例的名称。命名规则：融资方代号_流程编号_日期 |
| proStatus | byte | 流程当前状态-即实现的步骤数。例如：0=尚未启动 1=合同上传并完成数字签名 等。 |
| eventIdArray | long数组 | 该流程包含的事件ID的数组 |
| comments | string | 备注 |

## 3. 其它
### 3.1 签名方式
将所有**字段内容**，按照每个接口中“签名字段”的**顺序**进行字符串拼接，中间不需要任何分隔符。如果可选字段不设置，则用字符串“null”(不含双引号)填充之后使用SHA512-ECDSA算法进行签名。

**注意**：为防止重放攻击，如果服务端收到两条或两条以上的相同接口、相同datetime的请求，仅会执行第一条，其余的请求会被丢弃。

#### 3.1.1 密钥对生成方法
使用openssl工具，在linux的控制台执行：
```shell script
# 私钥生成
openssl ecparam -name secp521r1 -genkey -noout -out private.pem
# 公钥生成
openssl ec -in private.pem -pubout -out public.pem
```
#### 3.1.2 Java的依赖库（maven）
```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.67</version>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcpkix-jdk15on</artifactId>
    <version>1.67</version>
</dependency>
```

#### 3.1.3 加载私钥并签名
```java
String privateStr = "-----BEGIN EC PRIVATE KEY-----\n" +
"MIHcAgEBBEIB7HKA4LlfQAE3P/p7deFeZttQzp5qGmUSx2YRxnpSqkqP9U3hD5I4\n" +
"yuj6pfAr/KbfOHJ1Nd36e+vxEq5zR6tfvvagBwYFK4EEACOhgYkDgYYABABPDG3I\n" +
"gL4u1KBudkH0whHjInK/aF0BhGGY0GuNlxtlMyksMaNIehJWA7GAyAC0qBKYAKrn\n" +
"izGh+5Vbwo4vhpEp7AG5IDElnH2Dy9HvtYEDysWm5LhgRtPirTtIKrNT30vNBZXW\n" +
"cnaC2ZIasyltnvTwXPZypAAPTYfD0nqvqq4aPuRJAQ==\n" +
"-----END EC PRIVATE KEY-----";

// 加载BouncyCastle
Security.addProvider(new BouncyCastleProvider());

// 读取私钥pem，这里也可以选择从文件读取
PEMParser privatePemParser = new PEMParser(new StringReader(privateStr));
PEMKeyPair pemKeyPair = (PEMKeyPair) privatePemParser.readObject();

// 将私钥pem转为privateKey
JcaPEMKeyConverter converter = new JcaPEMKeyConverter();
KeyPair keyPair = converter.getKeyPair(pemKeyPair);
ECPrivateKey privateKey = (ECPrivateKey)keyPair.getPrivate();

// 签名
Signature signature = Signature.getInstance("SHA512withECDSA");
byte[] data = "这里是需要被签名的字符串".getBytes(StandardCharsets.UTF_8);
signature.initSign(privateKey);
signature.update(data);
byte[] result = signature.sign();

// 签名转为Base64
String signatureBase64 = Base64.getEncoder().encode(result);
```

### 3.2 返回值的结构
返回值都为`JSON`格式，如下：
```json
{"code": code, "msg": ...}
```

其中：
+ **code**：0表示成功，其它整数为错误
+ **msg**：按照每个接口“返回字段”中的数据结构返回