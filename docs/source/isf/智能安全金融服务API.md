# 2.智能安全金融服务API

### 2.1.简介

欢迎使用“星火·链网”-星火产融服务中台。
星火产融服务中台是面向产融客户，可以提供跨地域业务协作、多品类金融产品融合、多样化产融公共服务，通过将多个企业/机构连接成一个网络，建立起多方参与的新型数字化产融协作的统一服务中台。 
星火产融服务平台提供以下接口服务

| **服务类型**       | **描述**                                                     |
| ------------------ | ------------------------------------------------------------ |
| 账户服务           | 用于用户账户的操作，包含资金账簿开立、小额鉴权、入金、出金、转账、流水、电子回单下载，银行账户添加、资金账簿与银行账户关联等操作。 |
| 清结算服务         | 用于产融清结算信息操作，包含清结算信息预览，清算支付，清算调账等操作 |
| 融资服务           | 用于产融融资信息操作，包含融资客户信息推送，融资成本试算申请，融资申请，融资查询，查询等操作 |
| 贸易真实性验真服务 | 用于产融的贸易信息验真操作，包含发票OCR,验真等操作           |

### 2.2.接入方式

第一步：创建星火账户及企业认证；安装星火通并完成星火账户创建及企业认证。具体操作指引详见**浏览器插件钱包**章节。                    

第二步：登录“超级节点星火服务平台-星火产融服务ISF-产融平台接入”功能申请准入，审批后获取apiKey及apiSecret。
第三步：接入测试：接入测试环境进行测试联调。
第四步：应用对接：用生产环境申请的apiKey/apiSecret接入正式环境。

### 2.3.对接说明

#### API认证机制

所有API的安全认证一律采用accessToken认证，服务端根据生成算法验证认证字符串的正确性。accessToken需要应用方通过apiKey和apiSecret调用相关平台接口获得，具有时效性，有效时间为2小时，过期后需要重新获取否则会调用接口失败，建议妥善保存并管理。

#### API对接环境

- 测试环境域名为：http://test-isfopenapi.bitfactory.cn/
- 正式环境域名为：https://isfopenapi.bitfactory.cn/

#### API接口调用方式

| 规则     | 说明  |
| -------- | ----- |
| 通信协议 | HTTPS |
| 编码规则 | UTF-8 |
| 参数格式 | JSON  |

#### API接口公共参数说明

##### 公共请求参数

| 参数      | 类型       | 是否必填 | 说明   |
| --------- | :--------- | -------- | ------ |
| requestNo | String(32) | 是       | 请求号 |

##### 公共header携带

| key           | value                  |
| ------------- | ---------------------- |
| Authorization | Bearer ${access_token} |

access_token的获取见 [获取access_token](#获取access_token)

##### 公共响应参数

| 参数    | 类型   | 是否必填 | 说明               |
| ------- | :----- | -------- | ------------------ |
| code    | String | 是       | 响应码：00000-成功 |
| message | String | 否       | 错误码编码         |
| data    | String | 是       | 具体的响应内容     |

##### 响应成功报文

```json
{
    "data": {},
    "code": "00000",
    "message": "SUCCESS"
}
```

##### 响应异常报文

```json
{
    "code": "A0230",
    "message": "token invalid  or expired."
}
```



### 2.4.接口权限

##### 获取access_token

**接口地址**:`/if-ops/oauth/token?api_secret=${apiSecret}&grant_type=open_api&api_key=${apiKey}`

**请求方式**:`POST`

**请求数据类型**:`application/json`

**响应数据类型**:`application/json`

**接口描述**:`获取access_token`

**请求示例**:


```text
if-ops/oauth/token?api_secret=mxw6qx96vpzuif00fyt0tdz1jf52kfbvd5b3nk06&grant_type=open_api&api_key=did:bid:efe14gUdkN9ckUxhm5zBWj5uvoK57CKK
```

**header携带**

| key           | value                          |
| ------------- | ------------------------------ |
| Authorization | Basic aWYtb3Blbi1hcGk6MTIzNDU2 |

**请求参数**:


| 参数名称  | 参数说明                      | 是否必须 |
| --------- | ----------------------------- | -------- |
| apiSecret | 星火服务平台获取到的APISecret | true     |
| apiKey    | 星火服务平台获取到的APIKey    | true     |

**响应状态**:


| 状态码 | 说明         | schema |
| ------ | ------------ | ------ |
| 200    | OK           |        |
| 201    | Created      |        |
| 401    | Unauthorized |        |
| 403    | Forbidden    |        |
| 404    | Not Found    |        |


**响应参数**:


| 参数名称     | 参数说明                                                     | 类型    | schema |
| ------------ | ------------------------------------------------------------ | ------- | ------ |
| access_token | 业务接口所需token                                            | string  |        |
| expires_in   | access_token有效期，单位：秒                                 | integer |        |
| platformBid  | 当前apiKey对应的产融平台bid                                  | string  |        |
| serviceList  | 当前可用服务列表：ACCOUNT-账户服务；SETTLEMENT-清结算服务；FINANCE-融资服务；VOTI-贸易真实性验真服务 | array   |        |
| platformName | 当前apiKey对应的产融平台名称                                 | string  |        |


**响应示例**:

```javascript
{
    "access_token": "",
    "token_type": "bearer",
    "refresh_token": "",
    "expires_in": 7200,
    "scope": "all",
    "platformBid": "",
    "serviceList": [
        "ACCOUNT",
        "SETTLEMENT",
        "FINANCE",
        "VOTI"
    ],
    "platformName": "",
    "jti": ""
}
```



### 2.5.MD5签名示例代码

1、示例代码

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.serializer.SerializerFeature;
import org.apache.commons.codec.digest.DigestUtils;
import java.io.UnsupportedEncodingException;

public class TestSign {

    public static void main(String[] args) {
        // 参数对应的json串
        String params = "";
        //加密对应的key
        String key = "";
        //将参数进行处理，处理字段排序，转义符
        JSONObject jsonObject = JSONObject.parseObject(params, JSONObject.class);
        String jsonString = JSON.toJSONString(jsonObject, SerializerFeature.MapSortField);
        JSONObject jsonObject2 = JSONObject.parseObject(jsonString, JSONObject.class, Feature.OrderedField);
        //将入参对应json串和key进行拼接
        String originStr = JSONObject.toJSONString(jsonObject2) + "&key=" + key;
        byte [] originData;
        try {
            originData =  originStr.getBytes("utf-8");
        } catch (RuntimeException | UnsupportedEncodingException e) {
            throw new RuntimeException(e);
        }
        //MD5进行加密
        String signStr = DigestUtils.md5Hex(originData);
        System.out.println("signStr = "+signStr);
    }
}
```

2、依赖包

```xml
           <dependency>
                <artifactId>fastjson</artifactId>
                <groupId>com.alibaba</groupId>
                <version>1.2.83</version>
            </dependency>
            <dependency>
                <groupId>commons-codec</groupId>
                <artifactId>commons-codec</artifactId>
                <version>1.15</version>
            </dependency>
```

### 2.6.贸易真实性验真服务

**简介**:用于产融的贸易信息验真操作，包含发票OCR,验真等操作。



#### 贸易真实性验真服务：发票操作


##### 发票文件批量识别

**接口地址**:`/if-voti/api/voti/v1/invoice/function/batch-ocr`


**请求方式**:`POST`

**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>仅为ocr模型图片识别，不含发票与状态，发票各字段均有可能为空;图片正常识别完成，但无法识别出发票时，对应返回空数组</p>

**请求示例**:


```javascript
{
  "annexLabelList": [],
  "requestNo": ""
}
```


**请求参数**:


| 参数名称                   | 参数说明                                                     | 请求类型 | 是否必须 | 数据类型         | schema           |
| -------------------------- | ------------------------------------------------------------ | -------- | -------- | ---------------- | ---------------- |
| form                       | form                                                         | body     | true     | 发票识别请求对象 | 发票识别请求对象 |
| &emsp;&emsp;annexLabelList | 识别列表-文件在产融平台唯一标记，上限10个，需先通过”文件上传“接口，得到文件标识 |          | false    | array            | string           |
| &emsp;&emsp;requestNo      | 请求号                                                       |          | true     | string           |                  |


**响应状态**:


| 状态码 | 说明         | schema |
| ------ | ------------ | ------ |
| 200    | OK           |        |
| 201    | Created      |        |
| 401    | Unauthorized |        |
| 403    | Forbidden    |        |
| 404    | Not Found    |        |

**响应参数**:

返回Map<String, List>格式，key为对应的文件标识，value为该文件的识别结果。

List元素参数名称释义：

| 参数名称        | 参数说明                                     | 类型   | schema |
| --------------- | -------------------------------------------- | ------ | ------ |
| invoiceType     | 发票类型：[发票类型code值](#发票类型)        | string |        |
| invoiceCode     | 发票代码                                     | string |        |
| invoiceNumber   | 发票号码                                     | string |        |
| billingDate     | 开票日期 格式yyyy年MM月dd日                  | string |        |
| amountTax       | 不含税金额， 单位：元                        | string |        |
| totalAmount     | 总金额， 单位：元                            | string |        |
| tax             | 税额， 单位：元                              | string |        |
| checkCode       | 校验码                                       | string |        |
| salesName       | 销方纳税人名称                               | string |        |
| salesTaxNo      | 销方纳税人识别号                             | string |        |
| purchaserName   | 购方纳税人名称                               | string |        |
| purchaserTaxNo  | 购方纳税人识别号                             | string |        |
| companySeal     | 是否有公司印章 (0:没有; 1:有)                | string |        |
| formType        | 显示发票是第几联                             | string |        |
| formName        | 发票联次                                     | string |        |
| kind            | 发票消费类型                                 | string |        |
| ciphertext      | 密码区,四行密码,每行以逗号隔开               | string |        |
| transitMark     | 通行费标志                                   | string |        |
| refinedOil      | 成品油标志                                   | string |        |
| machineCode     | 机器编号                                     | string |        |
| travelTax       | 车船税                                       | string |        |
| receiptor       | 收款人                                       | string |        |
| reviewer        | 复核人                                       | string |        |
| issuer          | 开票人                                       | string |        |
| province        | 省                                           | string |        |
| city            | 市                                           | string |        |
| serviceName     | 服务类型                                     | string |        |
| remark          | 备注                                         | string |        |
| itemNames       | 品名，每个以逗号隔开                         | string |        |
| agentMark       | 是否代开 (0:没有; 1:有)                      | string |        |
| acquisitionMark | 是否收购 (0:没有; 1:有)                      | string |        |
| category        | 种类，oil 表示是加油票(机打发票，普票，卷票) | string |        |
| highwayFlag     | 高速标志 (0:没有; 1:有)                      | string |        |

**响应示例**:

```javascript
{
    "对应的文件标识": [
        {
            "invoiceType": "",
            "invoiceCode": "",
            "invoiceNumber": "",
            "billingDate": "",
            "amountTax": "",
            "totalAmount": "",
            "tax": "",
            "checkCode": "",
            "salesName": "",
            "salesTaxNo": "",
            "purchaserName": "",
            "purchaserTaxNo": "",
            "companySeal": "",
            "formType": "",
            "formName": "",
            "kind": "",
            "ciphertext": "",
            "transitMark": "",
            "refinedOil": "",
            "machineCode": "",
            "travelTax": "",
            "receiptor": "",
            "reviewer": "",
            "issuer": "",
            "province": "",
            "city": "",
            "serviceName": "",
            "remark": "",
            "itemNames": "",
            "agentMark": "",
            "acquisitionMark": "",
            "category": "",
            "highwayFlag": ""
        }
    ]
}
```


##### 发票验真

**接口地址**:`/if-voti/api/voti/v1/invoice/function/recognize-check`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`application/json`

**接口描述**:<p>发票验真</p>

**请求示例**:


```javascript
{
  "infoForms": [
    {
      "amountNoTax": 0,
      "checkCode": "",
      "invoiceCode": "",
      "invoiceDate": "",
      "invoiceNum": "",
      "invoiceType": ""
    }
  ],
  "requestNo": ""
}
```


**请求参数**:


| 参数名称                            | 参数说明                                  | 请求类型 | 是否必须 | 数据类型         | schema           |
| ----------------------------------- | ----------------------------------------- | -------- | -------- | ---------------- | ---------------- |
| form                                | form                                      | body     | true     | 发票识别响应对象 | 发票识别响应对象 |
| &emsp;&emsp;infoForms               | 发票查验请求list, 上限10个                |          | false    | array            | 发票查询请求对象 |
| &emsp;&emsp;&emsp;&emsp;amountNoTax | 发票不含税金额（元）:专票不能为空         |          | false    | number           |                  |
| &emsp;&emsp;&emsp;&emsp;checkCode   | 校验码后6位：普票不能为空                 |          | false    | string           |                  |
| &emsp;&emsp;&emsp;&emsp;invoiceCode | 发票代码                                  |          | true     | string           |                  |
| &emsp;&emsp;&emsp;&emsp;invoiceDate | 开票日期（eg：2020-01-01）                |          | true     | string           |                  |
| &emsp;&emsp;&emsp;&emsp;invoiceNum  | 发票号码                                  |          | true     | string           |                  |
| &emsp;&emsp;&emsp;&emsp;invoiceType | 发票类型,可用值:[发票类型枚举](#发票类型) |          | true     | string           |                  |
| &emsp;&emsp;requestNo               | 请求号                                    |          | true     | string           |                  |

**响应状态**:


| 状态码 | 说明         | schema       |
| ------ | ------------ | ------------ |
| 200    | OK           | 发票查验结果 |
| 201    | Created      |              |
| 401    | Unauthorized |              |
| 403    | Forbidden    |              |
| 404    | Not Found    |              |


**响应参数**:


| 参数名称            | 参数说明                                                    | 类型             | schema           |
| ------------------- | ----------------------------------------------------------- | ---------------- | ---------------- |
| invoiceMap          | 发票查验结果Map，入参字段与发票真实信息均一致时返回验真成功 | 单张发票查验结果 | 单张发票查验结果 |
| &emsp;&emsp;failMsg | 查验失败时返回查验失败的原因                                | string           |                  |
| &emsp;&emsp;status  | 发票查验结果,可用值:INIT,PROCESS,REJECTED,SUCCESS           | string           |                  |


**响应示例**:

```javascript
{
	"invoiceMap": {
		"additionalProperties1": {
			"failMsg": "",
			"status": ""
		}
	}
}
```

#### 字典表

##### 发票类型

| 发票类型                 | 枚举               | code |
| ------------------------ | ------------------ | ---- |
| 增值税专用发票           | SPECIAL_TICKET     | 01   |
| 货运运输业增值税专用发票 | TRANSPORT_TICKET   | 02   |
| 机动车销售统一发票       | VEHICLE            | 03   |
| 增值税普通发票           | POPULAR_VOTES      | 04   |
| 增值税普通发票（电子）   | E_TICHET           | 10   |
| 增值税普通发票（卷式）   | VOLUNE_TICKET      | 11   |
| 增值税普通发票（通行费） | VOLUNE_TICKET_TOLL | 14   |
| 二手车发票               | USERD_CAR          | 15   |

### 2.7.账户服务

**简介**:  用于用户账户的操作，包含资金账簿开立、小额鉴权、入金、出金、转账、流水、电子回单下载，银行账户添加、资金账簿与银行账户关联等操作。

#### 资金流向说明

1. 充值：资金流向为：银行卡->子账户
2. 提现：资金流向为：子账户->银行卡
3. 转账：资金流向为：子账户->子账户

#### 组合使用接口说明

##### 充值

1. 充值入参中的子账户区块链地址，与银行卡区块链地址，应先通过“账户绑定银行卡”接口（/if-account/api/account/v1/bindBankCard），完成绑定关系

2. 调用充值之前，需确保银行卡中有余额

3. 充值blob生成接口，返回的待签名数据需使用子账户对应的企业主体bid私钥进行数据签名

4. 提交签名结果接口的requestNo，需与blob生成接口保持一致

   <img src="../_static/images/image10.png"/>

##### 提现

1. 提现入参中的子账户区块链地址，与银行卡区块链地址，应先通过“账户绑定银行卡”接口（/if-account/api/account/v1/bindBankCard），完成绑定关系

2. 调用提现之前，需确保子账户中有余额

3. 提现blob生成接口，返回的待签名数据需使用子账户对应的企业主体bid私钥进行数据签名

4. 提交签名结果接口的requestNo，需与blob生成接口保持一致

5. 提现结果可通过“交易状态查询接口”（/if-account/api/account/v1/status/transaction-status），对交易结果进行查询

   <img src="../_static/images/image11.png"/>

##### 转账

1. 调用转账之前，需确保转出方子账户中有余额

2. 转账blob生成接口，返回的待签名数据需使用转出方子账户对应的企业主体bid私钥进行数据签名

3. 提交签名结果接口的requestNo，需与blob生成接口保持一致

   <img src="../_static/images/image12.png"/>

#### 账户服务：业务操作

##### 开立子账户


**接口地址**:`/if-account/api/account/v1/register`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>开立子账户</p>

**请求示例**:


```javascript
{
  "bankCode": "",
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "did:bid:efSwvJcRMyAc2P4q8S8avZcrAkFVkEyP",
  "payAccountIdentiNum": "",
  "payAccountName": "",
  "payAccountParams": "",
  "payAccountProperty": "",
  "payAccountType": "",
  "realAddress": "",
  "remark": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                        | 参数说明               | 请求类型 | 是否必须 | 数据类型                 | schema                   |
| ------------------------------- | ---------------------- | -------- | -------- | ------------------------ | ------------------------ |
| input                           | input                  | body     | true     | AccountRegisterInputForm | AccountRegisterInputForm |
| &emsp;&emsp;bankCode            | 银行编码               |          | true     | string                   |                          |
| &emsp;&emsp;merchantId          | 商户号                 |          | true     | string                   |                          |
| &emsp;&emsp;nonceStr            | 随机字符串             |          | true     | string                   |                          |
| &emsp;&emsp;payAccountAddress   | 支付账户bid地址        |          | true     | string                   |                          |
| &emsp;&emsp;payAccountIdentiNum | 证件号码               |          | true     | string                   |                          |
| &emsp;&emsp;payAccountName      | 名称                   |          | true     | string                   |                          |
| &emsp;&emsp;payAccountParams    | 支付拓展字段           |          | false    | string                   |                          |
| &emsp;&emsp;payAccountProperty  | 开户类型 0=企业 1=个人 |          | false    | string                   |                          |
| &emsp;&emsp;payAccountType      | 账户类型               |          | true     | string                   |                          |
| &emsp;&emsp;realAddress         | 实名账户地址           |          | true     | string                   |                          |
| &emsp;&emsp;remark              | 备注                   |          | false    | string                   |                          |
| &emsp;&emsp;requestNo           | 请求号                 |          | true     | string                   |                          |
| &emsp;&emsp;sign                | 签名串                 |          | true     | string                   |                          |
| &emsp;&emsp;signType            | 签名类型               |          | true     | string                   |                          |


**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | Result«AccountRegisterV0» |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称                      | 参数说明                     | 类型              | schema            |
| ----------------------------- | ---------------------------- | ----------------- | ----------------- |
| code                          |                              | string            |                   |
| data                          |                              | AccountRegisterV0 | AccountRegisterV0 |
| &emsp;&emsp;bankAccountNum    | 银⾏虚拟账号                 | string            |                   |
| &emsp;&emsp;bankInterNum      | 跨⾏收款账号（华夏银⾏才返回 | string            |                   |
| &emsp;&emsp;code              | 返回状态                     | string(byte)      |                   |
| &emsp;&emsp;payAccountAddress | bid- ⼦账户区块链地址        | string            |                   |
| message                       |                              | string            |                   |
| total                         |                              | integer(int32)    | integer(int32)    |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"bankAccountNum": "",
		"bankInterNum": "",
		"code": "",
		"payAccountAddress": ""
	},
	"message": "",
	"total": 0
}
```

##### 添加银行卡


**接口地址**:`/if-account/api/account/v1/addBankCard`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>添加银行卡</p>

**请求示例**:


```javascript
{
  "bankAccountName": "星火",
  "bankCardNum": 601106020000001093,
  "bankCode": "",
  "cnapsCode": 313146000035,
  "realAddress": "",
  "requestNo": 187542789578293248
}
```


**请求参数**:


| 参数名称                    | 参数说明       | 请求类型 | 是否必须 | 数据类型                    | schema                      |
| --------------------------- | -------------- | -------- | -------- | --------------------------- | --------------------------- |
| input                       | 添加银行卡参数 | body     | true     | AccountAddBankCardInputForm | AccountAddBankCardInputForm |
| &emsp;&emsp;bankAccountName | 银行账户名     |          | true     | string                      |                             |
| &emsp;&emsp;bankCardNum     | 银行卡号       |          | true     | string                      |                             |
| &emsp;&emsp;bankCode        | 银行编码       |          | true     | string                      |                             |
| &emsp;&emsp;cnapsCode       | 银行联行号     |          | true     | string                      |                             |
| &emsp;&emsp;realAddress     | 实名账户地址   |          | true     | string                      |                             |
| &emsp;&emsp;requestNo       | 请求号         |          | true     | string                      |                             |


**响应状态**:


| 状态码 | 说明         | schema                       |
| ------ | ------------ | ---------------------------- |
| 200    | OK           | Result«AccountAddBankCardVO» |
| 201    | Created      |                              |
| 401    | Unauthorized |                              |
| 403    | Forbidden    |                              |
| 404    | Not Found    |                              |


**响应参数**:


| 参数名称                       | 参数说明 | 类型                 | schema               |
| ------------------------------ | -------- | -------------------- | -------------------- |
| code                           |          | string               |                      |
| data                           |          | AccountAddBankCardVO | AccountAddBankCardVO |
| &emsp;&emsp;bankCardAddressBid |          | string               |                      |
| message                        |          | string               |                      |
| total                          |          | integer(int32)       | integer(int32)       |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"bankCardAddressBid": ""
	},
	"message": "",
	"total": 0
}
```

