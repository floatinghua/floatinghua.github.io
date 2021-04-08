---
title: Linux和Windows下集成海康摄像头(特别解决linux加载不到库)
date: 2020-07-01 11:38:55
keyword: linux windows springboot 海康摄像头
description: Linux和Windows下集成海康摄像头(特别解决linux加载不到库)
cover: https://img-blog.csdnimg.cn/20200528174109430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70
tags: 
- springboot
- 集成
- 工具
- 海康
categories: 
- linux
---

#### 问题

最近要集成海康摄像头部署到linux服务器，但是一直调用不到so文件，所以记录下解决过程，
由于自己不熟悉jna，所以请轻喷，**Stay Hungry,Stay Foolish.**

[海康sdk官方下载地址](https://www.hikvision.com/cn/download_61.html)


#### 前期准备

****至于为什么先讲linux等下就知道了****，linux的sdk下载下来目录如下：
![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Ra7VlK6d-1590658348978)(C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200528163929984.png)\]](https://img-blog.csdnimg.cn/20200528173418663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)

开发文档有自己的业务需求就看对应的开放文档

**我们直接看LinuxJavaDemo文件夹**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528173443515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)

**这边我选择的是方法一，但是不是在系统的/usr/lib文件下，自己新建的文件夹**

#### linux准备工作

1. linux下搭配jdk1.8，我这边linux服务器新建了一个目录（**/home/opt/hcnet**）来存放下面的so文件

2. 上传文件到linux目录下  就是图中的lib文件,**所有的都上传到你的linux目录** 

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528173517359.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)

3. 创建lib文件夹引入sdk包中的examples.jar和jna.jar
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528173551673.png)
   在pom中添加 不然会报错

   ```xml
   <dependency>
               <groupId>com.sun.jna</groupId>
               <artifactId>yx</artifactId>
               <version>0.0.1</version>
               <scope>system</scope>
               <systemPath>${project.basedir}/lib/jna.jar</systemPath>
           </dependency>
           <dependency>
               <groupId>com.sun.jna.examples</groupId>
               <artifactId>yx</artifactId>
               <version>0.0.1</version>
               <scope>system</scope>
               <systemPath>${project.basedir}/lib/examples.jar</systemPath>
           </dependency>
   ```

   打包添加如下配置
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528173624768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)

4. 开始集成

   **将linux中的HCNetSDK.java放到项目中，修改代码，将他初始化的地方注释掉，我们自己初始化**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528173635709.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
下面就是自己的初始化代码

```java
package com.yx.hkdemo.config;

import com.sun.jna.Native;
import com.sun.jna.NativeLong;
import com.sun.jna.Platform;
import com.yx.hkdemo.HCNetSDK;

import java.io.File;

/**
 * @author ：tangfan
 * @date ：Created in 2020/5/25 17:36
 * @description：
 * @modified By：
 */
public class GetHCNetSdk {
    public static HCNetSDK hcNetSDK = null;
    //windows下的路径
    String PATH_WIN = System.getProperty("user.dir") + File.separator + "hkwinlib" + File.separator + "HCNetSDK";
    String PATH_LINUX = "/home/opt/hcnet/libhcnetsdk.so";
    private HCNetSDK.NET_DVR_DEVICEINFO_V30 deviceInfo;//设备信息
    private NativeLong lUserID;//用户句柄
    private NativeLong lAlarmHandle;//报警布防句柄
    //    private HCNetSDK.FMSGCallBack_V31 fMSFCallBack_V31;//报警回调函数实现
    private HCNetSDK.FMSGCallBack fMSFCallBack;//报警回调函数实现
    private String deviceIP;//已登录设备的IP地址
    private int devicePort;//设备端口号
    private String username;//设备用户名
    private String password;//设备登陆密码
    private boolean init_flag;//初始化识别标志
    private boolean reg_flag;//设备注册识别标志

    public GetHCNetSdk() {
        install();
    }

    private void install() {
        if (Platform.isWindows()) {
            hcNetSDK = (HCNetSDK) Native.loadLibrary(PATH_WIN, HCNetSDK.class);
            
        }
        if (Platform.isLinux()) {
            hcNetSDK = (HCNetSDK) Native.loadLibrary(PATH_LINUX, HCNetSDK.class);
           
        }
    }
}

```

