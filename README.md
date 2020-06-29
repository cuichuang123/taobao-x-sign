python 版本demo
运行条件: python3 + requests 库

import hashlib
import json
import time
from urllib import parse

import requests

### 请求签名的校验参数
token='1234'

### 请求淘宝的api接口
api='mtop.taobao.rate.detaillist.get'

### 请求淘宝的api接口版本
v='4.0'

 ### 获取签名信息 参数data 要注意转义符号
data = "{\"pageSize\":\"10\",\"foldFlag\":\"0\",\"hasPic\":\"1\",\"pageNo\":\"1\",\"auctionNumId\":\"602659642364\"}"

###  淘宝请求地址，所有post请求基本都可以转换成get请求
taobao_url = "http://acs.m.taobao.com/gw/"+api+"/"+v+"?data=" + data

### sid 需要登陆淘宝后获取，一般长度为32位
### sid = '1d80'

### pv 对应签名版本
pv = '6.3'

### appKey 固定值，固定为'21646297'
appKey = '21646297'

### ttid 手机淘宝版本号
ttid = '701186@taobao_android_9.1.0'

### x_features 对应api功能
x_features = '27'

### deviceId 设备id，可44位随机
deviceId = 'AiN8kvbdEkyD1P3CqFhYct1a7PmMab5dj804e192TcrV'

### utdid 淘宝uuid，可以24位随机
utdid = 'XcZJFF61gMADAep76BgfX2AD'


def get_sign(data):

    url = 'http://api.xsign.com/api/sign'  # 获取签名的地址
    
    params = {
        #'sid': sid,
        'data': hashlib.md5(data.encode(encoding='UTF-8')).hexdigest(),  # 获取签名需要将data进行md5处理，以方便数据传输
        'api': api,
        'v': v,
        't': str(int(time.time())),  # 时间戳，单位：秒
        'ttid': ttid,
        'utdid': utdid,
        'deviceId': deviceId,
        'x-features': x_features,
        'appKey': appKey,
        'pv': pv,
        'dataMd5': 'true',  # 如果data进行了md5处理，那这里需要设置为'true',
        'token': token,  # 获取签名参数所需的token值，长度为32位，有需要的请联系qq951263019申请
    }
    header = {'Content-Type': 'application/json'}  # 协议头
    return requests.post(url, data=json.dumps(params), headers=header).json()


def fetch_rate(sign_res, data):

    header = {
        #'x-sid': sid,
        'user-Agent': 'MTOPSDK%2F3.1.1.7+%28Android%3B4.4.2%3BXiaomi%3BMI+6%29',
        'x-appkey': appKey,
        'x-ttid': parse.quote(ttid), # 进行URL encode处理，比如@符号要转换成%40
        'x-devid': parse.quote(deviceId),
        'x-features': x_features,
        'x-utdid': parse.quote(utdid),
        'x-pv': pv,
        'x-location': '%2C',  # 如果获取参数的时候有参数lat和lng，那这里就是lng%2Clat，本例为空则设置为%2C
        'x-t': sign_res['x-t'],
        'x-sign': sign_res['x-sign'],
        "x-mini-wua":sign_res["x-mini-wua"],
        "x-sgext":sign_res['x-sgext'],
    }
   
    return requests.get(taobao_url, headers=header).json()




if __name__ == '__main__':

    sign_res = get_sign(data)  # 获取x-sign有需要的请联系qq951263019
    
    print(sign_res)
    
    rate = fetch_rate(sign_res, data)
    
    print(rate)
    

# 技术支持： 
qq:516766219