##### 子账户绑定银行卡


**接口地址**:`/if-account/api/account/v1/bindBankCard`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>账户绑定银行卡</p>

**请求示例**:


```javascript
{
  "bankAccountName": "星火",
  "bankCardAddress": "a001671df866fc0a41e6d8e8db18da0dde375275cb1dd8",
  "bankCardNum": 601106020000001093,
  "bankCardType": 0,
  "cnapsNo": 313146000035,
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "a001117c3e9d3d5b6315833b9f96a7b452ce489c5c0fc3",
  "remark": "绑定银行卡",
  "requestNo": 187542789578293248,
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明                                 | 请求类型 | 是否必须 | 数据类型                     | schema                       |
| ----------------------------- | ---------------------------------------- | -------- | -------- | ---------------------------- | ---------------------------- |
| input                         | 账户绑定银行卡参数                       | body     | true     | AccountBindBankCardInputForm | AccountBindBankCardInputForm |
| &emsp;&emsp;bankAccountName   | 银行账户名                               |          | true     | string                       |                              |
| &emsp;&emsp;bankCardAddress   | 银行卡地址，需通过“添加银行卡”接口获得   |          | true     | string                       |                              |
| &emsp;&emsp;bankCardNum       | 银行卡号                                 |          | true     | string                       |                              |
| &emsp;&emsp;bankCardType      | 绑卡类型 0:出入金，1：出金，2：入金      |          | true     | string                       |                              |
| &emsp;&emsp;cnapsNo           | 银行联行号                               |          | true     | string                       |                              |
| &emsp;&emsp;merchantId        | 商户号                                   |          | true     | string                       |                              |
| &emsp;&emsp;nonceStr          | 随机字符串                               |          | true     | string                       |                              |
| &emsp;&emsp;payAccountAddress | 支付账户地址，需通过“开立子账户”接口获得 |          | true     | string                       |                              |
| &emsp;&emsp;remark            | 备注                                     |          | true     | string                       |                              |
| &emsp;&emsp;requestNo         | 请求号                                   |          | true     | string                       |                              |
| &emsp;&emsp;sign              | 签名串                                   |          | true     | string                       |                              |
| &emsp;&emsp;signType          | 签名类型                                 |          | true     | string                       |                              |


**响应状态**:


| 状态码 | 说明         | schema                        |
| ------ | ------------ | ----------------------------- |
| 200    | OK           | Result«AccountBindBankCardVO» |
| 201    | Created      |                               |
| 401    | Unauthorized |                               |
| 403    | Forbidden    |                               |
| 404    | Not Found    |                               |


**响应参数**:


| 参数名称                    | 参数说明 | 类型                  | schema                |
| --------------------------- | -------- | --------------------- | --------------------- |
| code                        |          | string                |                       |
| data                        |          | AccountBindBankCardVO | AccountBindBankCardVO |
| &emsp;&emsp;bankCardAddress |          | string                |                       |
| &emsp;&emsp;code            |          | string                |                       |
| message                     |          | string                |                       |
| total                       |          | integer(int32)        | integer(int32)        |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"bankCardAddress": "",
		"code": ""
	},
	"message": "",
	"total": 0
}
```

##### 子账户解绑银行卡


**接口地址**:`/if-account/api/account/v1/unBindBankCard`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>账户解绑银行卡</p>

**请求示例**:


```javascript
{
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "realCardAddress": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明                                 | 请求类型 | 是否必须 | 数据类型                       | schema                         |
| ----------------------------- | ---------------------------------------- | -------- | -------- | ------------------------------ | ------------------------------ |
| input                         | 账户解绑银行卡参数                       | body     | true     | AccountUnBindBankCardInputForm | AccountUnBindBankCardInputForm |
| &emsp;&emsp;merchantId        | 商户号                                   |          | true     | string                         |                                |
| &emsp;&emsp;nonceStr          | 随机字符串                               |          | true     | string                         |                                |
| &emsp;&emsp;payAccountAddress | 支付账户地址，需通过“开立子账户”接口获得 |          | true     | string                         |                                |
| &emsp;&emsp;realCardAddress   | 实体卡地址，需通过“添加银行卡”接口获得   |          | true     | string                         |                                |
| &emsp;&emsp;requestNo         | 请求号                                   |          | true     | string                         |                                |
| &emsp;&emsp;sign              | 签名串                                   |          | true     | string                         |                                |
| &emsp;&emsp;signType          | 签名类型                                 |          | true     | string                         |                                |


**响应状态**:


| 状态码 | 说明         | schema                          |
| ------ | ------------ | ------------------------------- |
| 200    | OK           | Result«AccountUnBindBankCardVO» |
| 201    | Created      |                                 |
| 401    | Unauthorized |                                 |
| 403    | Forbidden    |                                 |
| 404    | Not Found    |                                 |


**响应参数**:


| 参数名称                    | 参数说明 | 类型                    | schema                  |
| --------------------------- | -------- | ----------------------- | ----------------------- |
| code                        |          | string                  |                         |
| data                        |          | AccountUnBindBankCardVO | AccountUnBindBankCardVO |
| &emsp;&emsp;bankCardAddress |          | string                  |                         |
| &emsp;&emsp;code            |          | string                  |                         |
| message                     |          | string                  |                         |
| total                       |          | integer(int32)          | integer(int32)          |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"bankCardAddress": "",
		"code": ""
	},
	"message": "",
	"total": 0
}
```

##### 充值blob生成

**接口地址**:`/if-account/api/account/v1/blob/txrecharge`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>充值blob生成</p>

**请求示例**:


```javascript
{
  "amount": 0,
  "bankCardAccountAddress": "",
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "remark": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                           | 参数说明                                           | 请求类型 | 是否必须 | 数据类型              | schema                |
| ---------------------------------- | -------------------------------------------------- | -------- | -------- | --------------------- | --------------------- |
| input                              | input                                              | body     | true     | RechargeBlobInputForm | RechargeBlobInputForm |
| &emsp;&emsp;amount                 | 金额                                               |          | true     | number                |                       |
| &emsp;&emsp;bankCardAccountAddress | 银行卡地址，需与对应的子账户先通过绑定接口完成绑定 |          | true     | string                |                       |
| &emsp;&emsp;merchantId             | 商户号                                             |          | true     | string                |                       |
| &emsp;&emsp;nonceStr               | 随机字符串                                         |          | true     | string                |                       |
| &emsp;&emsp;payAccountAddress      | bid-⼦账户区块链地址，需通过“开立子账户”接口获得   |          | true     | string                |                       |
| &emsp;&emsp;remark                 | 备注                                               |          | true     | string                |                       |
| &emsp;&emsp;requestNo              | 请求号                                             |          | true     | string                |                       |
| &emsp;&emsp;sign                   | 签名串                                             |          | true     | string                |                       |
| &emsp;&emsp;signType               | 签名类型                                           |          | true     | string                |                       |


**响应状态**:


| 状态码 | 说明         | schema                 |
| ------ | ------------ | ---------------------- |
| 200    | OK           | Result«BlobOutputForm» |
| 201    | Created      |                        |
| 401    | Unauthorized |                        |
| 403    | Forbidden    |                        |
| 404    | Not Found    |                        |


**响应参数**:


| 参数名称         | 参数说明 | 类型           | schema         |
| ---------------- | -------- | -------------- | -------------- |
| code             |          | string         |                |
| data             |          | BlobOutputForm | BlobOutputForm |
| &emsp;&emsp;blob | blob     | string         |                |
| &emsp;&emsp;hash | hash     | string         |                |
| message          |          | string         |                |
| total            |          | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"blob": "",
		"hash": ""
	},
	"message": "",
	"total": 0
}
```


##### 转账blob生成


**接口地址**:`/if-account/api/account/v1/blob/txtransfer`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>转账blob生成</p>

**请求示例**:


```javascript
{
  "amount": 0,
  "fromPayAccountAddress": "",
  "merchantId": "",
  "nonceStr": "",
  "remark": "",
  "requestNo": "",
  "sign": "",
  "signType": "",
  "toPayAccountAddress": "",
  "transferMake": "",
  "transferType": ""
}
```


**请求参数**:


| 参数名称                          | 参数说明                   | 请求类型 | 是否必须 | 数据类型              | schema                |
| --------------------------------- | -------------------------- | -------- | -------- | --------------------- | --------------------- |
| input                             | input                      | body     | true     | TransferBlobInputForm | TransferBlobInputForm |
| &emsp;&emsp;amount                | 转账金额                   |          | true     | number                |                       |
| &emsp;&emsp;fromPayAccountAddress | bid-转出⽅⼦账户区块链地址 |          | true     | string                |                       |
| &emsp;&emsp;merchantId            | 商户号                     |          | true     | string                |                       |
| &emsp;&emsp;nonceStr              | 随机字符串                 |          | true     | string                |                       |
| &emsp;&emsp;remark                | 备注                       |          | true     | string                |                       |
| &emsp;&emsp;requestNo             | 请求号                     |          | true     | string                |                       |
| &emsp;&emsp;sign                  | 签名串                     |          | true     | string                |                       |
| &emsp;&emsp;signType              | 签名类型                   |          | true     | string                |                       |
| &emsp;&emsp;toPayAccountAddress   | bid-接受⽅⼦账户区块链地址 |          | true     | string                |                       |
| &emsp;&emsp;transferMake          | 转账标识[1、凭证 2、订单]  |          | true     | string                |                       |
| &emsp;&emsp;transferType          | 转账类型                   |          | true     | string                |                       |


**响应状态**:


| 状态码 | 说明         | schema                 |
| ------ | ------------ | ---------------------- |
| 200    | OK           | Result«BlobOutputForm» |
| 201    | Created      |                        |
| 401    | Unauthorized |                        |
| 403    | Forbidden    |                        |
| 404    | Not Found    |                        |


**响应参数**:


