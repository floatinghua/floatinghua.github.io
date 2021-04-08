---
title: windows使用docker搭建minio
date: 2021-03-30 16:11:40
description: windows使用docker搭建minio
cover: http://cdn.wutang.cc/image-20210401170355565.png
tags:
- docker
- maven
- minio
categories:
- java

---

官方文档：[MinIO | 高性能，对Kubernetes友好的对象存储](http://www.minio.org.cn/)

[详解块存储、文件存储、对象存储区别 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/280128756)

>总结来说用对象储存的好处：
>
>1. 对象储存可扩展性高，储存方式类似于hash表
>2. 对象储存无需迁移，横向扩展，自动计算储存节点
>3. 安全性高 ，
>4. 访问方便：不光支持HTTP(S)协议，采用REST的API方式调用和检索数据，同样增加了NFS和SMB支持；
>5. 成本相对低
>6. 效率高，没有复杂的文件系统

当然要是有钱买个oss服务也是可以的，只是本着不花钱的原则嘛，就只能折腾了 

#### 安装步骤

比较简单

##### 简介

-e "MINIO_ACCESS_KEY=TANG" 指定账号

-e "MINIO_SECRET_KEY=QWE123BNM" 指定密码

-v 指定挂载目录 minio上传的文件会在data目录下

```shell
docker run -p 9000:9000 --name minio -e "MINIO_ACCESS_KEY=TANG" -e "MINIO_SECRET_KEY=QWE123BNM" -v F:\minio\minio\data:/data -v F:\minio\minio\config:/root/.minio minio/minio server /data
```

##### 运行

运行得到下面的结果就行了：

![image-20210401162012106](http://cdn.wutang.cc/image-20210401162012106.png)

##### 登录

然后打开浏览器输入账号密码登录就行了

![image-20210401162137851](http://cdn.wutang.cc/image-20210401162137851.png)

##### 新建bucket 

![image-20210401162402061](http://cdn.wutang.cc/image-20210401162402061.png)

这时候你就有储存空间了，接下来就是写代码了

**开启访问权限,这样才能通过路径访问**

![image-20210401170103988](http://cdn.wutang.cc/image-20210401170103988.png)

![image-20210401170113557](http://cdn.wutang.cc/image-20210401170113557.png)

#### 搭建项目

##### 引入maven依赖

```xml
<!--        minio对象存储-->
        <dependency>
            <groupId>io.minio</groupId>
            <artifactId>minio</artifactId>
            <version>6.0.11</version>
        </dependency>
```



##### 编写配置yaml

```yaml
server:
  port: 8080
minio:
  config:
#    访问地址
    url: http://localhost:9000
#    账号
    access-key: TANG
#    密码
    secret-key: QWE123BNM
#    储存桶
    image-bucket: images
spring:
  servlet:
    multipart:
      max-file-size: 200MB
      max-request-size: 200MB


```

##### 自己的minioconfig 类

```java
@Configuration
@Data
public class MinIOConfig {
    @Value("${minio.config.url}")
    private String url;

    @Value("${minio.config.access-key}")
    private String accessKey;

    @Value("${minio.config.secret-key}")
    private String secretKey;

    @Value("${minio.config.image-bucket}")
    private String imageBucket;


}
```



##### util类

**额外的方法自己看下api继续写就是的了**

```java
@Component
public class MinIOUtils {
    @Autowired
    private MinIOConfig minIOConfig;

    private MinioClient minioClient;

    /**
    *初始化client
    */
    @PostConstruct
    public void init() {
        try {
            minioClient = new MinioClient(minIOConfig.getUrl(), minIOConfig.getAccessKey(), minIOConfig.getSecretKey());
        } catch (InvalidEndpointException e) {
            e.printStackTrace();
        } catch (InvalidPortException e) {
            e.printStackTrace();
        }
    }


	    /**
    *上传文件返回路径
    */
    public String upload(MultipartFile file) throws InvalidPortException, InvalidEndpointException, IOException, XmlPullParserException, NoSuchAlgorithmException, InvalidKeyException, InvalidArgumentException, InvalidResponseException, InternalException, NoResponseException, InvalidBucketNameException, InsufficientDataException, ErrorResponseException {
        InputStream inputStream = null;
        try {
            inputStream = file.getInputStream();
        } catch (IOException e) {
            e.printStackTrace();
        }

        String substring = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf("."), file.getOriginalFilename().length());
        String originalFilename = System.currentTimeMillis() + substring;
//        上传到minio

        minioClient.putObject(minIOConfig.getImageBucket(), originalFilename, inputStream, file.getSize(), null, null, file.getContentType());
        //这边的桶要开启访问权限才能通过这个路径访问
        String url = minioClient.getObjectUrl(minIOConfig.getImageBucket(), originalFilename);

        return url;
    }

    /**
    列出所有的bucket
    *
    */
    public List<String> listBuckets() throws InvalidPortException, InvalidEndpointException, IOException, InvalidKeyException, NoSuchAlgorithmException, InsufficientDataException, InvalidResponseException, InternalException, NoResponseException, InvalidBucketNameException, XmlPullParserException, ErrorResponseException {
        List<Bucket> buckets = minioClient.listBuckets();
        List<String> strings = new ArrayList<>();
        buckets.forEach(m -> {
            strings.add(m.name());
        });
        return strings;
    }

       /**
   	 根据文件名称删除
    *
    */
    public void remove(String str) throws InvalidPortException, InvalidEndpointException, IOException, InvalidKeyException, NoSuchAlgorithmException, InsufficientDataException, InvalidArgumentException, InvalidResponseException, InternalException, NoResponseException, InvalidBucketNameException, XmlPullParserException, ErrorResponseException {
        minioClient.removeObject(minIOConfig.getImageBucket(), str);
    }
}

```



api还是比较简单

![image-20210401162950729](http://cdn.wutang.cc/image-20210401162950729.png)



postman测试

![image-20210401170221837](http://cdn.wutang.cc/image-20210401170221837.png)

打开刚刚docker 启动映射的路径可以看到上传的图片

![image-20210401170305093](http://cdn.wutang.cc/image-20210401170305093.png)

##### api 详细文档[Java Client API参考文档 | Minio中文文档](http://docs.minio.org.cn/docs/master/java-client-api-reference)