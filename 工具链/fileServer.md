# 附件管理服务

附件，即附加在信息主体上的文件，在信息服务中本身并无业务含义。

concrete在逻辑结构上将附件管理和服务模块分离开来，针对附件，附件管理是Server，服务模块是Client

concrete提供了服务端、客户端、存储的参考实现

## 服务端

基于jaxrs2.0; apache commons-fileupload的实现
    
    上传: POST /attachments/upload/byform/{clientId}/{tokenId}
    下载：GET /attachments/download/{attachmentId};c=clientId;t=tokenId

## 客户端

客户端附件拦截器
cc.coodex.concrete.attachments.client.AttachmentInterceptor

返回的Pojo中，包含@Attachment注解的String域可以被自动许可

## 存储
 
基于fastDFS实现

## attachmentService.properties

公用

    key._clientId_ = 

服务端

    download.speedLimited = n (k), default 1024
    rule.read = check | public | default public
    _clientId_.location = 
    _clientId_.readonly = true or false | default true


服务端参考实现

    # 下载任务优先级
    download.priority = 1 to 10 | default 1
    upload.priority = 1 to 10 | default 5



客户端

    server.location = 
    
    # 附件授权有效期。单位:分钟；默认10分钟
    attachment.validity = 

附件仓库参考实现: fastdfs repository

    fastdfs.tracker = trackerServer:port; ....