| 参数名称         | 参数说明 | 类型           | schema         |
| ---------------- | -------- | -------------- | -------------- |
| code             |          | string         |                |
| data             |          | BlobOutputForm | BlobOutputForm |
| &emsp;&emsp;blob | blob     | string         |                |
| &emsp;&emsp;hash | hash     | string         |                |
| message          |          | string         |                |
| total            |          | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"blob": "",
		"hash": ""
	},
	"message": "",
	"total": 0
}
```


##### 提现blob生成


**接口地址**:`/if-account/api/account/v1/blob/txwithdraw`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>提现blob生成</p>

**请求示例**:


```javascript
{
  "amount": 0,
  "bankCardAccountAddress": "",
  "channelNo": "",
  "feePayType": "",
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "remark": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                           | 参数说明                                                     | 请求类型 | 是否必须 | 数据类型              | schema                |
| ---------------------------------- | ------------------------------------------------------------ | -------- | -------- | --------------------- | --------------------- |
| input                              | input                                                        | body     | true     | WithdrawBlobInputForm | WithdrawBlobInputForm |
| &emsp;&emsp;amount                 | 金额                                                         |          | true     | number                |                       |
| &emsp;&emsp;bankCardAccountAddress | bid-银⾏账户区块链地址，需与对应的子账户先通过绑定接口完成绑定 |          | true     | string                |                       |
| &emsp;&emsp;channelNo              | 提现渠道 华夏银⾏使⽤（默认 0000本\n⾏,3000⼤额出⾦,4000超级⽹银 |          | true     | string                |                       |
| &emsp;&emsp;feePayType             | 手续费承担方                                                 |          | true     | string                |                       |
| &emsp;&emsp;merchantId             | 商户号                                                       |          | true     | string                |                       |
| &emsp;&emsp;nonceStr               | 随机字符串                                                   |          | true     | string                |                       |
| &emsp;&emsp;payAccountAddress      | bid-⼦账户区块链地址，需通过“开立子账户”接口获得             |          | true     | string                |                       |
| &emsp;&emsp;remark                 | 备注                                                         |          | true     | string                |                       |
| &emsp;&emsp;requestNo              | 请求号                                                       |          | true     | string                |                       |
| &emsp;&emsp;sign                   | 签名串                                                       |          | true     | string                |                       |
| &emsp;&emsp;signType               | 签名类型                                                     |          | true     | string                |                       |


**响应状态**:


| 状态码 | 说明         | schema                 |
| ------ | ------------ | ---------------------- |
| 200    | OK           | Result«BlobOutputForm» |
| 201    | Created      |                        |
| 401    | Unauthorized |                        |
| 403    | Forbidden    |                        |
| 404    | Not Found    |                        |


**响应参数**:


| 参数名称         | 参数说明 | 类型           | schema         |
| ---------------- | -------- | -------------- | -------------- |
| code             |          | string         |                |
| data             |          | BlobOutputForm | BlobOutputForm |
| &emsp;&emsp;blob | blob     | string         |                |
| &emsp;&emsp;hash | hash     | string         |                |
| message          |          | string         |                |
| total            |          | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"blob": "",
		"hash": ""
	},
	"message": "",
	"total": 0
}
```

##### 充值-blob提交

**接口地址**:`/if-account/api/account/v1/transaction/recharge`


**请求方式**:`POST`


**请求数据类型**:`application/json`

**响应数据类型**:`*/*`

**接口描述**:<p>充值-blob提交</p>

**请求示例**:


```javascript
{
  "blob": "",
  "merchantId": "",
  "nonceStr": "",
  "requestNo": "",
  "sign": "",
  "signBlob": "",
  "signType": "",
  "signerPublicKey": ""
}
```


**请求参数**:


| 参数名称                    | 参数说明           | 请求类型 | 是否必须 | 数据类型          | schema            |
| --------------------------- | ------------------ | -------- | -------- | ----------------- | ----------------- |
| input                       | input              | body     | true     | RechargeInputForm | RechargeInputForm |
| &emsp;&emsp;blob            | blob串             |          | true     | string            |                   |
| &emsp;&emsp;merchantId      | 商户号             |          | true     | string            |                   |
| &emsp;&emsp;nonceStr        | 随机字符串         |          | true     | string            |                   |
| &emsp;&emsp;requestNo       | 请求号             |          | true     | string            |                   |
| &emsp;&emsp;sign            | 签名串             |          | true     | string            |                   |
| &emsp;&emsp;signBlob        | 账户充值操作签名串 |          | true     | string            |                   |
| &emsp;&emsp;signType        | 签名类型           |          | true     | string            |                   |
| &emsp;&emsp;signerPublicKey | 账户签名公钥       |          | true     | string            |                   |


**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | Result«RechargOutputForm» |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称              | 参数说明 | 类型              | schema            |
| --------------------- | -------- | ----------------- | ----------------- |
| code                  |          | string            |                   |
| data                  |          | RechargOutputForm | RechargOutputForm |
| &emsp;&emsp;bcHash    | 交易hash | string            |                   |
| &emsp;&emsp;requestNo | 请求号   | string            |                   |
| message               |          | string            |                   |
| total                 |          | integer(int32)    | integer(int32)    |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"bcHash": "",
		"requestNo": ""
	},
	"message": "",
	"total": 0
}
```


##### 提现-blob提交


**接口地址**:`/if-account/api/account/v1/transaction/withdraw`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>提现-blob提交</p>

**请求示例**:


```javascript
{
  "blob": "",
  "channelNo": "",
  "merchantId": "",
  "nonceStr": "",
  "requestNo": "",
  "sign": "",
  "signBlob": "",
  "signType": "",
  "signerPublicKey": ""
}
```


**请求参数**:


| 参数名称                    | 参数说明      | 请求类型 | 是否必须 | 数据类型          | schema            |
| --------------------------- | ------------- | -------- | -------- | ----------------- | ----------------- |
| input                       | input         | body     | true     | WithdrawInputForm | WithdrawInputForm |
| &emsp;&emsp;blob            | blob串        |          | true     | string            |                   |
| &emsp;&emsp;channelNo       | 华夏银行-渠道 |          | true     | string            |                   |
| &emsp;&emsp;merchantId      | 商户号        |          | true     | string            |                   |
| &emsp;&emsp;nonceStr        | 随机字符串    |          | true     | string            |                   |
| &emsp;&emsp;requestNo       | 请求号        |          | true     | string            |                   |
| &emsp;&emsp;sign            | 签名串        |          | true     | string            |                   |
| &emsp;&emsp;signBlob        | bolb签名串    |          | true     | string            |                   |
| &emsp;&emsp;signType        | 签名类型      |          | true     | string            |                   |
| &emsp;&emsp;signerPublicKey | 签名公钥      |          | true     | string            |                   |


**响应状态**:


| 状态码 | 说明         | schema                     |
| ------ | ------------ | -------------------------- |
| 200    | OK           | Result«WithdrawOutputForm» |
| 201    | Created      |                            |
| 401    | Unauthorized |                            |
| 403    | Forbidden    |                            |
| 404    | Not Found    |                            |


**响应参数**:


| 参数名称              | 参数说明                         | 类型               | schema             |
| --------------------- | -------------------------------- | ------------------ | ------------------ |
| code                  |                                  | string             |                    |
| data                  |                                  | WithdrawOutputForm | WithdrawOutputForm |
| &emsp;&emsp;code      | 内部状态 0成功，1失败，99 处理中 | string(byte)       |                    |
| &emsp;&emsp;requestNo | 请求号                           | string             |                    |
| &emsp;&emsp;tradeNo   | 交易号                           | string             |                    |
| message               |                                  | string             |                    |
| total                 |                                  | integer(int32)     | integer(int32)     |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"code": "",
		"requestNo": "",
		"tradeNo": ""
	},
	"message": "",
	"total": 0
}
```




##### 转账-blob提交


**接口地址**:`/if-account/api/account/v1/transaction/transfer`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>转账-blob</p>

**请求示例**:


```javascript
{
  "blob": "",
  "merchantId": "",
  "nonceStr": "",
  "requestNo": "",
  "sign": "",
  "signBlob": "",
  "signType": "",
  "signerPublicKey": "",
  "tradeAbstract": ""
}
```


**请求参数**:


| 参数名称                    | 参数说明           | 请求类型 | 是否必须 | 数据类型          | schema            |
| --------------------------- | ------------------ | -------- | -------- | ----------------- | ----------------- |
| input                       | input              | body     | true     | TransferInputForm | TransferInputForm |
| &emsp;&emsp;blob            | blob串             |          | true     | string            |                   |
| &emsp;&emsp;merchantId      | 商户号             |          | true     | string            |                   |
| &emsp;&emsp;nonceStr        | 随机字符串         |          | true     | string            |                   |
| &emsp;&emsp;requestNo       | 请求号             |          | true     | string            |                   |
| &emsp;&emsp;sign            | 签名串             |          | true     | string            |                   |
| &emsp;&emsp;signBlob        | 账户转账操作签名串 |          | true     | string            |                   |
| &emsp;&emsp;signType        | 签名类型           |          | true     | string            |                   |
| &emsp;&emsp;signerPublicKey | 签名公钥           |          | true     | string            |                   |
| &emsp;&emsp;tradeAbstract   | 交易摘要           |          | false    | string            |                   |


**响应状态**:


| 状态码 | 说明         | schema                     |
| ------ | ------------ | -------------------------- |
| 200    | OK           | Result«TransferOutputForm» |
| 201    | Created      |                            |
| 401    | Unauthorized |                            |
| 403    | Forbidden    |                            |
| 404    | Not Found    |                            |


**响应参数**:


| 参数名称              | 参数说明 | 类型               | schema             |
| --------------------- | -------- | ------------------ | ------------------ |
| code                  |          | string             |                    |
| data                  |          | TransferOutputForm | TransferOutputForm |
| &emsp;&emsp;bcHash    | 交易hash | string             |                    |
| &emsp;&emsp;requestNo | 请求号   | string             |                    |
| message               |          | string             |                    |
| total                 |          | integer(int32)     | integer(int32)     |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"bcHash": "",
		"requestNo": ""
	},
	"message": "",
	"total": 0
}
```

##### 子账户资金冻结


**接口地址**:`/if-account/api/account/v1/transaction/freeze`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>资金冻结</p>

**请求示例**:


```javascript
{
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "remark1": "",
  "requestNo": "",
  "sign": "",
  "signType": "",
  "tranAmount": 0
}
```


**请求参数**:


| 参数名称                      | 参数说明             | 请求类型 | 是否必须 | 数据类型                | schema                  |
| ----------------------------- | -------------------- | -------- | -------- | ----------------------- | ----------------------- |
| inputForm                     | inputForm            | body     | true     | PayFreezeTransInputForm | PayFreezeTransInputForm |
| &emsp;&emsp;merchantId        | 商户号               |          | true     | string                  |                         |
| &emsp;&emsp;nonceStr          | 随机字符串           |          | true     | string                  |                         |
| &emsp;&emsp;payAccountAddress | bid-⼦账户区块链地址 |          | false    | string                  |                         |
| &emsp;&emsp;remark1           | 备注                 |          | true     | string                  |                         |
| &emsp;&emsp;requestNo         | 请求号               |          | true     | string                  |                         |
| &emsp;&emsp;sign              | 签名串               |          | true     | string                  |                         |
| &emsp;&emsp;signType          | 签名类型             |          | true     | string                  |                         |
| &emsp;&emsp;tranAmount        | 冻结金额             |          | true     | number                  |                         |


**响应状态**:


| 状态码 | 说明         | schema                   |
| ------ | ------------ | ------------------------ |
| 200    | OK           | Result«PayFreezeTransVO» |
| 201    | Created      |                          |
| 401    | Unauthorized |                          |
| 403    | Forbidden    |                          |
| 404    | Not Found    |                          |


**响应参数**:


| 参数名称                    | 参数说明     | 类型             | schema           |
| --------------------------- | ------------ | ---------------- | ---------------- |
| code                        |              | string           |                  |
| data                        |              | PayFreezeTransVO | PayFreezeTransVO |
| &emsp;&emsp;balanceAmount   | 余额         | number           |                  |
| &emsp;&emsp;frezTotalAmount | 冻结总金额   | number           |                  |
| &emsp;&emsp;requestNo       | 请求号       | string           |                  |
| &emsp;&emsp;tradeNo         | 流水号       | string           |                  |
| &emsp;&emsp;tranAmount      | 本次冻结金额 | number           |                  |
| message                     |              | string           |                  |
| total                       |              | integer(int32)   | integer(int32)   |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"balanceAmount": 0,
		"frezTotalAmount": 0,
		"requestNo": "",
		"tradeNo": "",
		"tranAmount": 0
	},
	"message": "",
	"total": 0
}
```


##### 子账户资金解冻


**接口地址**:`/if-account/api/account/v1/transaction/unfreeze`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>资金解冻</p>

**请求示例**:


```javascript
{
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "remark1": "",
  "requestNo": "",
  "sign": "",
  "signType": "",
  "tranAmount": 0
}
```


**请求参数**:


| 参数名称                      | 参数说明             | 请求类型 | 是否必须 | 数据类型                  | schema                    |
| ----------------------------- | -------------------- | -------- | -------- | ------------------------- | ------------------------- |
| inputForm                     | inputForm            | body     | true     | PayUnFreezeTransInputForm | PayUnFreezeTransInputForm |
| &emsp;&emsp;merchantId        | 商户号               |          | true     | string                    |                           |
| &emsp;&emsp;nonceStr          | 随机字符串           |          | true     | string                    |                           |
| &emsp;&emsp;payAccountAddress | bid-⼦账户区块链地址 |          | false    | string                    |                           |
| &emsp;&emsp;remark1           | 备注                 |          | true     | string                    |                           |
| &emsp;&emsp;requestNo         | 请求号               |          | true     | string                    |                           |
| &emsp;&emsp;sign              | 签名串               |          | true     | string                    |                           |
| &emsp;&emsp;signType          | 签名类型             |          | true     | string                    |                           |
| &emsp;&emsp;tranAmount        | 解冻金额             |          | true     | number                    |                           |


**响应状态**:


| 状态码 | 说明         | schema                     |
| ------ | ------------ | -------------------------- |
| 200    | OK           | Result«PayUnFreezeTransVO» |
| 201    | Created      |                            |
| 401    | Unauthorized |                            |
| 403    | Forbidden    |                            |
| 404    | Not Found    |                            |


**响应参数**:


| 参数名称                    | 参数说明     | 类型               | schema             |
| --------------------------- | ------------ | ------------------ | ------------------ |
| code                        |              | string             |                    |
| data                        |              | PayUnFreezeTransVO | PayUnFreezeTransVO |
| &emsp;&emsp;balanceAmount   | 余额         | number             |                    |
| &emsp;&emsp;frezTotalAmount | 冻结总金额   | number             |                    |
| &emsp;&emsp;requestNo       | 请求号       | string             |                    |
| &emsp;&emsp;tradeNo         | 流水号       | string             |                    |
| &emsp;&emsp;tranAmount      | 本次解冻金额 | number             |                    |
| message                     |              | string             |                    |
| total                       |              | integer(int32)     | integer(int32)     |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"balanceAmount": 0,
		"frezTotalAmount": 0,
		"requestNo": "",
		"tradeNo": "",
		"tranAmount": 0
	},
	"message": "",
	"total": 0
}
```

#### 账户服务：查询操作


##### 电子回单下载


**接口地址**:`/if-account/api/account/v1/status/external-receipt`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>电子回单下载</p>

**请求示例**:


```javascript
{
  "accountTradeNo": "",
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "requestNo": "",
  "sign": "",
  "signType": "",
  "tranType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明                                                     | 请求类型 | 是否必须 | 数据类型             | schema               |
| ----------------------------- | ------------------------------------------------------------ | -------- | -------- | -------------------- | -------------------- |
| input                         | input                                                        | body     | true     | DownVoucherInputForm | DownVoucherInputForm |
| &emsp;&emsp;accountTradeNo    | 流水号                                                       |          | true     | string               |                      |
| &emsp;&emsp;merchantId        | 商户号                                                       |          | true     | string               |                      |
| &emsp;&emsp;nonceStr          | 随机字符串                                                   |          | true     | string               |                      |
| &emsp;&emsp;payAccountAddress | 支付账户地址bid                                              |          | true     | string               |                      |
| &emsp;&emsp;requestNo         | 请求号                                                       |          | true     | string               |                      |
| &emsp;&emsp;sign              | 签名串                                                       |          | true     | string               |                      |
| &emsp;&emsp;signType          | 签名类型                                                     |          | true     | string               |                      |
| &emsp;&emsp;tranType          | 交易类型 0 提现 1 转账 2 充值 3⼿续费 4融资\n贷款 5还款 7专项计划 10融资服务费 12平台服\n务费 14分润 |          | true     | string               |                      |


**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | ExternalReceiptOutputForm |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称   | 参数说明               | 类型   | schema |
| ---------- | ---------------------- | ------ | ------ |
| bankMemNum | 资金账簿信息           | string |        |
| downUrl    | 下载路径               | string |        |
| verifyCode | 下载电子回单打印校验码 | string |        |


**响应示例**:

```javascript
{
	"bankMemNum": "",
	"downUrl": "",
	"verifyCode": ""
}
```


##### 交易状态


**接口地址**:`/if-account/api/account/v1/status/transaction-status`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>交易状态</p>

**请求示例**:


```javascript
{
  "bizType": 0,
  "merchantId": "",
  "nonceStr": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称               | 参数说明                      | 请求类型 | 是否必须 | 数据类型          | schema            |
| ---------------------- | ----------------------------- | -------- | -------- | ----------------- | ----------------- |
| input                  | input                         | body     | true     | TxStatusInputForm | TxStatusInputForm |
| &emsp;&emsp;bizType    | 交易类型 0 提现 1 转账 2 充值 |          | true     | integer(int32)    |                   |
| &emsp;&emsp;merchantId | 商户号                        |          | true     | string            |                   |
| &emsp;&emsp;nonceStr   | 随机字符串                    |          | true     | string            |                   |
| &emsp;&emsp;requestNo  | 请求号                        |          | true     | string            |                   |
| &emsp;&emsp;sign       | 签名串                        |          | true     | string            |                   |
| &emsp;&emsp;signType   | 签名类型                      |          | true     | string            |                   |


**响应状态**:


| 状态码 | 说明         | schema        |
| ------ | ------------ | ------------- |
| 200    | OK           | TransStatusVO |
| 201    | Created      |               |
| 401    | Unauthorized |               |
| 403    | Forbidden    |               |
| 404    | Not Found    |               |


**响应参数**:


| 参数名称  | 参数说明 | 类型           | schema         |
| --------- | -------- | -------------- | -------------- |
| hash      | 交易hash | string         |                |
| requestNo | 请求号   | string         |                |
| txStatus  | 交易状态 | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"hash": "",
	"requestNo": "",
	"txStatus": 0
}
```


##### 账户余额


**接口地址**:`/if-account/api/account/v1/balance`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>账户余额</p>

**请求示例**:


```javascript
{
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "queryType": "0:账户余额，1:子账户余额，2:结算户余额,3:监管户余额",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明         | 请求类型 | 是否必须 | 数据类型                | schema                  |
| ----------------------------- | ---------------- | -------- | -------- | ----------------------- | ----------------------- |
| input                         | 账户余额参数     | body     | true     | AccountBalanceInputForm | AccountBalanceInputForm |
| &emsp;&emsp;merchantId        | 商户号           |          | true     | string                  |                         |
| &emsp;&emsp;nonceStr          | 随机字符串       |          | true     | string                  |                         |
| &emsp;&emsp;payAccountAddress | 支付账户地址     |          | true     | string                  |                         |
| &emsp;&emsp;queryType         | 查询余额账户类型 |          | true     | string                  |                         |
| &emsp;&emsp;requestNo         | 请求号           |          | true     | string                  |                         |
| &emsp;&emsp;sign              | 签名串           |          | true     | string                  |                         |
| &emsp;&emsp;signType          | 签名类型         |          | true     | string                  |                         |


**响应状态**:


| 状态码 | 说明         | schema                   |
| ------ | ------------ | ------------------------ |
| 200    | OK           | Result«AccountBalanceVO» |
| 201    | Created      |                          |
| 401    | Unauthorized |                          |
| 403    | Forbidden    |                          |
| 404    | Not Found    |                          |


**响应参数**:


| 参数名称                   | 参数说明 | 类型             | schema           |
| -------------------------- | -------- | ---------------- | ---------------- |
| code                       |          | string           |                  |
| data                       |          | AccountBalanceVO | AccountBalanceVO |
| &emsp;&emsp;frozenBalance  | 冻结余额 | number           |                  |
| &emsp;&emsp;totalBalance   | 总余额   | number           |                  |
| &emsp;&emsp;useAbleBalance | 可用余额 | number           |                  |
| message                    |          | string           |                  |
| total                      |          | integer(int32)   | integer(int32)   |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"frozenBalance": 0,
		"totalBalance": 0,
		"useAbleBalance": 0
	},
	"message": "",
	"total": 0
}
```


##### 账户流水


**接口地址**:`/if-account/api/account/v1/bill`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>账户流水</p>

**请求示例**:


```javascript
{
  "accountingStatus": "",
  "bizType": "",
  "endTime": "",
  "maxAmount": 0,
  "merchantId": "",
  "minAmount": 0,
  "nonceStr": "",
  "oppositeAccountName": "",
  "oppositeAccountNumber": "",
  "pageNum": 1,
  "pageSize": 20,
  "payAccountAddress": "",
  "requestNo": "",
  "sign": "",
  "signType": "",
  "startTime": ""
}
```


**请求参数**:


| 参数名称                          | 参数说明     | 请求类型 | 是否必须 | 数据类型             | schema               |
| --------------------------------- | ------------ | -------- | -------- | -------------------- | -------------------- |
| input                             | 账户流水参数 | body     | true     | AccountBillInputForm | AccountBillInputForm |
| &emsp;&emsp;accountingStatus      | 入账类型     |          | true     | string               |                      |
| &emsp;&emsp;bizType               | 交易类型     |          | true     | string               |                      |
| &emsp;&emsp;endTime               | 结束时间     |          | true     | string               |                      |
| &emsp;&emsp;maxAmount             | 最大金额     |          | false    | number               |                      |
| &emsp;&emsp;merchantId            | 商户号       |          | true     | string               |                      |
| &emsp;&emsp;minAmount             | 最小金额     |          | false    | number               |                      |
| &emsp;&emsp;nonceStr              | 随机字符串   |          | true     | string               |                      |
| &emsp;&emsp;oppositeAccountName   | 对手方账户名 |          | true     | string               |                      |
| &emsp;&emsp;oppositeAccountNumber | 对手方账户号 |          | true     | string               |                      |
| &emsp;&emsp;pageNum               | 当前页       |          | false    | integer(int32)       |                      |
| &emsp;&emsp;pageSize              | 每页记录数   |          | false    | integer(int32)       |                      |
| &emsp;&emsp;payAccountAddress     | 支付账户地址 |          | true     | string               |                      |
| &emsp;&emsp;requestNo             | 请求号       |          | true     | string               |                      |
| &emsp;&emsp;sign                  | 签名串       |          | true     | string               |                      |
| &emsp;&emsp;signType              | 签名类型     |          | true     | string               |                      |
| &emsp;&emsp;startTime             | 开始时间     |          | true     | string               |                      |


**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | PageResult«AccountBillVO» |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称                                  | 参数说明                              | 类型                | schema              |
| ----------------------------------------- | ------------------------------------- | ------------------- | ------------------- |
| code                                      |                                       | string              |                     |
| data                                      |                                       | Data«AccountBillVO» | Data«AccountBillVO» |
| &emsp;&emsp;list                          |                                       | array               | AccountBillVO       |
| &emsp;&emsp;&emsp;&emsp;accountTradeNo    | 交易流水号                            | string              |                     |
| &emsp;&emsp;&emsp;&emsp;accountingStatus  | 入账类型（支出0、收入1）              | string              |                     |
| &emsp;&emsp;&emsp;&emsp;accountingTime    | 入账时间                              | string              |                     |
| &emsp;&emsp;&emsp;&emsp;amount            | 入账金额                              | number              |                     |
| &emsp;&emsp;&emsp;&emsp;bankCode          | 银行code                              | string              |                     |
| &emsp;&emsp;&emsp;&emsp;billNum           | 账单数量                              | string              |                     |
| &emsp;&emsp;&emsp;&emsp;bizType           | 业务类型（充值、提现、融资、还款 等） | string              |                     |
| &emsp;&emsp;&emsp;&emsp;createTime        | 创建时间                              | string              |                     |
| &emsp;&emsp;&emsp;&emsp;feeAmount         | 手续费                                | number              |                     |
| &emsp;&emsp;&emsp;&emsp;id                |                                       | integer             |                     |
| &emsp;&emsp;&emsp;&emsp;metadata          | 业务信息                              | string              |                     |
| &emsp;&emsp;&emsp;&emsp;payAccountAddress | bid- ⼦账户区块链地址                 | string              |                     |
| &emsp;&emsp;&emsp;&emsp;payAccountBalance | 当时账户余额                          | number              |                     |
| &emsp;&emsp;&emsp;&emsp;remark1           | 备注                                  | string              |                     |
| &emsp;&emsp;&emsp;&emsp;updateTime        | 更新时间                              | string              |                     |
| &emsp;&emsp;&emsp;&emsp;uploadStatus      |                                       | string              |                     |
| &emsp;&emsp;total                         |                                       | integer(int64)      |                     |
| msg                                       |                                       | string              |                     |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"list": [
			{
				"accountTradeNo": "",
				"accountingStatus": "",
				"accountingTime": "",
				"amount": 0,
				"bankCode": "",
				"billNum": "",
				"bizType": "",
				"createTime": "",
				"feeAmount": 0,
				"id": 0,
				"metadata": "",
				"payAccountAddress": "",
				"payAccountBalance": 0,
				"remark1": "",
				"updateTime": "",
				"uploadStatus": ""
			}
		],
		"total": 0
	},
	"msg": ""
}
```


##### 跨行子账户查询


**接口地址**:`/if-account/api/account/v1/crossBankSubAccount`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>跨行子账户查询</p>

**请求示例**:


```javascript
{
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明           | 请求类型 | 是否必须 | 数据类型                     | schema                       |
| ----------------------------- | ------------------ | -------- | -------- | ---------------------------- | ---------------------------- |
| input                         | 跨行子账户查询参数 | body     | true     | AccountCrossBankSubInputForm | AccountCrossBankSubInputForm |
| &emsp;&emsp;merchantId        | 商户号             |          | true     | string                       |                              |
| &emsp;&emsp;nonceStr          | 随机字符串         |          | true     | string                       |                              |
| &emsp;&emsp;payAccountAddress | 支付账户地址       |          | true     | string                       |                              |
| &emsp;&emsp;requestNo         | 请求号             |          | true     | string                       |                              |
| &emsp;&emsp;sign              | 签名串             |          | true     | string                       |                              |
| &emsp;&emsp;signType          | 签名类型           |          | true     | string                       |                              |


**响应状态**:


| 状态码 | 说明         | schema                        |
| ------ | ------------ | ----------------------------- |
| 200    | OK           | Result«AccountCrossBankSubVO» |
| 201    | Created      |                               |
| 401    | Unauthorized |                               |
| 403    | Forbidden    |                               |
| 404    | Not Found    |                               |


**响应参数**:


| 参数名称                             | 参数说明                 | 类型                  | schema                |
| ------------------------------------ | ------------------------ | --------------------- | --------------------- |
| code                                 |                          | string                |                       |
| data                                 |                          | AccountCrossBankSubVO | AccountCrossBankSubVO |
| &emsp;&emsp;bankCode                 | 开户⾏银⾏编码           | string                |                       |
| &emsp;&emsp;bankInterNum             | 跨⾏收款账号             | string                |                       |
| &emsp;&emsp;bankPayeeSubAccName      | 跨行收款账户名称         | string                |                       |
| &emsp;&emsp;bankPayeeSubAccSetteName | 跨行收款子账号清算行名称 | string                |                       |
| &emsp;&emsp;payAccountAddress        | 跨⾏bid-⼦账户区块链地址 | string                |                       |
| message                              |                          | string                |                       |
| total                                |                          | integer(int32)        | integer(int32)        |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"bankCode": "",
		"bankInterNum": "",
		"bankPayeeSubAccName": "",
		"bankPayeeSubAccSetteName": "",
		"payAccountAddress": ""
	},
	"message": "",
	"total": 0
}
```


##### 账户信息查询


**接口地址**:`/if-account/api/account/v1/getInfo`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>账户信息</p>

**请求示例**:


```javascript
{
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明     | 请求类型 | 是否必须 | 数据类型             | schema               |
| ----------------------------- | ------------ | -------- | -------- | -------------------- | -------------------- |
| input                         | 账户信息参数 | body     | true     | AccountEditInputForm | AccountEditInputForm |
| &emsp;&emsp;merchantId        | 商户号       |          | true     | string               |                      |
| &emsp;&emsp;nonceStr          | 随机字符串   |          | true     | string               |                      |
| &emsp;&emsp;payAccountAddress | 支付账户地址 |          | true     | string               |                      |
| &emsp;&emsp;requestNo         | 请求号       |          | true     | string               |                      |
| &emsp;&emsp;sign              | 签名串       |          | true     | string               |                      |
| &emsp;&emsp;signType          | 签名类型     |          | true     | string               |                      |


**响应状态**:


| 状态码 | 说明         | schema                |
| ------ | ------------ | --------------------- |
| 200    | OK           | Result«AccountInfoVO» |
| 201    | Created      |                       |
| 401    | Unauthorized |                       |
| 403    | Forbidden    |                       |
| 404    | Not Found    |                       |


**响应参数**:


| 参数名称                      | 参数说明             | 类型           | schema         |
| ----------------------------- | -------------------- | -------------- | -------------- |
| code                          |                      | string         |                |
| data                          |                      | AccountInfoVO  | AccountInfoVO  |
| &emsp;&emsp;accountAddress    | bid-企业区块链地址   | string         |                |
| &emsp;&emsp;bankCode          | 银行CODE             | string         |                |
| &emsp;&emsp;bankInterNum      | 跨⾏收款账号         | string         |                |
| &emsp;&emsp;bankMemNum        | 银⾏会员⼦账户       | string         |                |
| &emsp;&emsp;bankName          | 银行名称             | string         |                |
| &emsp;&emsp;payAccountAddress | bid-⼦账户区块链地址 | string         |                |
| &emsp;&emsp;payAccountNum     | 支付账号             | string         |                |
| &emsp;&emsp;payAccountType    | 支付账户类型         | string         |                |
| message                       |                      | string         |                |
| total                         |                      | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"accountAddress": "",
		"bankCode": "",
		"bankInterNum": "",
		"bankMemNum": "",
		"bankName": "",
		"payAccountAddress": "",
		"payAccountNum": "",
		"payAccountType": ""
	},
	"message": "",
	"total": 0
}
```

##### 跨行出入金手续费试算


**接口地址**:`/if-account/api/account/v1/transaction/calculatefee`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>跨行出入金手续费试算</p>

**请求示例**:


```javascript
{
  "amount": "",
  "bankCode": "",
  "inOutMoneyType": "",
  "isCrossLine": "",
  "merchantId": "",
  "nonceStr": "",
  "payAccountAddress": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明                                       | 请求类型 | 是否必须 | 数据类型              | schema                |
| ----------------------------- | ---------------------------------------------- | -------- | -------- | --------------------- | --------------------- |
| input                         | input                                          | body     | true     | FeeCalculateInputForm | FeeCalculateInputForm |
| &emsp;&emsp;amount            | 金额                                           |          | true     | string                |                       |
| &emsp;&emsp;bankCode          | 银行CODE                                       |          | true     | string                |                       |
| &emsp;&emsp;inOutMoneyType    | 华夏银行-选择跨行手续费试算类型0-出金，1，入金 |          | true     | string                |                       |
| &emsp;&emsp;isCrossLine       | 是否跨行                                       |          | true     | string                |                       |
| &emsp;&emsp;merchantId        | 商户号                                         |          | true     | string                |                       |
| &emsp;&emsp;nonceStr          | 随机字符串                                     |          | true     | string                |                       |
| &emsp;&emsp;payAccountAddress | 华夏银行-支付账户地址                          |          | false    | string                |                       |
| &emsp;&emsp;requestNo         | 请求号                                         |          | true     | string                |                       |
| &emsp;&emsp;sign              | 签名串                                         |          | true     | string                |                       |
| &emsp;&emsp;signType          | 签名类型                                       |          | true     | string                |                       |


**响应状态**:


| 状态码 | 说明         | schema                         |
| ------ | ------------ | ------------------------------ |
| 200    | OK           | Result«FeeCalculateOutputForm» |
| 201    | Created      |                                |
| 401    | Unauthorized |                                |
| 403    | Forbidden    |                                |
| 404    | Not Found    |                                |


**响应参数**:


| 参数名称              | 参数说明   | 类型                   | schema                 |
| --------------------- | ---------- | ---------------------- | ---------------------- |
| code                  |            | string                 |                        |
| data                  |            | FeeCalculateOutputForm | FeeCalculateOutputForm |
| &emsp;&emsp;feeAmount | 手续费金额 | integer(int64)         |                        |
| &emsp;&emsp;feeNo     | 手续费模型 | string                 |                        |
| message               |            | string                 |                        |
| total                 |            | integer(int32)         | integer(int32)         |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"feeAmount": 0,
		"feeNo": ""
	},
	"message": "",
	"total": 0
}
```

### 2.8.融资服务

**简介**:用于产融融资信息操作，包含融资客户信息推送，融资成本试算申请，融资申请，融资查询，查询等操作。

#### 状态机

<img src="../_static/images/image1.png"/>

#### 操作类接口调用流程

- 步骤1

  开通融资产品，星火产融会维护不同的融资服务商，第三方平台根据产融端分配的融资服务商进行融资产品的开通调用。

  开通融资产品接口调用之后，不同的服务商应答时间不等，产融会利用定时任务去不同的服务商去查询融资产品的开通结果。

  <img src="../_static/images/image2.png"/>

- 步骤2

  融资提交准备接口，入参nonce值由第三方产融平台自行维护，可通过辅助接口：/api/finance/v1/blockchain/get-nonce获取指定账户在链上的实时nonce值，+1后作为入参。

  融资提交准备接口，返回的待签名数据需使用clientPlatformBid对应账户私钥进行数据签名

  相同融资申请单号，如果单据不被退回，不允许多次提交

  /api/finance/v1/finance/submit接口执行之后，需要等待区块链执行结果，区块链交易执行成功，该操作成功，此时业务状态为初始化。

  <img src="../_static/images/image3.png"/>

- 步骤3

  针对步骤2已执行成功的融资数据，可以调用融资指令转发接口，将融资数据推送到服务商。

  入参中，contractAddress、hash、label 需要依赖步骤2中返回的结果。

  调用此接口之后，业务状态为处理中。

  <img src="../_static/images/image4.png"/>

#### 融资服务：业务操作

##### 开通融资产品


**接口地址**:`/if-finance/api/finance/v1/custom/open-finance`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>开通融资产品</p>

**请求示例**:


```javascript
{
  "astAmt": 0,
  "clientPlatformBid": "",
  "corpName": "",
  "corpNo": "",
  "corpNumber": "",
  "ctfExpDate": "",
  "ctfRegDate": "",
  "ctfType": "",
  "customBid": "",
  "customName": "",
  "customNo": "",
  "dtlAddr": "",
  "financeYear": "",
  "interestPaymentCode": "",
  "lbyAmt": 0,
  "linkName": "",
  "linkNumber": "",
  "merchantId": "",
  "netAsset": 0,
  "netProfit": 0,
  "oppCtfexpDt": "",
  "oppCtfrgstDt": "",
  "oppNm": "",
  "oppNo": "",
  "oppType": "",
  "oppmbNo": "",
  "permitNo": "",
  "postAddr": "",
  "registerTime": "",
  "requestNo": "",
  "saleIncm": ""
}
```


**请求参数**:


| 参数名称                        | 参数说明                                        | 请求类型 | 是否必须 | 数据类型             | schema               |
| ------------------------------- | ----------------------------------------------- | -------- | -------- | -------------------- | -------------------- |
| input                           | input                                           | body     | true     | OpenFinanceInputForm | OpenFinanceInputForm |
| &emsp;&emsp;astAmt              | 资产总额:元                                     |          | false    | number               |                      |
| &emsp;&emsp;clientPlatformBid   | 产融平台Bid                                     |          | true     | string               |                      |
| &emsp;&emsp;corpName            | 法人姓名                                        |          | true     | string               |                      |
| &emsp;&emsp;corpNo              | 法人证件号                                      |          | false    | string               |                      |
| &emsp;&emsp;corpNumber          | 法人联系电话                                    |          | false    | string               |                      |
| &emsp;&emsp;ctfExpDate          | 证件到期日-yyyyMMdd                             |          | false    | string               |                      |
| &emsp;&emsp;ctfRegDate          | 证件注册日-yyyyMMdd                             |          | false    | string               |                      |
| &emsp;&emsp;ctfType             | 客户证件类型01-组织机构代码 02-统⼀社会信⽤代码 |          | true     | string               |                      |
| &emsp;&emsp;customBid           | 客户Bid                                         |          | true     | string               |                      |
| &emsp;&emsp;customName          | 客户名称                                        |          | true     | string               |                      |
| &emsp;&emsp;customNo            | 客户证件号                                      |          | true     | string               |                      |
| &emsp;&emsp;dtlAddr             | 注册地址                                        |          | false    | string               |                      |
| &emsp;&emsp;financeYear         | 财务信息所属年份                                |          | false    | string               |                      |
| &emsp;&emsp;interestPaymentCode | 付息⽅式 1-卖⽅付息 2-买⽅付息                  |          | false    | string               |                      |
| &emsp;&emsp;lbyAmt              | 负债总额:元                                     |          | false    | number               |                      |
| &emsp;&emsp;linkName            | 联系人                                          |          | false    | string               |                      |
| &emsp;&emsp;linkNumber          | 联系电话                                        |          | false    | string               |                      |
| &emsp;&emsp;merchantId          | 商户号                                          |          | true     | string               |                      |
| &emsp;&emsp;netAsset            | 净资产:元                                       |          | false    | number               |                      |
| &emsp;&emsp;netProfit           | 净利润:元                                       |          | false    | number               |                      |
| &emsp;&emsp;oppCtfexpDt         | 经办⼈证件到期⽇                                |          | false    | string               |                      |
| &emsp;&emsp;oppCtfrgstDt        | 经办⼈证件注册⽇                                |          | false    | string               |                      |
| &emsp;&emsp;oppNm               | 经办⼈姓名                                      |          | false    | string               |                      |
| &emsp;&emsp;oppNo               | 经办⼈证件号                                    |          | false    | string               |                      |
| &emsp;&emsp;oppType             | 经办⼈证件类型                                  |          | false    | string               |                      |
| &emsp;&emsp;oppmbNo             | 经办⼈⼿机号                                    |          | false    | string               |                      |
| &emsp;&emsp;permitNo            | 开户许可证核准号                                |          | false    | string               |                      |
| &emsp;&emsp;postAddr            | 通讯地址                                        |          | false    | string               |                      |
| &emsp;&emsp;registerTime        | 企业在平台注册的时间                            |          | false    | string               |                      |
| &emsp;&emsp;requestNo           | 请求号                                          |          | true     | string               |                      |
| &emsp;&emsp;saleIncm            | 营业收⼊                                        |          | false    | string               |                      |


**响应状态**:


| 状态码 | 说明         | schema                      |
| ------ | ------------ | --------------------------- |
| 200    | OK           | Result«OpenFinanceOutputVO» |
| 201    | Created      |                             |
| 401    | Unauthorized |                             |
| 403    | Forbidden    |                             |
| 404    | Not Found    |                             |


**响应参数**:


| 参数名称           | 参数说明           | 类型                | schema              |
| ------------------ | ------------------ | ------------------- | ------------------- |
| code               |                    | string              |                     |
| data               |                    | OpenFinanceOutputVO | OpenFinanceOutputVO |
| &emsp;&emsp;status | 状态 0-成功 1-失败 | string              |                     |
| message            |                    | string              |                     |
| total              |                    | integer(int32)      | integer(int32)      |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"status": ""
	},
	"message": "",
	"total": 0
}
```

##### 融资提交准备

**接口地址**:`/if-finance/api/finance/v1/finance/prepare`


**请求方式**:`POST`

**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>融资提交准备</p>

**请求示例**:


```javascript
{
  "annexInfoList": [
    {
      "annexLabel": "",
      "annexName": "",
      "annexType": "",
      "ext": ""
    }
  ],
  "applyRate": 0,
  "certCode": "",
  "certType": "",
  "clientPlatformBid": "",
  "contractInfoList": [
    {
      "annexLabel": "",
      "buyerName": "",
      "contractAmount": 0,
      "contractCode": "",
      "contractName": "",
      "endDate": 0,
      "sellerName": "",
      "signDate": 0
    }
  ],
  "customBid": "",
  "customName": "",
  "dueDate": 0,
  "financeNo": "",
  "financingAmount": 0,
  "financingRatio": 0,
  "interestPaymentCode": "",
  "invoiceInfoList": [
    {
      "applyAmount": 0,
      "invoiceHash": ""
    }
  ],
  "loanAccountBid": "",
  "merchantId": "",
  "nonce": 0,
  "nonceStr": "",
  "phone": "",
  "repaymentAccountBid": "",
  "requestNo": "",
  "serviceFee": {
    "amount": 0,
    "fromAccountBid": "",
    "toAccountBid": ""
  },
  "sign": "",
  "signType": "",
  "validateInfo": {
    "hash": "",
    "originalData": "",
    "publicKey": "",
    "signature": "",
    "smartContractBid": "",
    "smartContractLabel": "",
    "validateType": ""
  },
  "voucherAmount": 0,
  "voucherInfoList": [
    {
      "amount": 0,
      "cashTime": 0,
      "contractInfoList": [
        {
          "annexLabel": "",
          "buyerName": "",
          "contractAmount": 0,
          "contractCode": "",
          "contractName": "",
          "endDate": 0,
          "sellerName": "",
          "signDate": 0
        }
      ],
      "createTime": 0,
      "invoiceInfoList": [
        {
          "applyAmount": 0,
          "invoiceHash": ""
        }
      ],
      "parentVoucherNo": "",
      "payee": {
        "certNo": "",
        "certType": "",
        "companyName": "",
        "phone": ""
      },
      "payer": {
        "certNo": "",
        "certType": "",
        "companyName": "",
        "phone": ""
      },
      "rootVouchNo": "",
      "voucherNo": ""
    }
  ],
  "voucherNo": ""
}
```


**请求参数**:


| 参数名称                                           | 参数说明                                                     | 请求类型 | 是否必须 | 数据类型                  | schema                    |
| -------------------------------------------------- | ------------------------------------------------------------ | -------- | -------- | ------------------------- | ------------------------- |
| input                                              | input                                                        | body     | true     | FinancePrepareInputForm   | FinancePrepareInputForm   |
| &emsp;&emsp;annexInfoList                          | 附件列表                                                     |          | true     | array                     | ExAnnexInputForm          |
| &emsp;&emsp;&emsp;&emsp;annexLabel                 | 产融中台为每个文件提供的唯一标识                             |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;annexName                  | 附件名称                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;annexType                  | 附件类型：0001-个人征信授权书等,可用值:TYPE0,TYPE1,TYPE2,TYPE3,TYPE4,TYPE5,TYPE6 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;ext                        | 扩展字段                                                     |          | false    | string                    |                           |
| &emsp;&emsp;applyRate                              | 融资申请利率                                                 |          | true     | number                    |                           |
| &emsp;&emsp;certCode                               | 客户证件号                                                   |          | true     | string                    |                           |
| &emsp;&emsp;certType                               | 客户证件类型                                                 |          | true     | string                    |                           |
| &emsp;&emsp;clientPlatformBid                      | 产融平台bid                                                  |          | true     | string                    |                           |
| &emsp;&emsp;contractInfoList                       | 合同信息列表                                                 |          | true     | array                     | ExContractInputForm       |
| &emsp;&emsp;&emsp;&emsp;annexLabel                 | 文件唯一标识                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;buyerName                  | 买方名称                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;contractAmount             | 合同金额:分                                                  |          | false    | number                    |                           |
| &emsp;&emsp;&emsp;&emsp;contractCode               | 商务合同编号                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;contractName               | 商务合同名称                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;endDate                    | 合同截止日期精确到日                                         |          | false    | integer                   |                           |
| &emsp;&emsp;&emsp;&emsp;sellerName                 | 卖方名称                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;signDate                   | 合同签署日期精确到日                                         |          | false    | integer                   |                           |
| &emsp;&emsp;customBid                              | 客户Bid                                                      |          | true     | string                    |                           |
| &emsp;&emsp;customName                             | 客户名称                                                     |          | true     | string                    |                           |
| &emsp;&emsp;dueDate                                | 融资到期日                                                   |          | true     | integer(int64)            |                           |
| &emsp;&emsp;financeNo                              | 融资申请单号                                                 |          | true     | string                    |                           |
| &emsp;&emsp;financingAmount                        | 融资申请金额（元）                                           |          | true     | number                    |                           |
| &emsp;&emsp;financingRatio                         | 融资比例                                                     |          | true     | number                    |                           |
| &emsp;&emsp;interestPaymentCode                    | 付息方式 1-卖方付息 2-买方付息,可用值:TYPE1,TYPE2            |          | true     | string                    |                           |
| &emsp;&emsp;invoiceInfoList                        | 发票信息列表                                                 |          | true     | array                     | ExInvoiceInputForm        |
| &emsp;&emsp;&emsp;&emsp;applyAmount                | 本次融资申请占用金额（单位：分）                             |          | false    | number                    |                           |
| &emsp;&emsp;&emsp;&emsp;invoiceHash                | 发票Hash                                                     |          | false    | string                    |                           |
| &emsp;&emsp;loanAccountBid                         | 融资申请人用于放款的账户BID                                  |          | true     | string                    |                           |
| &emsp;&emsp;merchantId                             | 商户号                                                       |          | true     | string                    |                           |
| &emsp;&emsp;nonce                                  | 产融平台bid指定区块链交易的nonce值                           |          | true     | integer(int64)            |                           |
| &emsp;&emsp;nonceStr                               | 随机字符串                                                   |          | true     | string                    |                           |
| &emsp;&emsp;phone                                  | 联系电话                                                     |          | true     | string                    |                           |
| &emsp;&emsp;repaymentAccountBid                    | 融资申请人用于还款的账户BID                                  |          | true     | string                    |                           |
| &emsp;&emsp;requestNo                              | 请求号                                                       |          | true     | string                    |                           |
| &emsp;&emsp;serviceFee                             | 代收手续费信息                                               |          | true     | ExServiceFeeInfoInputForm | ExServiceFeeInfoInputForm |
| &emsp;&emsp;&emsp;&emsp;amount                     | 代收手续费金额（单位：元）                                   |          | false    | number                    |                           |
| &emsp;&emsp;&emsp;&emsp;fromAccountBid             | 代收手续费From账户BID                                        |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;toAccountBid               | 代收手续费To账户BID                                          |          | false    | string                    |                           |
| &emsp;&emsp;sign                                   | 签名串                                                       |          | true     | string                    |                           |
| &emsp;&emsp;signType                               | 签名方式                                                     |          | true     | string                    |                           |
| &emsp;&emsp;validateInfo                           | 主体确权信息（对融资申请基础信息进行签名）                   |          | false    | ExValidateInfoInputForm   | ExValidateInfoInputForm   |
| &emsp;&emsp;&emsp;&emsp;hash                       | 区块链交易hash，当确权类型为“区块链交易自带签名”时使用，合约自动取值 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;originalData               | 签名原文：当确权类型为“BID私钥签名”/“CA证书签名”时使用       |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;publicKey                  | 签名公钥：当确权类型为“BID私钥签名”时使用                    |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;signature                  | 签名结果：当确权类型为“BID私钥签名”/“CA证书签名”时使用       |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;smartContractBid           | 业务合约BID，当确权类型为“业务合约衍生”时使用                |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;smartContractLabel         | 在业务合约中的标识，使用时根据合约类型进一步解析业务，当确权类型为“业务合约衍生”时使用 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;validateType               | 确权类型：无/BID私钥签名/CA证书签名/业务合约衍生,可用值:BID_PRIVATE_KEY,BUSINESS_CONSTART,CA_PRIVATE_KEY,CURRENT_TX,NONE |          | false    | string                    |                           |
| &emsp;&emsp;voucherAmount                          | 上一首凭证金额（元）                                         |          | true     | number                    |                           |
| &emsp;&emsp;voucherInfoList                        | 溯源凭证信息                                                 |          | true     | array                     | ExVoucherInfoInputForm    |
| &emsp;&emsp;&emsp;&emsp;amount                     | 凭证金额:元                                                  |          | false    | number                    |                           |
| &emsp;&emsp;&emsp;&emsp;cashTime                   | 兑付时间 精确到日                                            |          | false    | integer                   |                           |
| &emsp;&emsp;&emsp;&emsp;contractInfoList           | 该凭证合同信息                                               |          | false    | array                     | ExContractInputForm       |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;annexLabel     | 文件唯一标识                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;buyerName      | 买方名称                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;contractAmount | 合同金额:分                                                  |          | false    | number                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;contractCode   | 商务合同编号                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;contractName   | 商务合同名称                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;endDate        | 合同截止日期精确到日                                         |          | false    | integer                   |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;sellerName     | 卖方名称                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;signDate       | 合同签署日期精确到日                                         |          | false    | integer                   |                           |
| &emsp;&emsp;&emsp;&emsp;createTime                 | 凭证创建时间 精确到日                                        |          | false    | integer                   |                           |
| &emsp;&emsp;&emsp;&emsp;invoiceInfoList            | 该凭证发票信息                                               |          | false    | array                     | ExInvoiceInputForm        |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;applyAmount    | 本次融资申请占用金额（单位：分）                             |          | false    | number                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;invoiceHash    | 发票Hash                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;parentVoucherNo            | 上级凭证编号                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;payee                      | 该凭证收款方信息                                             |          | false    | ExCompanyInfoInputForm    | ExCompanyInfoInputForm    |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;certNo         | 证件号码                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;certType       | 企业证件类型                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;companyName    | 企业名称                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;phone          | 手机号                                                       |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;payer                      | 该凭证付款方信息                                             |          | false    | ExCompanyInfoInputForm    | ExCompanyInfoInputForm    |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;certNo         | 证件号码                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;certType       | 企业证件类型                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;companyName    | 企业名称                                                     |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;phone          | 手机号                                                       |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;rootVouchNo                | 原始凭证编号                                                 |          | false    | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;voucherNo                  | 凭证编号                                                     |          | false    | string                    |                           |
| &emsp;&emsp;voucherNo                              | 上一首凭证编号                                               |          | true     | string                    |                           |


**响应状态**:


| 状态码 | 说明         | schema                   |
| ------ | ------------ | ------------------------ |
| 200    | OK           | Result«FinancePrepareVO» |
| 201    | Created      |                          |
| 401    | Unauthorized |                          |
| 403    | Forbidden    |                          |
| 404    | Not Found    |                          |


**响应参数**:


| 参数名称         | 参数说明 | 类型             | schema           |
| ---------------- | -------- | ---------------- | ---------------- |
| code             |          | string           |                  |
| data             |          | FinancePrepareVO | FinancePrepareVO |
| &emsp;&emsp;blob | 待签数据 | string           |                  |
| &emsp;&emsp;hash | 交易hash | string           |                  |
| message          |          | string           |                  |
| total            |          | integer(int32)   | integer(int32)   |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"blob": "",
		"hash": ""
	},
	"message": "",
	"total": 0
}
```


##### 融资提交


**接口地址**:`/if-finance/api/finance/v1/finance/submit`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`


**接口描述**:<p>融资提交</p>



**请求示例**:


```javascript
{
  "blob": "",
  "clientPlatformBid": "",
  "hash": "",
  "merchantId": "",
  "nonceStr": "",
  "publicKey": "",
  "requestNo": "",
  "sign": "",
  "signStr": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明    | 请求类型 | 是否必须 | 数据类型               | schema                 |
| ----------------------------- | ----------- | -------- | -------- | ---------------------- | ---------------------- |
| input                         | input       | body     | true     | FinanceSubmitInputForm | FinanceSubmitInputForm |
| &emsp;&emsp;blob              | 待签数据    |          | false    | string                 |                        |
| &emsp;&emsp;clientPlatformBid | 产融平台bid |          | true     | string                 |                        |
| &emsp;&emsp;hash              | hash        |          | false    | string                 |                        |
| &emsp;&emsp;merchantId        | 商户号      |          | true     | string                 |                        |
| &emsp;&emsp;nonceStr          | 随机字符串  |          | true     | string                 |                        |
| &emsp;&emsp;publicKey         | 公钥        |          | false    | string                 |                        |
| &emsp;&emsp;requestNo         | 请求号      |          | true     | string                 |                        |
| &emsp;&emsp;sign              | 签名串      |          | true     | string                 |                        |
| &emsp;&emsp;signStr           | 签名串      |          | false    | string                 |                        |
| &emsp;&emsp;signType          | 签名方式    |          | true     | string                 |                        |


**响应状态**:


| 状态码 | 说明         | schema                  |
| ------ | ------------ | ----------------------- |
| 200    | OK           | Result«FinanceSubmitVO» |
| 201    | Created      |                         |
| 401    | Unauthorized |                         |
| 403    | Forbidden    |                         |
| 404    | Not Found    |                         |


**响应参数**:


| 参数名称                    | 参数说明 | 类型            | schema          |
| --------------------------- | -------- | --------------- | --------------- |
| code                        |          | string          |                 |
| data                        |          | FinanceSubmitVO | FinanceSubmitVO |
| &emsp;&emsp;contractAddress | 合约地址 | string          |                 |
| &emsp;&emsp;label           | 唯一标识 | string          |                 |
| message                     |          | string          |                 |
| total                       |          | integer(int32)  | integer(int32)  |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"contractAddress": "",
		"label": ""
	},
	"message": "",
	"total": 0
}
```


##### 融资指令转发


**接口地址**:`/if-finance/api/finance/v1/finance/transmit`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>融资指令转发</p>

**请求示例**:


```javascript
{
  "clientPlatformBid": "",
  "contractAddress": "",
  "hash": "",
  "label": "",
  "merchantId": "",
  "nonceStr": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明    | 请求类型 | 是否必须 | 数据类型                 | schema                   |
| ----------------------------- | ----------- | -------- | -------- | ------------------------ | ------------------------ |
| input                         | input       | body     | true     | FinanceTransmitInputForm | FinanceTransmitInputForm |
| &emsp;&emsp;clientPlatformBid | 产融平台bid |          | true     | string                   |                          |
| &emsp;&emsp;contractAddress   | 合约地址    |          | false    | string                   |                          |
| &emsp;&emsp;hash              | 交易hash    |          | false    | string                   |                          |
| &emsp;&emsp;label             | 唯一标识    |          | false    | string                   |                          |
| &emsp;&emsp;merchantId        | 商户号      |          | true     | string                   |                          |
| &emsp;&emsp;nonceStr          | 随机字符串  |          | true     | string                   |                          |
| &emsp;&emsp;requestNo         | 请求号      |          | true     | string                   |                          |
| &emsp;&emsp;sign              | 签名串      |          | true     | string                   |                          |
| &emsp;&emsp;signType          | 签名方式    |          | true     | string                   |                          |


**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | Result«FinanceTransmitVO» |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称           | 参数说明 | 类型              | schema            |
| ------------------ | -------- | ----------------- | ----------------- |
| code               |          | string            |                   |
| data               |          | FinanceTransmitVO | FinanceTransmitVO |
| &emsp;&emsp;status | 状态     | string            |                   |
| message            |          | string            |                   |
| total              |          | integer(int32)    | integer(int32)    |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"status": ""
	},
	"message": "",
	"total": 0
}
```


#### 融资服务：查询操作


##### 融资成本试算


**接口地址**:`/if-finance/api/finance/v1/finance-query/finance-fee`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>融资成本试算</p>

**接口功能描述**:针对已经开通完融资产品的调用方，可以调用此接口，根据融资相关数据，计算本次融资所需的融资成本。

**请求示例**:


```javascript
{
  "clientPlatformBid": "",
  "custName": "",
  "custNo": "",
  "dueDate": "",
  "interestPaymentCode": "",
  "invoiceNum": "",
  "invoiceTotalAmount": "",
  "isRttm": "",
  "loanAmt": "",
  "merchantId": "",
  "requestNo": "",
  "scoCod": "",
  "scoName": ""
}
```


**请求参数**:


| 参数名称                        | 参数说明                                       | 请求类型 | 是否必须 | 数据类型                 | schema                   |
| ------------------------------- | ---------------------------------------------- | -------- | -------- | ------------------------ | ------------------------ |
| input                           | input                                          | body     | true     | QueryFinanceFeeInputForm | QueryFinanceFeeInputForm |
| &emsp;&emsp;clientPlatformBid   | 产融平台bid                                    |          | true     | string                   |                          |
| &emsp;&emsp;custName            | 供应商企业名称(中信)                           |          | true     | string                   |                          |
| &emsp;&emsp;custNo              | 【民生:客户平台编号;招商:供应商证件号码;中信】 |          | true     | string                   |                          |
| &emsp;&emsp;dueDate             | 账款到期日(招商：单据最晚付款日期)             |          | true     | string                   |                          |
| &emsp;&emsp;interestPaymentCode |                                                |          | false    | string(byte)             |                          |
| &emsp;&emsp;invoiceNum          | 发票合计数量【民生为空，招商必填,上海为空】    |          | true     | string                   |                          |
| &emsp;&emsp;invoiceTotalAmount  | 发票合计金额【民生为空，招商必填,上海为空】    |          | true     | string                   |                          |
| &emsp;&emsp;isRttm              | 利率档期Y01-一年(中信)                         |          | true     | string                   |                          |
| &emsp;&emsp;loanAmt             | 申请融资金额：元                               |          | true     | string                   |                          |
| &emsp;&emsp;merchantId          | 商户号                                         |          | true     | string                   |                          |
| &emsp;&emsp;requestNo           | 请求号                                         |          | true     | string                   |                          |
| &emsp;&emsp;scoCod              | 核心企业证件号码                               |          | true     | string                   |                          |
| &emsp;&emsp;scoName             | 核心企业客户名称(中信)                         |          | true     | string                   |                          |


**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | Result«QueryFinanceFeeVO» |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称                | 参数说明       | 类型              | schema            |
| ----------------------- | -------------- | ----------------- | ----------------- |
| code                    |                | string            |                   |
| data                    |                | QueryFinanceFeeVO | QueryFinanceFeeVO |
| &emsp;&emsp;interestFee | 融资利率费:元  | number            |                   |
| &emsp;&emsp;serviceFee  | 融资服务费：元 | number            |                   |
| message                 |                | string            |                   |
| total                   |                | integer(int32)    | integer(int32)    |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"interestFee": 0,
		"serviceFee": 0
	},
	"message": "",
	"total": 0
}
```


##### 融资结果查询


**接口地址**:`/if-finance/api/finance/v1/finance-query/loan`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`


**接口描述**:<p>融资结果查询</p>

**接口功能描述**:针对已有明确融资结果（已放款、审批退回、还款中、已还款）的融资数据，可以根据此接口查询融资放款的数据。

**请求示例**:


```javascript
{
  "clientPlatformBid": "",
  "requestNo": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明    | 请求类型 | 是否必须 | 数据类型         | schema           |
| ----------------------------- | ----------- | -------- | -------- | ---------------- | ---------------- |
| query                         | query       | body     | true     | FinanceLoanQuery | FinanceLoanQuery |
| &emsp;&emsp;clientPlatformBid | 产融平台Bid |          | true     | string           |                  |
| &emsp;&emsp;requestNo         | 请求号      |          | true     | string           |                  |


**响应状态**:


| 状态码 | 说明         | schema                |
| ------ | ------------ | --------------------- |
| 200    | OK           | Result«FinanceLoanVO» |
| 201    | Created      |                       |
| 401    | Unauthorized |                       |
| 403    | Forbidden    |                       |
| 404    | Not Found    |                       |


**响应参数**:


| 参数名称                       | 参数说明           | 类型           | schema         |
| ------------------------------ | ------------------ | -------------- | -------------- |
| code                           |                    | string         |                |
| data                           |                    | FinanceLoanVO  | FinanceLoanVO  |
| &emsp;&emsp;amount             | 融资金额，单位：分 | integer(int64) |                |
| &emsp;&emsp;dueDate            | 融资到期日期       | string         |                |
| &emsp;&emsp;financingNo        | 融资申请编号       | string         |                |
| &emsp;&emsp;financingTimeLimit | 融资期限，单位：天 | integer(int32) |                |
| &emsp;&emsp;fundingLoanNo      | 行方业务编号       | string         |                |
| &emsp;&emsp;interest           | 利息，单位：分     | integer(int64) |                |
| &emsp;&emsp;loanAmount         | 实际放款金额       | number         |                |
| &emsp;&emsp;loanDate           | 放款日期           | string         |                |
| &emsp;&emsp;rate               | 利率，单位：分     | integer(int64) |                |
| &emsp;&emsp;refuseReason       | 失败原因           | string         |                |
| &emsp;&emsp;requestNo          | 请求号             | string         |                |
| &emsp;&emsp;status             | 融资处理结果       | string         |                |
| message                        |                    | string         |                |
| total                          |                    | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"amount": 0,
		"dueDate": "",
		"financingNo": "",
		"financingTimeLimit": 0,
		"fundingLoanNo": "",
		"interest": 0,
		"loanAmount": 0,
		"loanDate": "",
		"rate": 0,
		"refuseReason": "",
		"requestNo": "",
		"status": ""
	},
	"message": "",
	"total": 0
}
```


##### 还款结果查询


**接口地址**:`/if-finance/api/finance/v1/finance-query/repayment`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>还款结果查询</p>

**接口功能描述**:针对已还款的融资数据，可以根据此接口查询融资还款数据。

**请求示例**:


```javascript
{
  "clientPlatformBid": "",
  "requestNo": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明    | 请求类型 | 是否必须 | 数据类型              | schema                |
| ----------------------------- | ----------- | -------- | -------- | --------------------- | --------------------- |
| query                         | query       | body     | true     | FinanceRepaymentQuery | FinanceRepaymentQuery |
| &emsp;&emsp;clientPlatformBid | 产融平台Bid |          | true     | string                |                       |
| &emsp;&emsp;requestNo         | 请求号      |          | true     | string                |                       |


**响应状态**:


| 状态码 | 说明         | schema                     |
| ------ | ------------ | -------------------------- |
| 200    | OK           | Result«FinanceRepaymentVO» |
| 201    | Created      |                            |
| 401    | Unauthorized |                            |
| 403    | Forbidden    |                            |
| 404    | Not Found    |                            |


**响应参数**:


| 参数名称                  | 参数说明     | 类型               | schema             |
| ------------------------- | ------------ | ------------------ | ------------------ |
| code                      |              | string             |                    |
| data                      |              | FinanceRepaymentVO | FinanceRepaymentVO |
| &emsp;&emsp;financingNo   | 融资申请编号 | string             |                    |
| &emsp;&emsp;repaymentTime | 还款时间     | string             |                    |
| &emsp;&emsp;requestNo     | 请求号       | string             |                    |
| &emsp;&emsp;status        | 还款处理结果 | string(byte)       |                    |
| message                   |              | string             |                    |
| total                     |              | integer(int32)     | integer(int32)     |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"financingNo": "",
		"repaymentTime": "",
		"requestNo": "",
		"status": ""
	},
	"message": "",
	"total": 0
}
```


##### 开通信息查询


**接口地址**:`/if-finance/api/finance/v1/custom/open-finance-query`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`


**接口描述**:<p>开通信息查询</p>

**接口功能描述**:调用开通产品接口之后，可调用此接口查询开通融资产品的相关信息。如果开通融资产品还未成功或者还未调用过开通融资产品 ，此接口返回null

**请求示例**:


```javascript
{
  "clientPlatformBid": "",
  "requestNo": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明    | 请求类型 | 是否必须 | 数据类型         | schema           |
| ----------------------------- | ----------- | -------- | -------- | ---------------- | ---------------- |
| query                         | query       | body     | true     | OpenFinanceQuery | OpenFinanceQuery |
| &emsp;&emsp;clientPlatformBid | 产融平台bid |          | false    | string           |                  |
| &emsp;&emsp;requestNo         | 请求号      |          | true     | string           |                  |


**响应状态**:


| 状态码 | 说明         | schema                           |
| ------ | ------------ | -------------------------------- |
| 200    | OK           | Result«OpenFinanceQueryOutputVO» |
| 201    | Created      |                                  |
| 401    | Unauthorized |                                  |
| 403    | Forbidden    |                                  |
| 404    | Not Found    |                                  |


**响应参数**:


| 参数名称                        | 参数说明                 | 类型                     | schema                   |
| ------------------------------- | ------------------------ | ------------------------ | ------------------------ |
| code                            |                          | string                   |                          |
| data                            |                          | OpenFinanceQueryOutputVO | OpenFinanceQueryOutputVO |
| &emsp;&emsp;availableCreditLine | 剩余授信额度             | integer(int64)           |                          |
| &emsp;&emsp;bankAccountName     | 供应商账户名称           | string                   |                          |
| &emsp;&emsp;bankAccountNo       | 供应商账户               | string                   |                          |
| &emsp;&emsp;bankCode            | 银行编码                 | string                   |                          |
| &emsp;&emsp;companyName         | 企业名称                 | string                   |                          |
| &emsp;&emsp;contractNo          | 交易合同编号             | string                   |                          |
| &emsp;&emsp;createTime          | 创建时间                 | integer(int64)           |                          |
| &emsp;&emsp;creditLine          | 授信额度                 | integer(int64)           |                          |
| &emsp;&emsp;custNo              | 平台编号社会统一信用代码 | string                   |                          |
| &emsp;&emsp;factFeeRate         | 保理手续费率             | integer(int32)           |                          |
| &emsp;&emsp;merchantId          | 授信机构ID               | string                   |                          |
| &emsp;&emsp;rateMax             | 利率最大值               | integer(int32)           |                          |
| &emsp;&emsp;rateMin             | 利率最小值               | integer(int32)           |                          |
| &emsp;&emsp;ratioMax            | 融资比率最大值           | integer(int32)           |                          |
| &emsp;&emsp;ratioMin            | 融资比率最小值           | integer(int32)           |                          |
| &emsp;&emsp;repayType           | 还款方式(计费融资)       | string(byte)             |                          |
| &emsp;&emsp;status              | 生效状态                 | string                   |                          |
| &emsp;&emsp;t24Id               | t24id                    | string                   |                          |
| &emsp;&emsp;timeLimitMax        | 融资期限最大值           | integer(int32)           |                          |
| &emsp;&emsp;timeLimitMin        | 融资期限最小值           | integer(int32)           |                          |
| &emsp;&emsp;validBeginTime      | 生效开始时间             | integer(int64)           |                          |
| &emsp;&emsp;validEndTime        | 生效结束时间             | integer(int64)           |                          |
| &emsp;&emsp;validTime           | 授信有效期(天)           | integer(int64)           |                          |
| message                         |                          | string                   |                          |
| total                           |                          | integer(int32)           | integer(int32)           |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"availableCreditLine": 0,
		"bankAccountName": "",
		"bankAccountNo": "",
		"bankCode": "",
		"companyName": "",
		"contractNo": "",
		"createTime": 0,
		"creditLine": 0,
		"custNo": "",
		"factFeeRate": 0,
		"merchantId": "",
		"rateMax": 0,
		"rateMin": 0,
		"ratioMax": 0,
		"ratioMin": 0,
		"repayType": "",
		"status": "",
		"t24Id": "",
		"timeLimitMax": 0,
		"timeLimitMin": 0,
		"validBeginTime": 0,
		"validEndTime": 0,
		"validTime": 0
	},
	"message": "",
	"total": 0
}
```


##### 链上查询操作


**接口地址**:`/if-finance/api/finance/v1/bc-query/query-status`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>链上查询操作</p>

**请求示例**:


```javascript
{
  "hash": "",
  "requestNo": ""
}
```


**请求参数**:


| 参数名称              | 参数说明 | 请求类型 | 是否必须 | 数据类型             | schema               |
| --------------------- | -------- | -------- | -------- | -------------------- | -------------------- |
| query                 | query    | body     | true     | BcOperateStatusQuery | BcOperateStatusQuery |
| &emsp;&emsp;hash      | 交易hash |          | true     | string               |                      |
| &emsp;&emsp;requestNo | 请求号   |          | true     | string               |                      |

**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | Result«BcOperateStatusVO» |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称           | 参数说明                             | 类型              | schema            |
| ------------------ | ------------------------------------ | ----------------- | ----------------- |
| code               |                                      | string            |                   |
| data               |                                      | BcOperateStatusVO | BcOperateStatusVO |
| &emsp;&emsp;status | 状态 1-初始化 2-成功 3-失败 4-上链中 | string(byte)      |                   |
| message            |                                      | string            |                   |
| total              |                                      | integer(int32)    | integer(int32)    |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"status": ""
	},
	"message": "",
	"total": 0
}
```

##### 查询指定账户当前实时nonce值

**服务调用**:当前接口获取到的nonce值，+1之后，作为入参调用融资提交准备接口

**接口地址**:`/if-finance/api/finance/v1/blockchain/get-nonce`

**请求方式**:`POST`

**请求数据类型**:`application/json`


**响应数据类型**:`*/*`


**接口描述**:


**请求示例**:


```javascript
{
  "accountAddress": "",
  "requestNo": ""
}
```


**请求参数**:


| 参数名称                   | 参数说明           | 请求类型 | 是否必须 | 数据类型          | schema            |
| -------------------------- | ------------------ | -------- | -------- | ----------------- | ----------------- |
| input                      | input              | body     | true     | AccountNonceQuery | AccountNonceQuery |
| &emsp;&emsp;accountAddress | 待查询的区块链地址 |          | true     | string            |                   |
| &emsp;&emsp;requestNo      | 请求号             |          | true     | string            |                   |


**响应状态**:


| 状态码 | 说明         | schema       |
| ------ | ------------ | ------------ |
| 200    | OK           | Result«long» |
| 201    | Created      |              |
| 401    | Unauthorized |              |
| 403    | Forbidden    |              |
| 404    | Not Found    |              |


**响应参数**:


| 参数名称 | 参数说明 | 类型           | schema         |
| -------- | -------- | -------------- | -------------- |
| code     |          | string         |                |
| data     |          | integer(int64) | integer(int64) |
| message  |          | string         |                |
| total    |          | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": 0,
	"message": "",
	"total": 0
}
```

