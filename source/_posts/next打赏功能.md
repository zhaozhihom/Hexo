---
title: next主题打赏文字改成汉字&页面底部总字数添加汉字
date: 2017-02-28 17:50:33
categories:
tags:
    - hexo
---

#### 打赏文字修改 

我在按照[官网的方法][1]在next主题下加了打赏功能，但是文字都是英文，在我Ctrl+F了多个文件后终于找到修改的地方，记录一下：

<!--more-->

打开文件`themes\next\layout\_macro\reward.swig`，修改`赏，微信打赏，支付宝打赏`等地方(下面是我改过的，原来是英文)，效果就是你现在看到的这样了。

```html
<div style="padding: 10px 0; margin: 20px auto; width: 90%; text-align: center;">
  <div>{{ theme.reward_comment }}</div>
  <button id="rewardButton" disable="enable" onclick="var qr = document.getElementById('QR'); if (qr.style.display === 'none') {qr.style.display='block';} else {qr.style.display='none'}">
    <span>赏</span>
  </button>
  <div id="QR" style="display: none;">

    {% if theme.wechatpay %}
      <div id="wechat" style="display: inline-block">
        <img id="wechat_qr" src="{{ theme.wechatpay }}" alt="{{ theme.author }} WeChat Pay"/>
        <p>微信打赏</p>
      </div>
    {% endif %}

    {% if theme.alipay %}
      <div id="alipay" style="display: inline-block">
        <img id="alipay_qr" src="{{ theme.alipay }}" alt="{{ theme.author }} Alipay"/>
        <p>支付宝打赏</p>
      </div>
    {% endif %}

    {% if theme.bitcoin %}
      <div id="bitcoin" style="display: inline-block">
        <img id="bitcoin_qr" src="{{ theme.bitcoin }}" alt="{{ theme.author }} Bitcoin"/>
        <p>Bitcoin</p>
      </div>
    {% endif %}
  </div>
</div>
```

#### 字数统计添加文字

此外还有个相同的问题，网站底部总字数统计没有汉字，只有数字，这个要修改`\themes\next\layout\_partials\footer.swig`文件：

在totalcount(site, '0,0.0a')前后加相应的汉字即可。
  [1]: http://theme-next.iissnan.com/theme-settings.html#reward