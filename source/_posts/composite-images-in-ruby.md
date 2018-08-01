---
title: composite images in ruby
date: 2018-08-01 15:53:07
tags:
---

## 应用场景
  在一张宣传图片上加上二维码和自己的微信头像, 用到mini_magick这个gem即可.

## 代码
```ruby
require "mini_magick"
require 'net/http'
require 'tempfile'

def composite_image(qrcode_ticket="gQEn8DwAAAAAAAAAAS5odHRwOi8vd2VpeGluLnFxLmNvbS9xLzAyd2RybG95cTNma2wxMDAwME0wMy0AAgRK9RhbAwQAAAAA")
  qrcode_uri = URI("https://mp.weixin.qq.com/cgi-bin/showqrcode?ticket=#{qrcode_ticket}")
  res = Net::HTTP.get_response(qrcode_uri)
  raise "get qrcode faild error " unless res.is_a?(Net::HTTPSuccess)

  tmpfile = Tempfile.new(qrcode_ticket)
  open(tmpfile.path, "wb") { |file| file.write(res.body) }

  profile_uri = URI("http://thirdwx.qlogo.cn/mmopen/8StU3PHCdLkPhx9kGZM1AYf9Ou2kncJb1RCCYc3DGoBoapgtdqrSDAKWIq7oNUcekicmxfDoLok5Spicf9uG4G5ZwpkkuKoXRw/132")
  profile_res = Net::HTTP.get_response(profile_uri)
  raise "get qrcode faild error " unless profile_res.is_a?(Net::HTTPSuccess)

  profile_image = Tempfile.new(["profile", '.jpg'])
  open(profile_image.path, "wb") { |file| file.write(profile_res.body) }

  MiniMagick::Tool::Convert.new do |c|
    c << "monkey100.jpg"
    c.merge! [tmpfile.path, "-geometry", "205x205+433+955", "-composite"]  # -geometry 选项指定大小和位置，这里只是设置了位置。
    c.merge! [profile_image.path, "-geometry", "95x95+10+10", "-composite"]
    c.merge! ["-pointsize", "26", "-font", "./simfang.ttf", "-fill", "black", "-draw", "text 120,60 '安'"]
    c << "out.jpg"
  end

end

composite_image()

```
## git demo
[Goooooo](https://github.com/icecoll/blog-demos/tree/master/composite-images-in-ruby)