### 2.9.清结算服务

**简介**:用于产融清结算信息操作，包含清结算信息预览，清算支付，清算调账等操作。

#### 状态机

<img src="../_static/images/image5.png"/>

#### 操作类接口调用流程

- 步骤1

  清结算提交准备接口，涉及到资金清算路径收付款账户的入参，需使用账户服务登记过的子账户或结算户bid，目前仅子账户可在申请清结算的时候使用线上清算（payMode为ONLINE）功能，结算户仅可使用线下结算（payMode为OFFLINE）的功能。

  清结算提交准备接口，入参nonce值由第三方产融平台自行维护，可通过辅助接口：/api/settlement/v1/blockchain/get-nonce获取指定账户在链上的实时nonce值，+1后作为入参。

  清结算提交准备接口，返回的待签名数据需使用clientPlatformBid对应账户私钥进行数据签名。

  入参有ISSUE类业务登记时，认为是一笔新的清结算业务登记，数据提交成功后会返回该笔业务对应的合约地址+业务唯一标识label。若要基于此业务追加非ISSUE的资产流转，需指定“合约地址+业务唯一标识label”，使用步骤1继续追加。

  步骤1完成后，需等待区块链交易执行结果，区块链交易执行成功，该操作成功，业务状态为“业务上链”。

  <img src="../_static/images/image6.png"/>

