## 配置

### 开通OSS服务

1. 登录[阿里云官网](https://www.aliyun.com/?spm=a2c4g.11186623.2.11.154c28bckD0xOk)。

2. 将鼠标移至**产品**，单击**对象存储 OSS**，打开 **OSS 产品详情**页面。

3. 在 [OSS 产品详情页](https://www.aliyun.com/product/oss?spm=a2c4g.11186623.2.12.154c28bckD0xOk)，单击**立即开通**。

4. 开通服务后，在 **OSS 产品详情**页单击**管理控制台**直接进入 OSS 管理控制台界面。

   您也可以单击位于官网首页右上方菜单栏的**控制台**，进入阿里云管理控制台首页，然后单击左侧的**对象存储 OSS** 菜单进入 OSS 管理控制台界面。

### 创建Bucket

Bucket实质就是阿里云OSS对象存储的一个存储空间，按照计算机理解的话可以理解为一个磁盘（不知道这样比喻是否恰当）。

1. 登录[OSS管理控制台](https://oss.console.aliyun.com/)。

2. 打开**创建Bucket**对话框。

   - 新版控制台：单击**Bucket列表**，之后单击**创建Bucket**。
   - 旧版控制台：在左侧Bucket列表中单击**+**号。

   您也可以单击**概览**，之后单击右侧的**创建Bucket**。

3. 在**创建Bucket**对话框配置Bucket参数。

### 创建AccessKey

####  AccessKey

AccessKey是访问阿里云API的秘钥，这里也需要提前创建一份，创建后我们需要记住自己的AccessKey ID和Access Key Secret

### SDK使用

## 导入依赖

```xml
        <dependency>
            <groupId>com.aliyun.oss</groupId>
            <artifactId>aliyun-sdk-oss</artifactId>
            <version>3.8.1</version>
        </dependency>
```

### 图片上传

**上传图片二进制流**

```java
    public static Object barcodeImgUpload(String msg){
        //创建上传Object的Metadata
        ObjectMetadata objectMetadata = new ObjectMetadata();
        objectMetadata.setContentType("image/jpg");
        String fileName=msg+".png";
        objectMetadata.setContentDisposition("inline;filename=" +fileName);//该设置可以使图片直接预览
        //生成图片二进制流
        byte[] content=generate(msg);
        //上传图片,第二个参数objectname=目录+文件名，相当于访问路径
        String fileDir="lsl/img/";//上传的OSS下的文件目录
        ossClient.putObject(AppConstant.BUCKET_NAME,fileDir+fileName,
                new ByteArrayInputStream(content),objectMetadata);
        // 设置URL过期时间为10年  3600l* 1000*24*365*10
        Date expiration = new Date(new Date().getTime() + 3600l * 1000 * 24 * 365 * 10);
        URL url=ossClient.generatePresignedUrl(AppConstant.BUCKET_NAME,fileDir+fileName,expiration);
        System.out.println(url.toString());
        return null;
    }
```

