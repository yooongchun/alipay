## python-alipay-sdk
[![codecov](https://codecov.io/gh/fzlee/alipay/branch/master/graph/badge.svg)](https://codecov.io/gh/fzlee/alipay)
## Changelog

### 2017-01-17(version 0.5.1)
* 修复网页扫码支付的退款bug(感谢varwey)

### 2017-01-13(version 0.5)
* 默认使用SHA256withRSA签名算法, 如果你想用SHA1withRSA算法，创建alipay的时候传入sign_type = "RSA"

### 2017-01-06(version 0.4)
* 添加了退款功能
* 添加了测试
* 修复Python 2下中文编码问题

##  支付宝Python SDK
支付宝没有提供Python SDK。生成预付订单需要使用SHA1withRSA签名，签名的生成比较麻烦容易出错。这里提供了一个简单的库，希望能够简化一些Python开发的流程。

目前我们支持如下三种支付方式，且签名类型必须为RSA
* [App支付](https://doc.open.alipay.com/docs/doc.htm?treeId=193&articleId=105051&docType=1)
* [手机网站支付](https://doc.open.alipay.com/docs/doc.htm?treeId=193&articleId=105288&docType=1)
* [即时到帐](https://doc.open.alipay.com/doc2/detail?treeId=62&articleId=103566&docType=1)

其他支持的功能：
* [退款](https://doc.open.alipay.com/docs/api.htm?docType=4&apiId=759)

关于支付宝移动支付的详细介绍参看[这篇教程](https://ifconfiger.com/page/python-alipay-sdk). 如果你不希望深入了解技术实现的细节，你可以直接参看下面的使用教程。

## 使用教程
#### 安装

```bash
pip install python-alipay-sdk
```

####初始化
```Python
    from alipay import AliPay
    
    # 手机网站或者app支付
    alipay = AliPay(
      appid="",
      app_notify_url="", 
      app_private_key_path="", 
      app_alipay_public_key_path=""  # 支付宝的公钥，验证支付宝回传消息使用，不是你自己的公钥,
      sign_type="RSA" # RSA 或者 RSA2
    )
	
	# 即时到帐
	alipay = AliPay(
      partner="",
      web_notify_url="", 
      web_private_key_path="", 
      web_alipay_public_key_path=""  # 支付宝的公钥，验证支付宝回传消息使用，不是你自己的公钥
    )
	
	# 如果你希望 alipay能够同时处理三种支付方式, 那就传入所有的参数
	alipay = AliPay(
	  appid="",
      app_notify_url="", 
      app_private_key_path="",
      app_alipay_public_key_path="",
      partner="", 
      web_notify_url="",
      web_private_key_path="", 
      web_alipay_public_key_path="" 
    )
```
	
#### 生成订单
```Python
	# App支付，将order_string返回给app即可
	order_string = alipay.create_app_trade(out_trade_no="20161112", total_amount="0.01", subject="测试订单")
	# 手机网站支付，需要跳转到https://openapi.alipay.com/gateway.do? + order_string
	order_string = alipay.create_wap_trade(out_trade_no="20161112", total_amount="0.01", subject="测试订单", return_url="https://example.com")
	# 即时到帐，需要跳转到https://mapi.alipay.com/gateway.do? + order_string
	order_string = alipay.create_web_trade(out_trade_no="20161112", total_amount="0.01", subject="测试订单", return_url="https://example.com")
```
#### 通知验证
```Python
	# 验证alipay的异步通知，data来自支付宝回调POST 给你的data，字典格式.
	data = {
          "subject": "测试订单",
          "gmt_payment": "2016-11-16 11:42:19",
          "charset": "utf-8",
          "seller_id": "xxxx",
          "trade_status": "TRADE_SUCCESS",
          "buyer_id": "xxxx",
          "auth_app_id": "xxxx",
          "buyer_pay_amount": "0.01",
          "version": "1.0",
          "gmt_create": "2016-11-16 11:42:18",
          "trade_no": "xxxx",
          "fund_bill_list": "[{\"amount\":\"0.01\",\"fundChannel\":\"ALIPAYACCOUNT\"}]",
          "app_id": "xxxx",
          "notify_time": "2016-11-16 11:42:19",
          "point_amount": "0.00",
          "total_amount": "0.01",
          "notify_type": "trade_status_sync",
          "out_trade_no": "xxxx",
          "buyer_logon_id": "xxxx",
          "notify_id": "xxxx",
          "seller_email": "xxxx",
          "receipt_amount": "0.01",
          "invoice_amount": "0.01",
          "sign": "xxx"
        }
	data.pop("sign_type", "") #  sign_type 不参与签名
    signature = data.pop("sign")
	# 验证app支付
	success = alipay.verify_app_notify(data, signature)
    # 验证手机网站支付
	success = alipay.verify_wap_notify(data, signature)
	# 验证即时到帐
	success = alipay.verify_web_notify(data, signature)
	if success and (data["trade_status"] == "TRADE_SUCCESS" or data["trade_status"] == "TRADE_FINISHED" ):
		print("trade succeed")
```

#### 退款

refund 需要传入的参数参见[官方文档](https://doc.open.alipay.com/docs/api.htm?docType=4&apiId=759)的**请求参数**

```Python
# 即时到账退款
try:
    alipay.refund_web_order(out_trade_no="xxx", refund_amount="xxx", ...)
except AliPayException as e:
    raise e
return True
```

## 测试
```
python -m unittest discover
```

## 感谢（排名不分先后）
* varwey
