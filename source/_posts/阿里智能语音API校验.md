---
title: 阿里智能语音API校验
date: 2018-08-10 11:11:58
tags: ['ruby', 'third-party']
---

### 应用场景
调用阿里录音文件识别API时，需要在header中添加校验字符串，参考官网的node示例（[传送门](https://help.aliyun.com/document_detail/30245.html?spm=a2c4g.11186623.2.4.2AZ7xc)），以下给出ruby生成校验字符串的方法．

### 代码
```ruby
require("net/http")
require('base64')
require('digest')
require('json')
require('openssl')

app_id = 'id'
app_secret = "secret"
params = {"app_key":"nls-service-telephone8khz","enable_callback":false,"oss_link":"http://aliyun-nls.oss.aliyuncs.com/asr/fileASR/examples/nls-sample.wav"}

now = "Tue, 19 Dec 2017 07:43:40 GMT" 
# 使用GMT格式时间
# now = Time.now.gmtime.strftime("%a, %d %b %Y %T GMT")

headers = {
  method: 'POST',
  accept: 'application/json',
  'content-type': 'application/json',
  date: "#{now}"
}

## step 1. 组stringToSign,注意这里不加urlpath
string2sign = "POST" + "\n" + headers[:accept] + "\n" +
  Digest::MD5.base64digest(params.to_json) + "\n" + 
  headers[:"content-type"] + "\n" +
  headers[:date]
puts string2sign ## => POST
                 ##    application/json
                 ##    0yo5NmJ4dReSHOxlItYpvA==
                 ##    application/json
                 ##    Tue, 19 Dec 2017 07:43:40 GMT


## step 2. 加密[Signature = Base64( HMAC-SHA1( AccessSecret, UTF-8-Encoding-Of(StringToSign) ) )]
digest = OpenSSL::Digest.new('sha1')
signature = Base64.encode64(OpenSSL::HMAC.digest(digest, app_secret, string2sign))

## step 3. 组authorization header [Authorization =  Dataplus AccessKeyId + ":" + Signature]
auth_header = "Dataplus " + app_id + ":" + signature
puts auth_header ## => Dataplus id:oqoyn3Y1/cPpK2DfU5jy2DbaKws=

uri = URI("https://nlsapi.aliyun.com/transcriptions")
response = Net::HTTP.start(uri.host, uri.port, use_ssl: uri.scheme == 'https') do |http|
  request = Net::HTTP::Post.new(uri, {'Accept' => 'application/json',
                                      'Content-type' => 'application/json',
                                      'Date' => "#{now}",
                                      'Authorization' => "#{auth_header}"})
  request.body = params.to_json
  http.request request
end

puts response.body

```
