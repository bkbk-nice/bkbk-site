---
slug: ftp
title: ftp文件上传
authors: [bkbk]
tags: [ftp]
---
  
:::caution 目录
**ftp** 
**上传后可使用nginx完成访问**        
::: -->
<!--truncate-->
 

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








