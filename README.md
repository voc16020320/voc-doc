# 第三方APP授权文档

* #####联系官方申请第三方APP，申请人需提供VID、APP英文名、获得授权url（例如 http://www.xxx.com ，以下简称cb_url），申请成功后获得API加密秘钥（以下简称apiSecret）。

* #####用户在VOC客户端（以下简称客户端）授权过后请求cb_url，GET方法，参数为token，例如: http://www.xxx.com?id=foolbar ，此id为该用户对应的身份特征码。

* ##### APP需在收到id特征码后一小时内完成授权绑定，请求接口为: http://app.vocx.io/auth/access ，POST方法，参数如下：

参数名  | 类型 | 示例或说明
------------- | ------------- | -------------
 appName | string  | 即申请APP时提供的英文名
id  | string  | 上一步获取的id特征码

返回示例：

字段  | 类型 | 示例或说明
------------- | ------------- | -------------
 token | string  | 用户token

 * ##### 返回数据标准格式：
正确返回示例：
```javascript
{
     "success": true, // 是否正确
     "errcode": "",
     "message": "",
     "data": { // 返回数据
	     "fool":"9e860558e7fc4844abadc752d0e180f3"
	  }
}
```
错误返回示例：
```javascript
{
     "success": false, // 是否正确
     "errcode": "VID_NOT_FOUND", // 错误码
     "message": "无法找到VID", // 错误信息
     "data": {} // 附加数据
}
```

* ##### 至此APP即可进行业务接口api调用，所有业务API接口均需进行接口加密传输，域名为http://app.vocx.io ,加密方法如下：
例如api为 `http://app.vocx.io/api/fool`，POST方法，参数数据为 `{bbb:11,aaa:22}`，步骤如下：

1. 将参数与apiSecret合并为同一对象，合并后为 `{bbb:11,aaa:22,apiSecret:"xxx"}`；

2. 将以上合并后的对象按照字段排序，排序后为 `{aaa:22,apiSecret:"xxx",bbb:11}`；

3. 将以上对象进行stringfry和unescape操作变成字符串，最后将字符串进行sha1加密，以nodejs语法示例：
```javascript
import * as querystring from "querystring";
import * as crypto from 'crypto';
const params = {aaa:22,apiSecret:"xxx",bbb:11}; // 排序过后的对象
let str = querystring.stringify(params); // 进行stringfry操作
str = querystring.unescape(str); // 进行unescape操作
const sha1 = crypto.createHash('sha1');
const code = sha1.update(str, 'utf8').digest('hex'); // sha1加密，得到的code即最终结果
```

4. 最终的请求变成：`http://app.vocx.io/api/fool?name=appName&code=code`，POST方法，

	header添加用户token，例如：

```javascript
{
	"Content-Type":"application/json",
	"token":"token"
}
```
POST数据不变，仍为 `{bbb:11,aaa:22}`。

### API

* ##### 划转用户VOCD
 action： `/api/vocd_trans`

 method： `POST`

参数名  | 类型 | 示例或说明
------------- | ------------- | -------------
 amount | string  | 带符号字符串，例如 "23" 或者 "-6"
 
返回bool值表示是否调用成功
 
 * ##### 查询用户VOCD可用余额
 action: `/api/vocd_balance`
 
 method： `POST`
 
 无参数
 
 返回示例：
 
参数名  | 类型 | 示例或说明
------------- | ------------- | -------------
 balance | string  | 带符号字符串，例如 "23" 或者 "-6"
 
### END
