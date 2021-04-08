---
title: 微信公众号指定openid发送文本信息
date: 2020-10-21 14:39:49
keyword: 微信公众号
description: 微信公众号通过客服接口，给指定openid发送文本消息
cover: https://gitee.com/float97/wutang-imgs/raw/master/20201021144250.png
tags:
- 微信公众号
categories:
- 整合
---



##### 背景

公司需要给指定用户发送文本信息，[微信官方文档](http://caibaojian.com/wxwiki/d475b3690426f2a7231ca605ab24d485ac9c623e.html)写的是用户主动操作后，客户才能回复消息，所以摸索了下。

##### 步骤

1. 用户需要关注当前公众号
2. 公众号配置
3. 获取当前公众号的access_token
4. 通过客服接口发送信息

##### 公众号配置

登录到公众号后台，进入到开发的基本配置，<font color=red>记录下AppId和AppSecret,并且将服务器（接口调用）的ip地址配置到IP白名单。</font>

![image-20201021140805395](https://gitee.com/float97/wutang-imgs/raw/master/20201021140811.png)



##### 代码

1. 获得access_token

```java
 /**
     * 获取access_token 后续发送请求会需要这个access_token作为参数传入
     */
    public ResultObject<Void> getAccessToken() {
        //我这边是存入redis当中的
        Object o = redisUtil.get(CataLogEnum.WX_AUTH.getValue());
        //通过appId和appSercret获取access_token
        String url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=" + wxPayConfig.getAppID() +
                "&secret=" + wxPayConfig.getAppSecret();
        String content = "";
        int count = 0;
        log.info("获取access:{}", o);
        if (o == null) {
            while (true) {
                count++;
//                这需要吗？
                if (count > 7) {
                    return ResultObject.error("获取微信access_token超时,请稍后重试");
                }
                try {
                    //构建一个Client
                    HttpClient client = HttpClientBuilder.create().build();
                    //构建一个GET请求
                    HttpGet get = new HttpGet(url.toString());
                    //提交GET请求
                    HttpResponse response = client.execute(get);
                    //拿到返回的HttpResponse的"实体"
                    HttpEntity result = response.getEntity();
                    content = EntityUtils.toString(result);
                    JSONObject json = JSON.parseObject(content);
                    log.info("或者access的json:{}" + json);
                    if (json.getString("errcode") != null && !"0".equals(json.getString("errcode"))) {
//                    发送失败
                        return ResultObject.error("获取微信access_token错误:[" + json.getString("errcode") + "]");
                    }

                    WxAutuDto wxAutuDto = JSONObject.parseObject(content, WxAutuDto.class);
                    log.info("wxAutuDto:{}", wxAutuDto);
//                    存入redis
                    boolean set = redisUtil.set(CataLogEnum.WX_AUTH.getValue(), content, wxAutuDto.getExpires_in());
                    log.info("access存入redis:{}", set);
                    if (set) {
                        return ResultObject.ok("获取成功");
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

        return ResultObject.ok("获取成功");
    }
```



2. 发送信息

   ```java
   	/**
        * 发送信息
        */
       public ResultObject<Void> sendMessage(ReqSendWxMessageDto dto) {
   
           String result = "";
   //        获得token
           String o = (String) redisUtil.get(CataLogEnum.WX_AUTH.getValue());
           WxAutuDto wxAutuDto = JSONObject.parseObject(o, WxAutuDto.class);
   //        发送的url 需要获得token传入
           String sendUrl = "https://api.weixin.qq.com/cgi-bin/message/custom/send?access_token=" + wxAutuDto.getAccess_token();
   //        发送实体类
           List<String> openIds = dto.getOpenIds();
   //        发送的消息
           Map<String, Object> map = new HashMap<>();
           map.put("content", dto.getContent());
   //        循环
           for (String m : openIds) {
               WxMessageDto testMessage = new WxMessageDto();
               //        暂时是text类型,需要其他的自己扩展
               testMessage.setMsgtype("text");
               testMessage.setTouser(m);
               testMessage.setText(map);
               String jsonTestMessage = JSONObject.toJSONString(testMessage);
               int count = 1;
               while (true) {
                   try {
   //                发送post请求
                       log.info("开始发送请求");
                       result = WxMessageUtils.sendPost(sendUrl, jsonTestMessage);
                       JSONObject json = JSON.parseObject(result);
                       log.info("返回:{}", json);
                       if (!"0".equals(json.getString("errcode"))) {
   //                    发送失败
                           return ResultObject.error("发送失败");
                       } else {
                           break;
                       }
                   } catch (Exception e) {
                       e.printStackTrace();
                   } finally {
                       if (count >= 5) {
                           return ResultObject.error("发送失败");
                       }
                       count++;
                   }
               }
           }
   
           return ResultObject.ok("发送成功");
       }
   
   
   /**
        * post 请求
        *
        * @param url
        * @param param
        * @return
        */
       public static String sendPost(String url, String param) {
           PrintWriter out = null;
           BufferedReader in = null;
           String result = "";
           try {
               URL realUrl = new URL(url);
               // 打开和URL之间的连接
               URLConnection conn = realUrl.openConnection();
               //设置通用的请求属性
               conn.setRequestProperty("user-agent", "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:21.0) Gecko/20100101 Firefox/21.0)");
               // 发送POST请求必须设置如下两行
               conn.setDoOutput(true);
               conn.setDoInput(true);
               // 获取URLConnection对象对应的输出流
               OutputStreamWriter outWriter = new OutputStreamWriter(conn.getOutputStream(), "utf-8");
               out = new PrintWriter(outWriter);
               // 发送请求参数
               out.print(param);
               // flush输出流的缓冲
               out.flush();
               // 定义BufferedReader输入流来读取URL的响应
               in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
               String line;
               while ((line = in.readLine()) != null) {
                   in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                   result += line;
               }
           } catch (Exception e) {
               log.info("发送 POST 请求出现异常:{}" + e);
               e.printStackTrace();
           }
           //使用finally块来关闭输出流、输入流
           finally {
               try {
                   if (out != null) {
                       out.close();
                   }
                   if (in != null) {
                       in.close();
                   }
               } catch (IOException ex) {
                   ex.printStackTrace();
               }
           }
           return result;
       }
   ```

如果有报错，请查看对应的[状态码地址](http://caibaojian.com/wxwiki/c69a3e39afca9a7285a83ebc9cbd44d940e7f916.html)



完整的代码

Controller

```java
package com.yx.controller.test;

import com.yx.tool.ResultObject;
import com.yx.tool.wxMessage.ReqSendWxMessageDto;
import com.yx.tool.wxMessage.WxMessageUtils;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author ：tangfan
 * @date ：Created in 2020/10/21 10:00
 * @description：
 * @modified By：
 */
@Api(tags = "微信消息")
@RestController
@RequestMapping("/api/wxMessage")
public class WxMessageController {
    @Autowired
    private WxMessageUtils wxMessageUtils;

    @PostMapping("/getAuth")
    @ApiOperation(value = "首先获取微信公众号的token，有过期时间的", notes = "tf")
    public ResultObject<Void> getAuth() {
        return wxMessageUtils.getAccessToken();
    }

    @PostMapping("/sendMessage")
    @ApiOperation(value = "通过用户openId发送微信消息", notes = "tf")
    public ResultObject send(@RequestBody ReqSendWxMessageDto dto) {
        return wxMessageUtils.sendMessage(dto);
    }

}

```

获得access_token和发送消息的工具类

```java
package com.yx.tool.wxMessage;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.yx.tool.CataLogEnum;
import com.yx.tool.RedisUtil;
import com.yx.tool.ResultObject;
import com.yx.tool.wx.wxPay.WXPayConfig;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.util.EntityUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.io.*;
import java.net.URL;
import java.net.URLConnection;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author ：tangfan
 * @date ：Created in 2020/10/21 8:53
 * @description： 公众号发送消息工具类
 * @modified By：
 */
@Component
@Slf4j
public class WxMessageUtils {
    @Autowired
    private WXPayConfig wxPayConfig;
    @Autowired
    private RedisUtil redisUtil;


    /**
     * post 请求
     *
     * @param url
     * @param param
     * @return
     */
    public static String sendPost(String url, String param) {
        PrintWriter out = null;
        BufferedReader in = null;
        String result = "";
        try {
            URL realUrl = new URL(url);
            // 打开和URL之间的连接
            URLConnection conn = realUrl.openConnection();
            //设置通用的请求属性
            conn.setRequestProperty("user-agent", "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:21.0) Gecko/20100101 Firefox/21.0)");
            // 发送POST请求必须设置如下两行
            conn.setDoOutput(true);
            conn.setDoInput(true);
            // 获取URLConnection对象对应的输出流
            OutputStreamWriter outWriter = new OutputStreamWriter(conn.getOutputStream(), "utf-8");
            out = new PrintWriter(outWriter);
            // 发送请求参数
            out.print(param);
            // flush输出流的缓冲
            out.flush();
            // 定义BufferedReader输入流来读取URL的响应
            in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
            String line;
            while ((line = in.readLine()) != null) {
                in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                result += line;
            }
        } catch (Exception e) {
            log.info("发送 POST 请求出现异常:{}" + e);
            e.printStackTrace();
        }
        //使用finally块来关闭输出流、输入流
        finally {
            try {
                if (out != null) {
                    out.close();
                }
                if (in != null) {
                    in.close();
                }
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
        return result;
    }

    /**
     * 获取access_token 后续发送请求会需要这个access_token作为参数传入
     */
    public ResultObject<Void> getAccessToken() {
        Object o = redisUtil.get(CataLogEnum.WX_AUTH.getValue());
        String url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=" + wxPayConfig.getAppID() +
                "&secret=" + wxPayConfig.getAppSecret();
        String content = "";
        int count = 0;
        log.info("获取access:{}", o);
        if (o == null) {
            while (true) {
                count++;
//                这需要吗？
                if (count > 7) {
                    return ResultObject.error("获取微信access_token超时,请稍后重试");
                }
                try {
                    //构建一个Client
                    HttpClient client = HttpClientBuilder.create().build();
                    //构建一个GET请求
                    HttpGet get = new HttpGet(url.toString());
                    //提交GET请求
                    HttpResponse response = client.execute(get);
                    //拿到返回的HttpResponse的"实体"
                    HttpEntity result = response.getEntity();
                    content = EntityUtils.toString(result);
                    JSONObject json = JSON.parseObject(content);
                    log.info("或者access的json:{}" + json);
                    if (json.getString("errcode") != null && !"0".equals(json.getString("errcode"))) {
//                    发送失败
                        return ResultObject.error("获取微信access_token错误:[" + json.getString("errcode") + "]");
                    }

                    WxAutuDto wxAutuDto = JSONObject.parseObject(content, WxAutuDto.class);
                    log.info("wxAutuDto:{}", wxAutuDto);
//                    存入redis
                    boolean set = redisUtil.set(CataLogEnum.WX_AUTH.getValue(), content, wxAutuDto.getExpires_in());
                    log.info("access存入redis:{}", set);
                    if (set) {
                        return ResultObject.ok("获取成功");
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

        return ResultObject.ok("获取成功");
    }

    /**
     * 发送信息
     */
    public ResultObject<Void> sendMessage(ReqSendWxMessageDto dto) {

        String result = "";
//        获得token
        String o = (String) redisUtil.get(CataLogEnum.WX_AUTH.getValue());
        WxAutuDto wxAutuDto = JSONObject.parseObject(o, WxAutuDto.class);
//        发送的url
        String sendUrl = "https://api.weixin.qq.com/cgi-bin/message/custom/send?access_token=" + wxAutuDto.getAccess_token();
//        发送实体类
        List<String> openIds = dto.getOpenIds();
//        发送的消息
        Map<String, Object> map = new HashMap<>();
        map.put("content", dto.getContent());
//        循环
        for (String m : openIds) {
            WxMessageDto testMessage = new WxMessageDto();
            //        暂时是text类型,需要其他的自己扩展
            testMessage.setMsgtype("text");
            testMessage.setTouser(m);
            testMessage.setText(map);
            String jsonTestMessage = JSONObject.toJSONString(testMessage);
            int count = 1;
            while (true) {
                try {
//                发送post请求
                    log.info("开始发送请求");
                    result = WxMessageUtils.sendPost(sendUrl, jsonTestMessage);
                    JSONObject json = JSON.parseObject(result);
                    log.info("返回:{}", json);
                    if (!"0".equals(json.getString("errcode"))) {
//                    发送失败
                        return ResultObject.error("发送失败");
                    } else {
                        break;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    if (count >= 5) {
                        return ResultObject.error("发送失败");
                    }
                    count++;
                }
            }
        }

        return ResultObject.ok("发送成功");
    }
}

```



dto

```java
package com.yx.tool.wxMessage;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.util.Map;

/**
 * @author ：tangfan
 * @date ：Created in 2020/10/21 8:50
 * @description： 客户接口消息发送实体
 * @modified By：
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class WxMessageDto implements Serializable {

    //openid
    private String touser;

    //消息类型
    private String msgtype;

    //消息内容
    private Map<String, Object> text;

}

```



最后在方便测试的时候，你可以在公众号的用户管理直接找到用户的openId

![image-20201021142922334](https://gitee.com/float97/wutang-imgs/raw/master/20201021142922.png)



测试成功，可以指定多个用户

![image-20201021143044809](https://gitee.com/float97/wutang-imgs/raw/master/20201021143044.png)

