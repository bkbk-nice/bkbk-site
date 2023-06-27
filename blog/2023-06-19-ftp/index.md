---
slug: ftp
title: ftp API
authors: [bkbk]
tags: [ftp,API]
---
  


:::caution 目录
**ftp** 
**上传后可使用nginx完成访问**        
::: -->
<!--truncate-->
 


### ftp文件上传

``` bash title="查看是否安装"
vsftpd -v
```

``` bash title="安装(需要root)"
yum -y install vsftpd
```
 
``` bash title="创建用户"
useradd -d /home/ftpuser ftpuser
passwd ftpuser 
```
 
``` bash title="查看外部访问权限并开启"
getsebool -a | grep ftp
setsebool tftp_home_dir on
setsebool ftpd_full_access on
```

``` bash title="启动vsftp、设置开机启动"
systemctl start vsftpd
systemctl enable vsftpd
```

``` java title="java" {1-8} 
ftp:
  hostname: 192.168.xx
  port: 21
  encoding: utf-8
  username: ftpuser
  password: xx
  dir: /home/ftpuser/store
  accessUri: http://192.168.xx:8888/


@Value("${ftp.hostname}")
private String hostname;

@Value("${ftp.port}")
private Integer port;

@Value("${ftp.username}")
private String username;

@Value("${ftp.password}")
private String password;

@Value("${ftp.encoding}")
private String encoding;

@Value("${ftp.dir}")
private String dir;



@Value("${ftp.accessUri}")
private String accessUri;


@PostMapping("/singleImageUpload")
public ResultVo singleImageUpload(MultipartFile file){ 
    return  ResultVo.success(singleFile(file));
}



private String singleFile(MultipartFile file)   {
    FTPClient ftpClient = new FTPClient();
    ftpClient.setControlEncoding(encoding);
    String fileName;
    try { 

        ftpClient.connect(hostname,port);
        ftpClient.login(username,password);
        ftpClient.enterLocalPassiveMode();
        int replycode = ftpClient.getReplyCode();
        if (!FTPReply.isPositiveCompletion(replycode)) {
            System.out.println(" connect failed. . .ftp服务器:" + hostname + ":" + port);
        }

        ftpClient.setFileType(FTP.BINARY_FILE_TYPE);
        boolean boo = ftpClient.changeWorkingDirectory(dir);
        if(!boo) {
            System.out.println("进入文件夹失败");
        }

        fileName = UUID.randomUUID().toString() + file.getOriginalFilename().substring(
                file.getOriginalFilename().lastIndexOf(".")); 

        ftpClient.storeFile(fileName,file.getInputStream()); 
        ftpClient.logout();
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
    return accessUri +  fileName; 
}
```

### 第三方API上传
上传图片并返回url,根据response返回值做条件判断
[smms.app](http://smms.app)

``` java title="service" {1}
//主要方法   String re = doPostFormData(file,id);
@Service
public class ClientServiceImpl implements ClientService {  
    @Autowired
    private ClientMapper clientMapper;
    @Override
    public ResultVo uploadAvatar(MultipartFile file,  Integer id)   { 
        if (null == file && !file.isEmpty()) {  return ResultVo.fail("文件为空");  }
        if (null == id  ) {  return ResultVo.fail("id为空");  }
        String re = doPostFormData(file,id);
        if(re.equals("异常")){  return ResultVo.fail(re);  } 
        return ResultVo.success(re);
    }
}
```

``` java title="doPostFormData" {1}
OkHttpClient发送http请求，获取结果并导入数据库
public   String doPostFormData( MultipartFile file ,Integer id) { 
    try {
        OkHttpClient client = new OkHttpClient().newBuilder()
                .build();
        MediaType mediaType = MediaType.parse("multipart/form-data; boundary=--------------------------500812903788481473868931");
        String newFileName = UUID.randomUUID().toString()+file.getOriginalFilename();
        File tempfile =  multipartFileToFile(file,newFileName);

        RequestBody body = new MultipartBody.Builder().setType(MultipartBody.FORM)
                .addFormDataPart("smfile", newFileName,
                        RequestBody.create(MediaType.parse("application/octet-stream"),
                                tempfile  ))
                .build();
        Request request = new Request.Builder()
                .url("https://smms.app/api/v2/upload")
                .method("POST", body)
                .addHeader("Authorization", "ZvaUPBNxxxxxxxxxxxxx4soUj7zUNb")
                .addHeader("User-Agent", "Apifox/1.0.0 (https://apifox.com)")
                .addHeader("Accept", "*/*")
                .addHeader("Host", "smms.app")
                .addHeader("Connection", "keep-alive")
                .addHeader("Content-Type", "multipart/form-data; boundary=--------------------------500812903788481473868931")
                .addHeader("Cookie", "PHPSESSID=5babcaa3-e606-bf37-812b-5e8b880a0cad; smms=rIuamMGvf7qF2K8NHiLOsPQSodVzb0Y9")
                .build();
        Response  response = client.newCall(request).execute();
        String res = response.body().string(); 
        tempfile.delete(); 
        ObjectMapper mapper =new ObjectMapper();
        JsonNode node = mapper.readTree(res);// 这里的JsonNode和XML里面的Node很像  
        if( node.get("success").toString().equals("true") ){
            node = node.get("data");
            String imageurl = node.get("url").toString();
            String imagedelete = node.get("delete").toString();
            
            //导入数据库
            imageurl = imageurl.trim().replace("\"", "");
            imagedelete = imagedelete.trim().replace("\"", "");  
            UpdateWrapper updateWrapper = new UpdateWrapper();
            updateWrapper.eq("id", id);
            updateWrapper.set("image_url",imageurl);
            updateWrapper.set("image_delete",imagedelete); 
            clientMapper.update(null, updateWrapper); 
            return  imageurl; 
        }else if(node.get("message").toString().contains("Image upload repeated limit")){
            System.out.println("已存在");
            String imageurl = node.get("images").toString();
            imageurl = imageurl.trim().replace("\"", ""); 
            UpdateWrapper updateWrapper = new UpdateWrapper();
            updateWrapper.eq("id", id);
            updateWrapper.set("image_url",imageurl); 
            clientMapper.update(null, updateWrapper); 
            return imageurl;
        }else{ 
            return "异常";
        } 
    } catch (IOException e) {
        throw new RuntimeException(e);
    } catch (Exception e) {
        throw new RuntimeException(e);
    } 
} 
```