- 步骤2

  申请清结算接口，依赖步骤1得到的“合约地址+业务唯一标识label.

  申请清结算接口，入参nonce值由第三方产融平台自行维护，可通过辅助接口：/api/settlement/v1/blockchain/get-nonce获取指定账户在链上的实时nonce值，+1后作为入参。

  申请清结算接口，返回的待签名数据需使用clientPlatformBid对应账户私钥进行数据签名。

  步骤2完成后，需等待区块链交易执行结果，区块链交易执行成功，该操作成功，业务状态为“申请清结算。

  <img src="../_static/images/image7.png"/>

- 步骤3

  开始清结算接口，依赖步骤1得到的“合约地址+业务唯一标识label”。

  若选择子账户+线上清算模式，需提前调用账户服务的子账户余额查询接口，确保ISSUE的资金清算路径from账户有足够的余额进行兑付。

  步骤3完成后，需等待区块链交易执行结果，区块链交易执行成功，该操作成功，业务状态为“开始清结算”。

  清结算服务会对该笔业务进行清结算处理，处理进度可通过“清结算结果查询”接口进行查询。

  <img src="../_static/images/image8.png"/>

  <img src="../_static/images/image9.png"/>

  

  

#### 清结算服务：业务操作

##### 清算提交准备


**接口地址**:`/if-settlement/api/settlement/v1/settlement/prepare`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>清算提交准备-合约</p>

