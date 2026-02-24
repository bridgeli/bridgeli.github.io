---
title: 微信JS SDK Demo（Java版）
author: Bridge Li
type: post
date: 2015-03-15T14:17:09+00:00

duoshuo_thread_id:
  - 1.1604454626757E+18
categories:
  - 微信
tags:
  - JS SDK
  - 微信

---
前几天在公司开发一个功能：微信分享，要求是分享的不是用户当前看到的这个页面，大家知道这在以前其实很简单的，但去年的最后一天，微信大力打击诱导分享、关注之后，以前的分享就不能用了，好在后来微信开放了JS SDK接口，可以满足这个需求，由于网上的例子写的都很简单，而且大多都是php，今天老夫就记录一下自己用Java如何实现的这个功能，窃以为比网上的很多Demo更具有实用性，大家在使用中如果有什么疑问，欢迎留言交流。废话到此为止，下面看看如何实现，当然了首先肯定要参考微信的文档：http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html，其实这个文档上面说的已经比较详细了，但是距离真正使用当然还是有一定的距离的，在说老夫的代码之前，再多说一句，想调用微信的JS SDK，请确保你有一个经过微信认证的服务号，否则你是调用不了的，好，下面就看代码了：

```  
$(&#8216;.share&#8217;).tap(function(){
    var urlCurrery = window.location.href.split(&#8216;#&#8217;)[0];
    $.ajax({
            type : "post",
            url : &#8216;share.do&#8217;,
    data : {&#8216;urlCurrery&#8217;:urlCurrery},
    success : function(data) {
        var dataObj = eval(data)[0];
        var url =dataObj.url;
        var jsapi_ticket =dataObj.jsapi_ticket;
        var nonce_str =dataObj.nonceStr;
        var timestamp =dataObj.timestamp;
        var signature =dataObj.signature;

        wx.config({
                debug: true,
                appId: appId;,
        timestamp: timestamp,
                nonceStr: nonce_str,
                signature: signature,
                jsApiList: [  
&#8216;checkJsApi&#8217;,  
&#8216;onMenuShareTimeline&#8217;,  
&#8216;onMenuShareAppMessage&#8217;  
]  
});

        wx.ready(function () {
            wx.onMenuShareAppMessage({
                    title: &#8216;分享测试&#8217;,
            desc: &#8216;分享测试&#8217;,
            link: &#8216;https://bridgeli.cn&#8217;,  
            imgUrl: &#8221;,

            success: function (res) {
                alert(&#8216;已分享&#8217;);
            }  
});
            wx.onMenuShareTimeline({
                    title: &#8216;分享测试&#8217;,
            link: &#8216;https://bridgeli.cn&#8217;,  
            imgUrl: &#8221;,

            success: function (res) {
                alert(&#8216;已分享&#8217;);
            }  
});
            alert(&#8216;已注册获取“发送给朋友”状态事件&#8217;);
            alert(&#8216;已注册获取“分享到朋友圈”状态事件&#8217;);
        });
    }  
});

});  
```

经过分析微信的文档我们知道：如果想使用分享功能，那么请给分享注册事件，所以我们这里给文本中一个class为：share 的dom，绑定了分享事件，也就是说，我们想改变分享url，必须先点击一下该button，否则是不行的；另外通过文档里面的常见错误，我们可以知道，参与签名的url必须是动态获取的，如果不是，请通过ajax传到后台，为了强调这一点，微信专门用红色标注了，所以老夫在给class为share的button绑定事件的同时，将当前页面的 url （文中用：urlCurrery表示的）通用ajax传到后台 share.do 参与签名，我们一会再说后台怎么实现，接着分析前台代码，在经过后台签名之后，后台会传给前台一些数据，封装在data里面，这些就是我们需要的数据，相信通过微信的文档，所有人都能看得明白，就不多做解释了，在 wx.ready()方法里，老夫举了两个例子分别是分享给朋友和分享到朋友圈，其中的参数 link 就是你要分享的 url ，这一点很重要，这样就实现了我们分析的不是当前页面的 url 的需求，好了前台的说完了，下面就要看后台是怎么实现的了，这个其实很简单

