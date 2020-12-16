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