**请求示例**:


```javascript
{
  "clientPlatformBid": "",
  "contractAddress": "",
  "contractInputFormMap": {
    "additionalProperties1": {
      "dataVersion": "",
      "finance": {
        "amount": 0,
        "cashTime": 0,
        "dataVersion": "",
        "ext": "",
        "financeAmount": 0,
        "financeMode": "",
        "financeRatio": 0,
        "from": "",
        "interestRepaid": 0,
        "loanAmount": 0,
        "loanStatus": true,
        "loanTime": 0,
        "loanType": "",
        "principalRepaid": 0,
        "rate": 0,
        "repayType": "",
        "serviceCommissionAmount": 0,
        "serviceCommissionFromAccBid": "",
        "serviceCommissionToAccBid": "",
        "settlementType": "",
        "timeLimit": 0,
        "to": "",
        "type": "",
        "voucherAmount": 0,
        "voucherNo": ""
      },
      "pub": {
        "action": "",
        "no": "",
        "parentNo": "",
        "type": ""
      },
      "transaction": {
        "amount": 0,
        "from": "",
        "to": ""
      },
      "validateInfo": [
        {
          "originalData": "",
          "publicKey": "",
          "signature": "",
          "smartContractBid": "",
          "smartContractLabel": "",
          "txHash": "",
          "validateType": ""
        }
      ],
      "voucher": {
        "amount": 0,
        "amountPaid": 0,
        "cashTime": 0,
        "dataVersion": "",
        "ext": "",
        "from": "",
        "serviceCommissionAmount": 0,
        "serviceCommissionFromAccBid": "",
        "serviceCommissionToAccBid": "",
        "settlementType": "",
        "to": "",
        "type": ""
      }
    }
  },
  "label": "",
  "merchantId": "",
  "nonce": 0,
  "nonceStr": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                                                     | 参数说明                                                     | 请求类型 | 是否必须 | 数据类型                   | schema                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------- | -------- | -------------------------- | -------------------------- |
| input                                                        | input                                                        | body     | true     | SettlementPrepareInputForm | SettlementPrepareInputForm |
| &emsp;&emsp;clientPlatformBid                                | 产融平台bid                                                  |          | true     | string                     |                            |
| &emsp;&emsp;contractAddress                                  | 合约地址                                                     |          | true     | string                     |                            |
| &emsp;&emsp;contractInputFormMap                             | 合约入参                                                     |          | false    | BaseContractInputForm      | BaseContractInputForm      |
| &emsp;&emsp;&emsp;&emsp;dataVersion                          | 数据版本,可用值:V1                                           |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;finance                              |                                                              |          | false    | DetailFinancePrivInputForm | DetailFinancePrivInputForm |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;amount                   | 资金清算的确切金额（分）                                     |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;cashTime                 | 承诺付款日期（精确到日，毫秒级时间戳）                       |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;dataVersion              | 数据版本,可用值:V1                                           |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ext                      | 扩展                                                         |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;financeAmount            | 融资金额（分）                                               |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;financeMode              | 融资模式：ASSIGNMENT_FINANCING 转让融资；PLEDGE_FINANCING 质押融资,可用值:ASSIGNMENT_FINANCING,PLEDGE_FINANCING |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;financeRatio             | 融资比率（ppm, 百万分之（几），1ppm=0.0001%）                |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;from                     | 付款账户地址（资金清算路径付款账户）                         |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;interestRepaid           | 已还利息（分）                                               |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;loanAmount               | 放款金额（分）                                               |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;loanStatus               | 放款状态：false-未放款；true-已放款                          |          | false    | boolean                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;loanTime                 | 放款时间（精确到日，毫秒级时间戳）                           |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;loanType                 | 放款方式：ONLINE_PAY_TYPE 线上放款；OFFLINE_PAY_TYPE 线下放款,可用值:OFFLINE_PAY_TYPE,ONLINE_PAY_TYPE |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;principalRepaid          | 已还本金（分）                                               |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;rate                     | 利率（ppm，百万分之（几），1ppm=0.0001%）                    |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;repayType                | 还款方式：TYPE1-一次性还本付息；TYPE2-计费融资,可用值:TYPE1,TYPE2 |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;serviceCommissionAmount  | 代收手续费金额（分，从from到代收手续费账户）                 |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;serviceCommissionFromAccBid | 代收手续费From账户BID                                        |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;serviceCommissionToAccBid | 代收手续费To账户BID                                          |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;settlementType           | 清算方式：NONE 不清算；TRANSFER 转账；WITHDRAW 提现,可用值:NONE,TRANSFER,WITHDRAW |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;timeLimit                | 融资期限（天）                                               |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;to                       | 收款账户地址（资金清算路径收款账户）                         |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;type                     | 资金业务类型：贸易金额、融资本金、融资利息等,可用值:TYPE1,TYPE2,TYPE3 |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;voucherAmount            | 凭证金额（分）                                               |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;voucherNo                | 融资凭证编号/质押物凭证编号                                  |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;pub                                  | pub                                                          |          | false    | DetailPubInputForm         | DetailPubInputForm         |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;action                   | 业务动作：ISSUE/TRANSFER_VOUCHER/TRANSFER_FINANCE,可用值:ISSUE,REVOKE_TRANSFER_ABN,TRANSFER_ABN,TRANSFER_FINANCE,TRANSFER_VOUCHER |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;no                       | 业务编号                                                     |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;parentNo                 | 父业务编号                                                   |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;type                     | 资产类型：ACCOUNTS_RECEIVABLE-应收账款；ORDER-订单,可用值:ACCOUNTS_RECEIVABLE,ORDER |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;transaction                          | transaction                                                  |          | false    | TransactionInputForm       | TransactionInputForm       |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;amount                   | 资产额度                                                     |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;from                     | 资产转出主体BID                                              |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;to                       | 资产转入主体BID                                              |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;validateInfo                         | validateInfo                                                 |          | false    | array                      | ValidateInfoInputForm      |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;originalData             | 签名原文：当确权类型为“BID私钥签名”/“CA证书签名”时使用       |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;publicKey                | 签名公钥：当确权类型为“BID私钥签名”时使用                    |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;signature                | 签名结果：当确权类型为“BID私钥签名”/“CA证书签名”时使用       |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;smartContractBid         | 业务合约BID，当确权类型为“业务合约衍生”时使用                |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;smartContractLabel       | 在业务合约中的标识，使用时根据合约类型进一步解析业务，当确权类型为“业务合约衍生”时使用 |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;txHash                   | 区块链交易hash，当确权类型为“区块链交易自带签名”时使用，合约自动取值 |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;validateType             | 确权类型：无/BID私钥签名/CA证书签名/业务合约衍生,可用值:BID_PRIVATE_KEY,BUSINESS_CONSTART,CA_PRIVATE_KEY,CURRENT_TX,NONE |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;voucher                              |                                                              |          | false    | DetailVoucherPrivInputForm | DetailVoucherPrivInputForm |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;amount                   | 资金清算的确切金额（分）                                     |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;amountPaid               | 已付金额（分）                                               |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;cashTime                 | 承诺付款日期（精确到日，毫秒级时间戳）                       |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;dataVersion              | 数据版本,可用值:V1                                           |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;ext                      | 扩展                                                         |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;from                     | 付款账户地址（资金清算路径付款账户）                         |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;serviceCommissionAmount  | 代收手续费金额（分，从from到代收手续费账户）                 |          | false    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;serviceCommissionFromAccBid | 代收手续费From账户BID                                        |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;serviceCommissionToAccBid | 代收手续费To账户BID                                          |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;settlementType           | 清算方式：NONE 不清算；TRANSFER 转账；WITHDRAW 提现,可用值:NONE,TRANSFER,WITHDRAW |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;to                       | 收款账户地址（资金清算路径收款账户）                         |          | false    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;type                     | 资金业务类型：贸易金额、融资本金、融资利息等,可用值:TYPE1,TYPE2,TYPE3 |          | false    | string                     |                            |
| &emsp;&emsp;label                                            | 唯一标识                                                     |          | true     | string                     |                            |
| &emsp;&emsp;merchantId                                       | 商户号                                                       |          | true     | string                     |                            |
| &emsp;&emsp;nonce                                            | 产融平台Bid指定区块链交易的nonce值                           |          | true     | integer(int64)             |                            |
| &emsp;&emsp;nonceStr                                         | 随机字符串                                                   |          | true     | string                     |                            |
| &emsp;&emsp;requestNo                                        | 请求号                                                       |          | true     | string                     |                            |
| &emsp;&emsp;sign                                             | 签名串                                                       |          | true     | string                     |                            |
| &emsp;&emsp;signType                                         | 签名方式                                                     |          | true     | string                     |                            |


**响应状态**:


| 状态码 | 说明         | schema                      |
| ------ | ------------ | --------------------------- |
| 200    | OK           | Result«SettlementPrepareVO» |
| 201    | Created      |                             |
| 401    | Unauthorized |                             |
| 403    | Forbidden    |                             |
| 404    | Not Found    |                             |


**响应参数**:


| 参数名称         | 参数说明 | 类型                | schema              |
| ---------------- | -------- | ------------------- | ------------------- |
| code             |          | string              |                     |
| data             |          | SettlementPrepareVO | SettlementPrepareVO |
| &emsp;&emsp;blob | 待签数据 | string              |                     |
| &emsp;&emsp;hash | 交易hash | string              |                     |
| message          |          | string              |                     |
| total            |          | integer(int32)      | integer(int32)      |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"blob": "",
		"hash": ""
	},
	"message": "",
	"total": 0
}
```