**如果在linux上运行报错，请在linux下的hcNetSDK = (HCNetSDK) Native.loadLibrary(PATH_LINUX, HCNetSDK.class);后面加入官方的这段代码，并且修改成自己的路径，我的是/home/opt/hcnet**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528173704738.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)
**代码在下面：**

```java
  //设置HCNetSDKCom组件库所在路径
//        /home/opt/hcnet/libhcnetsdk.so
//        /home/opt/hcnet
            String strPathCom = "/home/opt/hcnet";
            HCNetSDK.NET_DVR_LOCAL_SDK_PATH struComPath = new HCNetSDK.NET_DVR_LOCAL_SDK_PATH();
            System.arraycopy(strPathCom.getBytes(), 0, struComPath.sPath, 0, strPathCom.length());
            struComPath.write();
            hcNetSDK.NET_DVR_SetSDKInitCfg(2, struComPath.getPointer());

//设置libcrypto.so所在路径
            HCNetSDK.BYTE_ARRAY ptrByteArrayCrypto = new HCNetSDK.BYTE_ARRAY(256);
            String strPathCrypto = "/home/opt/hcnet/libcrypto.so";
            System.arraycopy(strPathCrypto.getBytes(), 0, ptrByteArrayCrypto.byValue, 0, strPathCrypto.length());
            ptrByteArrayCrypto.write();
            hcNetSDK.NET_DVR_SetSDKInitCfg(3, ptrByteArrayCrypto.getPointer());

//设置libssl.so所在路径
            HCNetSDK.BYTE_ARRAY ptrByteArraySsl = new HCNetSDK.BYTE_ARRAY(256);
            String strPathSsl = "/home/opt/hcnet/libssl.so";
            System.arraycopy(strPathSsl.getBytes(), 0, ptrByteArraySsl.byValue, 0, strPathSsl.length());
            ptrByteArraySsl.write();
            hcNetSDK.NET_DVR_SetSDKInitCfg(4, ptrByteArraySsl.getPointer());
```




5. 编写工具类，自己写方法