```  
package cn.bridgeli.demo;

public class Share {
    public String share() {
        String url = request().getParameter("urlCurrery");
        Map<String, String> resultMap = getResult(url);
        String result = JSONUtils.toJSONString(resultMap);
        JSONUtils.printStr(result);
        return null;

    }

    private Map<String, String> getResult(String url) {
        String access_token = TokenUtil.getAccessToken();
        String jsapi_ticket = TokenUtil.getJsapiTicket(access_token);

        Map<String, String> result = Sign.sign(jsapi_ticket, url);
        return result;
    }
}  
```

这个类很明显，其实就是一个controller，用来接收前端ajax传来的参数，其实就一个url，然后签名，并把签名之后得到的数据返回回去，比较简单，其中 share() 方法的最后三行，大家可能看不懂，无所谓，这是我们公司分装的一个像前台返回数据的一个方法，大家自己正常写就行，另外那个私有的方法 getResult() 的方法调用了其他工具类的方法，这里面有两点要注意的：1、access_token，jsapi_ticket一定要缓存（文档里面说的有），2、至于怎么获取，老夫以前的文章，怎么发起一个http request 和 https request请求里面说过，不会的可以自己查看，拿到这两个参数之后，下面最重要的其实就是 sign() 方法了，这个方法其实微信官方Demo里面已经给出了，老夫就是完整的copy过来的，现在还完整的贴在下面：

```  
package cn.bridgeli.demo;

import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Formatter;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

public class Sign {
    public static Log log = LogFactory.getLog(Sign.class);

    public static Map<String, String> sign(String jsapi_ticket, String url) {
        Map<String, String> ret = new HashMap<String, String>();
        String nonce_str = create_nonce_str();
        String timestamp = create_timestamp();
        String string1;
        String signature = "";

// 注意这里参数名必须全部小写，且必须有序  
        string1 = "jsapi_ticket=" + jsapi_ticket + "&noncestr=" + nonce_str + "&timestamp=" + timestamp + "&url=" + url;
        log.info("string1 ===> " + string1);

        try {
            MessageDigest crypt = MessageDigest.getInstance("SHA-1");
            crypt.reset();
            crypt.update(string1.getBytes("UTF-8"));
            signature = byteToHex(crypt.digest());
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            log.error(e);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            log.error(e);
        }

        ret.put("url", url);
        ret.put("jsapi_ticket", jsapi_ticket);
        ret.put("nonceStr", nonce_str);
        ret.put("timestamp", timestamp);
        ret.put("signature", signature);
        log.info("signature ===> " + signature);
        return ret;
    }

    private static String byteToHex(final byte[] hash) {
        Formatter formatter = new Formatter();
        for (byte b : hash) {
            formatter.format("%02x", b);
        }
        String result = formatter.toString();
        formatter.close();
        return result;
    }

    private static String create_nonce_str() {
        return UUID.randomUUID().toString();
    }

    private static String create_timestamp() {
        return Long.toString(System.currentTimeMillis() / 1000);
    }

}  
```

这里面老夫就是添加了log，其他的一点未动，相信结合老夫的这篇文章和微信的官方文档，大家一定可以实现自己了自己的需求，如果有什么问题，欢迎大家留言交流，一块学习成长，最后多说一句，大家一定要自己看文档，按照文档要求的来写，就像这次，老夫在写这个功能的有一点卡克，当时老夫对照文档，有一点是不符合文档的，老夫不知道怎么写才符合文档，所以老夫定位了一定是这一点错，但老夫的老大却说，文档上面虽然是这么写的，但是不一定啊不一定啊，是不是其他地方导致的错误，巴拉巴拉一堆，最终浪费了很多无谓的时间，老夫想说的是：如果连文档你都不相信，那么这个功能你还怎么做？完全就没法做了，因为没有文档，我们就没有一点参考资料了啊，因为网上的那些资料的作者参考的资料肯定也是文档，既然他们能做出来，那么说明文档一定是没有大问题的