##### 申请清结算-准备阶段


**接口地址**:`/if-settlement/api/settlement/v1/settlement/apply`


**请求方式**:`POST`

**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>申请清结算-准备阶段</p>

**请求示例**:


```javascript
{
  "clientPlatformBid": "",
  "contractAddress": "",
  "label": "",
  "merchantId": "",
  "nonce": 0,
  "nonceStr": "",
  "requestNo": "",
  "settlementType": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明                                          | 请求类型 | 是否必须 | 数据类型                 | schema                   |
| ----------------------------- | ------------------------------------------------- | -------- | -------- | ------------------------ | ------------------------ |
| input                         | input                                             | body     | true     | SettlementApplyInputForm | SettlementApplyInputForm |
| &emsp;&emsp;clientPlatformBid | 产融平台bid                                       |          | true     | string                   |                          |
| &emsp;&emsp;contractAddress   | 合约地址                                          |          | true     | string                   |                          |
| &emsp;&emsp;label             | 唯一标识                                          |          | true     | string                   |                          |
| &emsp;&emsp;merchantId        | 商户号                                            |          | true     | string                   |                          |
| &emsp;&emsp;nonce             | 产融平台Bid指定区块链交易的nonce值                |          | true     | integer(int64)           |                          |
| &emsp;&emsp;nonceStr          | 随机字符串                                        |          | true     | string                   |                          |
| &emsp;&emsp;requestNo         | 请求号                                            |          | true     | string                   |                          |
| &emsp;&emsp;settlementType    | 结算类型：线上结算/线下结算,可用值:OFFLINE,ONLINE |          | true     | string                   |                          |
| &emsp;&emsp;sign              | 签名串                                            |          | true     | string                   |                          |
| &emsp;&emsp;signType          | 签名方式                                          |          | true     | string                   |                          |


**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | Result«SettlementApplyVO» |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称         | 参数说明 | 类型              | schema            |
| ---------------- | -------- | ----------------- | ----------------- |
| code             |          | string            |                   |
| data             |          | SettlementApplyVO | SettlementApplyVO |
| &emsp;&emsp;blob | 待签数据 | string            |                   |
| &emsp;&emsp;hash | 交易hash | string            |                   |
| message          |          | string            |                   |
| total            |          | integer(int32)    | integer(int32)    |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"blob": "",
		"hash": ""
	},
	"message": "",
	"total": 0
}
```


##### 开始清结算


**接口地址**:`/if-settlement/api/settlement/v1/settlement/start-settle`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>开始清结算</p>

**请求示例**:


```javascript
{
  "bankAccountType": "",
  "capitalPathType": "",
  "channelCode": "",
  "clientPlatformBid": "",
  "contractAddress": "",
  "label": "",
  "merchantId": "",
  "nonceStr": "",
  "payMode": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明                                                     | 请求类型 | 是否必须 | 数据类型                 | schema                   |
| ----------------------------- | ------------------------------------------------------------ | -------- | -------- | ------------------------ | ------------------------ |
| input                         | input                                                        | body     | true     | SettlementStartInputForm | SettlementStartInputForm |
| &emsp;&emsp;bankAccountType   | 子账户模式,可用值:CARD,PLATFORM_ACCOUNT,SUB_ACC              |          | true     | string                   |                          |
| &emsp;&emsp;capitalPathType   | 资金清分方式,可用值:ENDTOEND,FINANCEENDTOEND,FINANCEGRADUALLY,GRADUALLY,GRADUALLY_CORE_REPAY_DEFAULT_DONE |          | true     | string                   |                          |
| &emsp;&emsp;channelCode       | 结算渠道配置                                                 |          | true     | string                   |                          |
| &emsp;&emsp;clientPlatformBid | 产融平台bid                                                  |          | true     | string                   |                          |
| &emsp;&emsp;contractAddress   | 合约地址                                                     |          | true     | string                   |                          |
| &emsp;&emsp;label             | 唯一标识                                                     |          | true     | string                   |                          |
| &emsp;&emsp;merchantId        | 商户号                                                       |          | true     | string                   |                          |
| &emsp;&emsp;nonceStr          | 随机字符串                                                   |          | true     | string                   |                          |
| &emsp;&emsp;payMode           | 结算方式,可用值:OFFLINE,ONLINE                               |          | true     | string                   |                          |
| &emsp;&emsp;requestNo         | 请求号                                                       |          | true     | string                   |                          |
| &emsp;&emsp;sign              | 签名串                                                       |          | true     | string                   |                          |
| &emsp;&emsp;signType          | 签名方式                                                     |          | true     | string                   |                          |


**响应状态**:


| 状态码 | 说明         | schema                    |
| ------ | ------------ | ------------------------- |
| 200    | OK           | Result«SettlementStartVO» |
| 201    | Created      |                           |
| 401    | Unauthorized |                           |
| 403    | Forbidden    |                           |
| 404    | Not Found    |                           |


**响应参数**:


| 参数名称                  | 参数说明   | 类型              | schema            |
| ------------------------- | ---------- | ----------------- | ----------------- |
| code                      |            | string            |                   |
| data                      |            | SettlementStartVO | SettlementStartVO |
| &emsp;&emsp;sourceAddress | 交易发起人 | string            |                   |
| &emsp;&emsp;txHash        | 交易hash   | string            |                   |
| message                   |            | string            |                   |
| total                     |            | integer(int32)    | integer(int32)    |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"sourceAddress": "",
		"txHash": ""
	},
	"message": "",
	"total": 0
}
```


##### 清结算业务区块链交易提交


**接口地址**:`/if-settlement/api/settlement/v1/settlement/submit`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>清算提交-合约</p>

**请求示例**:


```javascript
{
  "blob": "",
  "clientPlatformBid": "",
  "hash": "",
  "merchantId": "",
  "nonceStr": "",
  "publicKey": "",
  "requestNo": "",
  "sign": "",
  "signStr": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明    | 请求类型 | 是否必须 | 数据类型                  | schema                    |
| ----------------------------- | ----------- | -------- | -------- | ------------------------- | ------------------------- |
| input                         | input       | body     | true     | SettlementSubmitInputForm | SettlementSubmitInputForm |
| &emsp;&emsp;blob              | 待签数据    |          | false    | string                    |                           |
| &emsp;&emsp;clientPlatformBid | 产融平台bid |          | true     | string                    |                           |
| &emsp;&emsp;hash              | hash        |          | true     | string                    |                           |
| &emsp;&emsp;merchantId        | 商户号      |          | true     | string                    |                           |
| &emsp;&emsp;nonceStr          | 随机字符串  |          | true     | string                    |                           |
| &emsp;&emsp;publicKey         | 公钥        |          | false    | string                    |                           |
| &emsp;&emsp;requestNo         | 请求号      |          | true     | string                    |                           |
| &emsp;&emsp;sign              | 签名串      |          | true     | string                    |                           |
| &emsp;&emsp;signStr           | 签名串      |          | true     | string                    |                           |
| &emsp;&emsp;signType          | 签名方式    |          | true     | string                    |                           |


**响应状态**:


| 状态码 | 说明         | schema                     |
| ------ | ------------ | -------------------------- |
| 200    | OK           | Result«SettlementSubmitVO» |
| 201    | Created      |                            |
| 401    | Unauthorized |                            |
| 403    | Forbidden    |                            |
| 404    | Not Found    |                            |


**响应参数**:


| 参数名称                    | 参数说明 | 类型               | schema             |
| --------------------------- | -------- | ------------------ | ------------------ |
| code                        |          | string             |                    |
| data                        |          | SettlementSubmitVO | SettlementSubmitVO |
| &emsp;&emsp;contractAddress | 合约地址 | string             |                    |
| &emsp;&emsp;label           | 唯一标识 | string             |                    |
| message                     |          | string             |                    |
| total                       |          | integer(int32)     | integer(int32)     |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"contractAddress": "",
		"label": ""
	},
	"message": "",
	"total": 0
}
```

##### 调账


**接口地址**:`/if-settlement/api/settlement/v1/settlement/order`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>调账</p>

**请求示例**:


```javascript
{
  "bankAccountType": "",
  "capitalPathType": "",
  "clientPlatformBid": "",
  "contractAddress": "",
  "label": "",
  "merchantId": "",
  "nonceStr": "",
  "payMode": "",
  "requestId": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明                                                     | 请求类型 | 是否必须 | 数据类型               | schema                 |
| ----------------------------- | ------------------------------------------------------------ | -------- | -------- | ---------------------- | ---------------------- |
| input                         | input                                                        | body     | true     | CheckContractInputForm | CheckContractInputForm |
| &emsp;&emsp;bankAccountType   | 子账户模式,可用值:CARD,PLATFORM_ACCOUNT,SUB_ACC              |          | true     | string                 |                        |
| &emsp;&emsp;capitalPathType   | 资金清分方式,可用值:ENDTOEND,FINANCEENDTOEND,FINANCEGRADUALLY,GRADUALLY,GRADUALLY_CORE_REPAY_DEFAULT_DONE |          | true     | string                 |                        |
| &emsp;&emsp;clientPlatformBid | 产融平台bid                                                  |          | true     | string                 |                        |
| &emsp;&emsp;contractAddress   | 合约地址                                                     |          | true     | string                 |                        |
| &emsp;&emsp;label             | 合约唯一标识                                                 |          | true     | string                 |                        |
| &emsp;&emsp;merchantId        | 商户号                                                       |          | true     | string                 |                        |
| &emsp;&emsp;nonceStr          | 随机字符串                                                   |          | true     | string                 |                        |
| &emsp;&emsp;payMode           | 结算方式,可用值:OFFLINE,ONLINE                               |          | true     | string                 |                        |
| &emsp;&emsp;requestId         | 请求号                                                       |          | true     | string                 |                        |
| &emsp;&emsp;requestNo         | 请求号                                                       |          | true     | string                 |                        |
| &emsp;&emsp;sign              | 签名串                                                       |          | true     | string                 |                        |
| &emsp;&emsp;signType          | 签名方式                                                     |          | true     | string                 |                        |


**响应状态**:


| 状态码 | 说明         | schema             |
| ------ | ------------ | ------------------ |
| 200    | OK           | Result«ContractVO» |
| 201    | Created      |                    |
| 401    | Unauthorized |                    |
| 403    | Forbidden    |                    |
| 404    | Not Found    |                    |


**响应参数**:


| 参数名称              | 参数说明 | 类型           | schema         |
| --------------------- | -------- | -------------- | -------------- |
| code                  |          | string         |                |
| data                  |          | ContractVO     | ContractVO     |
| &emsp;&emsp;requestId |          | string         |                |
| message               |          | string         |                |
| total                 |          | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"requestId": ""
	},
	"message": "",
	"total": 0
}
```

#### 清结算服务：查询操作

##### 清结算资金路径预览


**接口地址**:`/if-settlement/api/settlement/v1/settlement/list/payments`

**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>根据合约号（结算渠道，资金路径）查询付款还款列表信息 （支持批量）</p>

**接口功能描述**:<p>数据状态是“业务上链”、“申请清结算”均可调用清结算预览，预览资金清算数据情况。</p>

**请求示例**:


```javascript
{
  "clearedData": [
    {
      "capitalPathType": "",
      "channelCode": "",
      "contractAddress": "",
      "label": "",
      "requestId": ""
    }
  ],
  "clientPlatformBid": "",
  "merchantId": "",
  "nonceStr": "",
  "requestNo": "",
  "sign": "",
  "signType": ""
}
```


**请求参数**:


| 参数名称                                | 参数说明                                                     | 请求类型 | 是否必须 | 数据类型           | schema              |
| --------------------------------------- | ------------------------------------------------------------ | -------- | -------- | ------------------ | ------------------- |
| input                                   | input                                                        | body     | true     | QueryContractQuery | QueryContractQuery  |
| &emsp;&emsp;clearedData                 | 请求参数                                                     |          | true     | array              | ContractClearedData |
| &emsp;&emsp;&emsp;&emsp;capitalPathType | 资金清分方式,可用值:ENDTOEND,FINANCEENDTOEND,FINANCEGRADUALLY,GRADUALLY,GRADUALLY_CORE_REPAY_DEFAULT_DONE |          | true     | string             |                     |
| &emsp;&emsp;&emsp;&emsp;channelCode     | 结算渠道配置                                                 |          | true     | string             |                     |
| &emsp;&emsp;&emsp;&emsp;contractAddress | 合约地址                                                     |          | true     | string             |                     |
| &emsp;&emsp;&emsp;&emsp;label           | 合约唯一标识                                                 |          | true     | string             |                     |
| &emsp;&emsp;&emsp;&emsp;requestId       | 请求号                                                       |          | true     | string             |                     |
| &emsp;&emsp;clientPlatformBid           | 产融平台bid                                                  |          | true     | string             |                     |
| &emsp;&emsp;merchantId                  | 商户号                                                       |          | true     | string             |                     |
| &emsp;&emsp;nonceStr                    | 随机字符串                                                   |          | true     | string             |                     |
| &emsp;&emsp;requestNo                   | 请求号                                                       |          | true     | string             |                     |
| &emsp;&emsp;sign                        | 签名串                                                       |          | true     | string             |                     |
| &emsp;&emsp;signType                    | 签名方式                                                     |          | true     | string             |                     |


**响应状态**:


| 状态码 | 说明         | schema |
| ------ | ------------ | ------ |
| 200    | OK           | Result |
| 201    | Created      |        |
| 401    | Unauthorized |        |
| 403    | Forbidden    |        |
| 404    | Not Found    |        |


**响应参数**:


| 参数名称 | 参数说明 | 类型           | schema         |
| -------- | -------- | -------------- | -------------- |
| code     |          | string         |                |
| data     |          | object         |                |
| message  |          | string         |                |
| total    |          | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": {},
	"message": "",
	"total": 0
}
```

##### 清算结果查询


**接口地址**:`/if-settlement/api/settlement/v1/settlement-query/execute-result`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>清算结果查询</p>

**接口功能描述**:数据状态是“执行清结算”、“待调账”、“清结算完成”均可调用清结算结果查询，查询清结算实时进度。

**请求示例**:


```javascript
{
  "clientPlatformBid": "",
  "contractAddress": "",
  "label": "",
  "requestNo": ""
}
```


**请求参数**:


| 参数名称                      | 参数说明    | 请求类型 | 是否必须 | 数据类型              | schema                |
| ----------------------------- | ----------- | -------- | -------- | --------------------- | --------------------- |
| query                         | query       | body     | true     | SettlementResultQuery | SettlementResultQuery |
| &emsp;&emsp;clientPlatformBid | 产融平台Bid |          | true     | string                |                       |
| &emsp;&emsp;contractAddress   | 合约地址    |          | true     | string                |                       |
| &emsp;&emsp;label             | 唯一标识    |          | true     | string                |                       |
| &emsp;&emsp;requestNo         | 请求号      |          | true     | string                |                       |


**响应状态**:


| 状态码 | 说明         | schema                            |
| ------ | ------------ | --------------------------------- |
| 200    | OK           | Result«SettlementExecuteResultVO» |
| 201    | Created      |                                   |
| 401    | Unauthorized |                                   |
| 403    | Forbidden    |                                   |
| 404    | Not Found    |                                   |


**响应参数**:


| 参数名称                          | 参数说明                             | 类型                      | schema                    |
| --------------------------------- | ------------------------------------ | ------------------------- | ------------------------- |
| code                              |                                      | string                    |                           |
| data                              |                                      | SettlementExecuteResultVO | SettlementExecuteResultVO |
| &emsp;&emsp;detailList            | 清算明细                             | array                     | SettlementExecuteDetailVO |
| &emsp;&emsp;&emsp;&emsp;amount    | 结算金额                             | integer                   |                           |
| &emsp;&emsp;&emsp;&emsp;from      | 付款账户地址（资金清算路径付款账户） | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;payTime   | 结算时间                             | integer                   |                           |
| &emsp;&emsp;&emsp;&emsp;requestNo | 业务编号                             | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;status    | 结算状态                             | string                    |                           |
| &emsp;&emsp;&emsp;&emsp;to        | 收款账户地址（资金清算路径收款账户） | string                    |                           |
| &emsp;&emsp;requestNo             | 请求号                               | string                    |                           |
| message                           |                                      | string                    |                           |
| total                             |                                      | integer(int32)            | integer(int32)            |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"detailList": [
			{
				"amount": 0,
				"from": "",
				"payTime": 0,
				"requestNo": "",
				"status": "",
				"to": ""
			}
		],
		"requestNo": ""
	},
	"message": "",
	"total": 0
}
```


##### 出金结果查询


**接口地址**:`/if-settlement/api/settlement/v1/settlement-query/withdraw-result`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:<p>出金结果查询</p>

**请求示例**:


```javascript
{
  "financeNo": [],
  "requestNo":""
}
```


**请求参数**:


| 参数名称              | 参数说明 | 请求类型 | 是否必须 | 数据类型                      | schema                        |
| --------------------- | -------- | -------- | -------- | ----------------------------- | ----------------------------- |
| query                 | query    | body     | true     | SettlementWithdrawResultQuery | SettlementWithdrawResultQuery |
| &emsp;&emsp;financeNo | 融资编号 |          | true     | array                         | string                        |
| requestNo             | 请求号   |          | true     |                               |                               |


**响应状态**:


| 状态码 | 说明         | schema                             |
| ------ | ------------ | ---------------------------------- |
| 200    | OK           | Result«SettlementWithdrawResultVO» |
| 201    | Created      |                                    |
| 401    | Unauthorized |                                    |
| 403    | Forbidden    |                                    |
| 404    | Not Found    |                                    |


**响应参数**:


| 参数名称                                | 参数说明                                           | 类型                       | schema                     |
| --------------------------------------- | -------------------------------------------------- | -------------------------- | -------------------------- |
| code                                    |                                                    | string                     |                            |
| data                                    |                                                    | SettlementWithdrawResultVO | SettlementWithdrawResultVO |
| &emsp;&emsp;orderDomains                | 订单列表                                           | array                      | SettlementOrderDomain对象  |
| &emsp;&emsp;&emsp;&emsp;action          | action 动作                                        | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;amount          | 交易金额:单位元                                    | number                     |                            |
| &emsp;&emsp;&emsp;&emsp;bankAccountType | 银行账户类型(0:子账户1:结算户)                     | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;bidPayee        | 收款方bid                                          | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;bidPayer        | 付款方bid                                          | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;capitalType     | 资金清分方式:gradually,逐级清算,end-to-end首尾兑付 | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;contractAddress | 合约地址                                           | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;created         |                                                    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;deleted         | 是否删除                                           | boolean                    |                            |
| &emsp;&emsp;&emsp;&emsp;id              |                                                    | integer                    |                            |
| &emsp;&emsp;&emsp;&emsp;label           | 业务唯一标识                                       | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;no              | 业务编号                                           | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;nodeNo          | 节点编号                                           | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;orderId         | 清算订单ID                                         | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;parentNodeNo    | 父节点编号                                         | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;payMode         | 支付方式(1:线上2:线下)                             | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;remark          | 备注                                               | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;requestId       | 请求号                                             | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;settlementType  | 交易类型:TRANSFER   NONE   WITHDRAW                | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;state           | 状态1待付款、2进行中、3已付款、4付款失败           | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;updated         |                                                    | string                     |                            |
| &emsp;&emsp;&emsp;&emsp;version         | 版本号                                             | string                     |                            |
| message                                 |                                                    | string                     |                            |
| total                                   |                                                    | integer(int32)             | integer(int32)             |


**响应示例**:

```javascript
{
	"code": "",
	"data": {
		"orderDomains": [
			{
				"action": "",
				"amount": 0,
				"bankAccountType": "",
				"bidPayee": "",
				"bidPayer": "",
				"capitalType": "",
				"contractAddress": "",
				"created": "",
				"deleted": true,
				"id": 0,
				"label": "",
				"no": "",
				"nodeNo": "",
				"orderId": "",
				"parentNodeNo": "",
				"payMode": "",
				"remark": "",
				"requestId": "",
				"settlementType": "",
				"state": "",
				"updated": "",
				"version": ""
			}
		]
	},
	"message": "",
	"total": 0
}
```

##### 查询指定账户当前实时nonce值


**接口地址**:`/if-settlement/api/settlement/v1/blockchain/get-nonce`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`

**接口描述**:


**请求示例**:


```javascript
{
  "accountAddress": "",
  "requestNo": ""
}
```


**请求参数**:


| 参数名称                   | 参数说明           | 请求类型 | 是否必须 | 数据类型          | schema            |
| -------------------------- | ------------------ | -------- | -------- | ----------------- | ----------------- |
| input                      | input              | body     | true     | AccountNonceQuery | AccountNonceQuery |
| &emsp;&emsp;accountAddress | 待查询的区块链地址 |          | true     | string            |                   |
| &emsp;&emsp;requestNo      | 请求号             |          | true     | string            |                   |


**响应状态**:


| 状态码 | 说明         | schema       |
| ------ | ------------ | ------------ |
| 200    | OK           | Result«long» |
| 201    | Created      |              |
| 401    | Unauthorized |              |
| 403    | Forbidden    |              |
| 404    | Not Found    |              |


**响应参数**:


| 参数名称 | 参数说明 | 类型           | schema         |
| -------- | -------- | -------------- | -------------- |
| code     |          | string         |                |
| data     |          | integer(int64) | integer(int64) |
| message  |          | string         |                |
| total    |          | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": 0,
	"message": "",
	"total": 0
}
```

### 2.10.文件存储服务

**简介**:星火产融中台通用文件上传下载接口。


#### 文件传输接口


##### 下载文件


**接口地址**:`/if-oss/api/oss/v1/downFile`


**请求方式**:`GET`


**请求数据类型**:`application/x-www-form-urlencoded`


**响应数据类型**:`*/*`


**接口描述**:


**请求参数**:


| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 | schema |
| -------- | -------- | -------- | -------- | -------- | ------ |
| fileUrl  | fileUrl  | query    | true     | string   |        |


**响应状态**:


| 状态码 | 说明         | schema |
| ------ | ------------ | ------ |
| 200    | OK           |        |
| 401    | Unauthorized |        |
| 403    | Forbidden    |        |
| 404    | Not Found    |        |


**响应参数**:


暂无


**响应示例**:

```javascript

```


##### 文件上传


**接口地址**:`/if-oss/api/oss/v1/upload`


**请求方式**:`POST`


**请求数据类型**:`application/json`


**响应数据类型**:`*/*`


**接口描述**:

**请求参数**:


暂无


**响应状态**:


| 状态码 | 说明         | schema                     |
| ------ | ------------ | -------------------------- |
| 200    | OK           | Result«List«文件上传结果»» |
| 201    | Created      |                            |
| 401    | Unauthorized |                            |
| 403    | Forbidden    |                            |
| 404    | Not Found    |                            |


**响应参数**:


| 参数名称                  | 参数说明             | 类型           | schema         |
| ------------------------- | -------------------- | -------------- | -------------- |
| code                      |                      | string         |                |
| data                      |                      | array          | 文件上传结果   |
| &emsp;&emsp;fileUrl       | 产融文件上传唯一标识 | string         |                |
| &emsp;&emsp;requestFileId | 调用方请求文件标识   | string         |                |
| message                   |                      | string         |                |
| total                     |                      | integer(int32) | integer(int32) |


**响应示例**:

```javascript
{
	"code": "",
	"data": [
		{
			"fileUrl": "",
			"requestFileId": ""
		}
	],
	"message": "",
	"total": 0
}
```