```java
package com.yx.hkdemo;

import com.sun.jna.NativeLong;
import com.sun.jna.Pointer;
import com.sun.jna.ptr.IntByReference;
import com.sun.jna.ptr.NativeLongByReference;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.swing.*;

//import org.apache.commons.lang.StringUtils;

//import cc.eguid.FFmpegCommandManager.FFmpegManager;
//import cc.eguid.FFmpegCommandManager.FFmpegManagerImpl;
//import com.dfzx.common.util.CommonKit;
@Data
@AllArgsConstructor
public class HCNetTools {
    HCNetSDK hCNetSDK = null;
    HCNetSDK.NET_DVR_DEVICEINFO_V30 m_strDeviceInfo;//设备信息
    HCNetSDK.NET_DVR_IPPARACFG m_strIpparaCfg;//IP参数
    HCNetSDK.NET_DVR_CLIENTINFO m_strClientInfo;//用户参数
    boolean bRealPlay;//是否在预览.
    String m_sDeviceIP;//已登录设备的IP地址
    NativeLong lUserID;//用户句柄
    NativeLong lPreviewHandle;//预览句柄
    NativeLongByReference m_lPort;//回调预览时播放库端口指针

    public HCNetTools(HCNetSDK sdk) {
        JPopupMenu.setDefaultLightWeightPopupEnabled(false);//防止被播放窗口(AWT组件)覆盖
        lUserID = new NativeLong(-1);
        lPreviewHandle = new NativeLong(-1);
        m_lPort = new NativeLongByReference(new NativeLong(-1));
        this.hCNetSDK = sdk;
    }


//    FFmpegManager manager;//rstp转rmtp工具


    //FRealDataCallBack fRealDataCallBack;//预览回调函数实现

    public HCNetTools() {
        JPopupMenu.setDefaultLightWeightPopupEnabled(false);//防止被播放窗口(AWT组件)覆盖
        lUserID = new NativeLong(-1);
        lPreviewHandle = new NativeLong(-1);
        m_lPort = new NativeLongByReference(new NativeLong(-1));
        //fRealDataCallBack= new FRealDataCallBack();
    }

    /**
     * 初始化资源配置
     */
    public int initDevices() {
        if (!hCNetSDK.NET_DVR_Init()) return 1;//初始化失败
        return 0;
    }

    /**
     * 设备注册
     *
     * @param name     设备用户名
     * @param password 设备登录密码
     * @param ip       IP地址
     * @param port     端口
     * @return 结果
     */
    public int deviceRegist(String name, String password, String ip, String port) {
        if (bRealPlay) {//判断当前是否在预览
            return 2;//"注册新用户请先停止当前预览";
        }
        if (lUserID.longValue() > -1) {//先注销,在登录
            hCNetSDK.NET_DVR_Logout_V30(lUserID);
            lUserID = new NativeLong(-1);
        }
        System.out.println("注册里面:lUserId:"+lUserID);
        //注册(既登录设备)开始
        m_sDeviceIP = ip;
        m_strDeviceInfo = new HCNetSDK.NET_DVR_DEVICEINFO_V30();//获取设备参数结构
        System.out.println("注册里面：ip:"+m_sDeviceIP+"     info:"+m_strDeviceInfo);
        lUserID = hCNetSDK.NET_DVR_Login_V30(m_sDeviceIP, (short) Integer.parseInt("8000"), name, password, m_strDeviceInfo);//登录设备
//        NativeLong lUserID, int dwCommand, NativeLong lChannel, Pointer lpInBuffer, int dwInBufferSize
        System.out.println("登录后的id"+lUserID);
        int i = hCNetSDK.NET_DVR_GetLastError();

        System.out.println("登录错误代码："+i);
        m_strIpparaCfg = new HCNetSDK.NET_DVR_IPPARACFG();
        Pointer lpIpParaConfig = m_strIpparaCfg.getPointer();
        boolean b = hCNetSDK.NET_DVR_SetDVRConfig(lUserID, HCNetSDK.NET_DVR_GET_IPPARACFG, new NativeLong(0), lpIpParaConfig, m_strIpparaCfg.size());
        int seti = hCNetSDK.NET_DVR_GetLastError();
        System.out.println("setConfig"+seti);
        System.out.println("b:"+b);
        long userID = lUserID.longValue();
        System.out.println("注册里面：userId:"+userID);
        if (userID == -1) {
            m_sDeviceIP = "";//登录未成功,IP置为空
            return 3;//"注册失败";
        }

        return 0;
    }

    /**
     * 获取设备通道
     */
    public int getChannelNumber() {
        IntByReference ibrBytesReturned = new IntByReference(0);//获取IP接入配置参数
        boolean bRet = false;
        int iChannelNum = -1;

        m_strIpparaCfg = new HCNetSDK.NET_DVR_IPPARACFG();
        m_strIpparaCfg.write();
        Pointer lpIpParaConfig = m_strIpparaCfg.getPointer();

        bRet = hCNetSDK.NET_DVR_GetDVRConfig(lUserID, HCNetSDK.NET_DVR_GET_IPPARACFG, new NativeLong(0), lpIpParaConfig, m_strIpparaCfg.size(), ibrBytesReturned);
        int geti = hCNetSDK.NET_DVR_GetLastError();
        System.out.println("geti:"+geti);
        m_strIpparaCfg.read();

        String devices = "";
        System.out.println("bRet:"+bRet);
        if (!bRet) {
            //设备不支持,则表示没有IP通道
            System.out.println("C:"+m_strDeviceInfo.byChanNum);
            for (int iChannum = 0; iChannum < m_strDeviceInfo.byChanNum; iChannum++) {
                devices = "Camera" + (iChannum + m_strDeviceInfo.byStartChan);
            }
        } else {
            System.out.println("IP"+HCNetSDK.MAX_IP_CHANNEL);
            for (int iChannum = 0; iChannum < HCNetSDK.MAX_IP_CHANNEL; iChannum++) {
                if (m_strIpparaCfg.struIPChanInfo[iChannum].byEnable == 1) {
                    devices = "IPCamera" + (iChannum + m_strDeviceInfo.byStartChan);
                }
            }
        }
        System.out.println("通道："+devices);
        if (devices != null && devices != "") {
            if (devices.charAt(0) == 'C') {//Camara开头表示模拟通道
                //子字符串中获取通道号
                System.out.println("C");
                iChannelNum = Integer.parseInt(devices.substring(6));
            } else {
                if (devices.charAt(0) == 'I') {//IPCamara开头表示IP通道
                    System.out.println("I");
                    //子字符创中获取通道号,IP通道号要加32
                    iChannelNum = Integer.parseInt(devices.substring(8)) + 32;
                } else {
                    return 4;
                }
            }
        }
        return iChannelNum;
    }

    /**
     * 获取通道信号状态
     *
     * @param channum
     * @return
     */
    public int getSignalStatus(int channum) {
        //获取设备状态
        HCNetSDK.NET_DVR_WORKSTATE_V30 devwork = new HCNetSDK.NET_DVR_WORKSTATE_V30();
        if (!hCNetSDK.NET_DVR_GetDVRWorkState_V30(lUserID, devwork)) {
            //返回Boolean值，判断是否获取设备能力
            System.out.println("返回设备状态失败");
        }
        return devwork.struChanStatic[channum].bySignalStatic;
    }

    /**
     * 拍照
     *
     * @return
     */
    public boolean takePic() {
        //拍照
        HCNetSDK.NET_DVR_JPEGPARA strJpeg = new HCNetSDK.NET_DVR_JPEGPARA();
        strJpeg.wPicQuality = 1; //图像参数
        strJpeg.wPicSize = 2;
        String filePath = "E:\\123q.jpg";
        lPreviewHandle.setValue(m_strDeviceInfo.byStartChan + 32);
        boolean b = hCNetSDK.NET_DVR_CaptureJPEGPicture(lUserID, lPreviewHandle, strJpeg, filePath);//尝试用NET_DVR_CaptureJPEGPicture_NEW方法，但不是报43就是JDK崩溃....
        if (!b) {//单帧数据捕获图片
            System.out.println("抓拍失败!" + " err: " + hCNetSDK.NET_DVR_GetLastError());
        } else {
            System.out.println("抓拍成功");
        }
        return b;
    }

    /**
     * 释放SDK
     */
    public void shutDownDev() {
        //如果已经注册,注销
        if (lUserID.longValue() > -1) {
            hCNetSDK.NET_DVR_Logout_V30(lUserID);
        }
        hCNetSDK.NET_DVR_Cleanup();
    }

}

```

   

   

   分别调用就行了

   Linux下运行成功的截图
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052817405633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)


#### win准备和集成

上面linux都集成好了，windows就只是需要修改一点东西就好了

因为我下载的是最新的linux SDK所以HCNetSDK.java就不需要做更改

1. 继续新建文件夹 引入win下面的dll文件 全部引入

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528174109430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQxNzIxMQ==,size_16,color_FFFFFF,t_70)


2. 因为上面的初始化类判断了linux和windows系统的，所以直接调用就可以了，因为linux下的sdk中的方法和win的方法基本一致，所以直接用就行了
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200528174502106.png)



#### 总结

**集成linux出现的问题就是加载不到库，在摸索官方的sdk后，终于成功，还是自己太年轻，sdk看得不够多**