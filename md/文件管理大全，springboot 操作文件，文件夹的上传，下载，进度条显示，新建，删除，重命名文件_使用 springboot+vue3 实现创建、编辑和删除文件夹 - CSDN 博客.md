> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_38362127/article/details/104991620)

#### 文件管理大全，springboot 操作文件，文件夹的上传，下载，[进度条](https://so.csdn.net/so/search?q=%E8%BF%9B%E5%BA%A6%E6%9D%A1&spm=1001.2101.3001.7020)显示，新建，删除，重命名文件

*   *   [说明](#_2)
    *   [1. 多文件上传](#1_11)
    *   [2. 文件夹上传](#2__73)
    *   [3. 文件上传的进度条实现](#3_140)
    *   [4. 根据根目录的路径，递归获取改目录下的所有目录](#4_306)
    *   [5. 根据目录的路径，获取改目录下的所有文件信息详情](#5_324)
    *   [6. 创建文件夹，名字重复就在后面加 name(1)，name（2）采用 windows 文件重命名方法](#6name1name2windows_401)
    *   [7. 修改文件夹名称，也支持修改文件名](#7_437)
    *   [8. 递归删除文件夹及其子文件](#8_475)
    *   [9. 文件下载](#9_495)
    *   [10.githup 完整代码下载](#10githup_552)
    *   [11. 结束语](#11_555)

### 说明

```
 业务所需，使用web页面管理服务器上的文件，支持指定目录，左侧显示目录树，右侧显示当前目录下的所有文件，文件包含文件名，大小，创建时间。 
 开发工具：idea2019 springboot版本：2.0以上 
 前端：vue,ElementUI

```

**页面展示**  
![](https://img-blog.csdnimg.cn/20200320155049112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MzYyMTI3,size_16,color_FFFFFF,t_70)

### 1. 多文件上传

Controller 层，注意参数，传递类型 enctype=“multipart/form-data”  
/**

*   上传多个文件

*   @param request 请求信息
*   @param filePath 上传文件的路径
*   @return  
    */  
    @RequestMapping(value = “/uploadMuilt”, method = RequestMethod.POST)  
    public Object uploadMuilt(HttpServletRequest request, String filePath) {  
    Map<String, Object> dataMap = new HashMap<>();  
    try {  
    String str = fileService.uploadMuilt(request, filePath);  
    dataMap.put(“data”, str);  
    dataMap.put(“code”, 200);  
    dataMap.put(“msg”, “多文件上传成功”);  
    } catch (Exception e) {  
    e.printStackTrace();  
    dataMap.put(“data”, “”);  
    dataMap.put(“code”, 500);  
    dataMap.put(“msg”, “多文件上传失败”);  
    }  
    return dataMap;  
    }

```
service实现层
/**
 * 多文件上传
 * @param request
 * @param filePath
 * @return
 */
public String uploadMuilt(HttpServletRequest request,String filePath) {
    List<MultipartFile> files = ((MultipartHttpServletRequest)request).getFiles("file");
    MultipartFile file = null;
    BufferedOutputStream stream = null;
    for (int i = 0; i <files.size() ; i++) {
        file = files.get(i);
        filePath = filePath+"/";
        if(!file.isEmpty()){
            try{
                byte[] bytes = file.getBytes();
                stream = new BufferedOutputStream(new FileOutputStream(
                        new File(filePath+file.getOriginalFilename())
                ));
                stream.write(bytes);
                stream.close();
            }catch(Exception e){
                stream = null;
                e.printStackTrace();
                return "the"+i+"file upload failure";
            }
        }else {
            return "the"+i+"file is empty";
        }
    }
    return "upload multifile success";
}

```

### 2. 文件夹上传

```
/**
 * 上传文件夹
 *
 * @param file     待上传的文件夹
 * @param filePath 要上传到的目录路径
 * @return
 */
@RequestMapping(value = "/uploadFolder", method = RequestMethod.POST)
public Object uploadFolder(MultipartFile[] file, String filePath) {
    Map<String, Object> dataMap = new HashMap<>();
    try {
        FileUtil.saveMultiFile(filePath, file);
        // FileUtil.uploadTest(filePath, file);
        dataMap.put("data", "");
        dataMap.put("code", 200);
        dataMap.put("msg", "文件夹上传成功");
    } catch (Exception e) {
        dataMap.put("data", "");
        dataMap.put("code", 500);
        dataMap.put("msg", "文件夹上传失败");
        e.printStackTrace();
    }

    return dataMap;
}

```

```
/**
 * 在basePath下保存上传的文件夹
 *
 * @param basePath
 * @param files
 */
public static void saveMultiFile(String basePath, MultipartFile[] files) {
    if (files == null || files.length == 0) {
        return;
    }
    if (basePath.endsWith("/")) {
        basePath = basePath.substring(0, basePath.length() - 1);
    }
    for (MultipartFile file : files) {
        String filename = "";
       /* String filename = file.getOriginalFilename();
        String filePath = basePath + "/" + file.getOriginalFilename();*/
        CommonsMultipartFile cf= (CommonsMultipartFile)file;
        FileItem fileItem = cf.getFileItem();
        String fileStr = fileItem.toString();
        String[] split = fileStr.split(",");
        for (int i = 0; i <split.length ; i++) {
            if(split[i].contains("name=")){
                 filename = split[i].substring(5,split[i].length());
            }
        }
        String filePath = basePath + "/" + filename;
        makeDir(filePath);
        File dest = new File(filePath);
        try {
            file.transferTo(dest);
        } catch (IllegalStateException | IOException e) {
            e.printStackTrace();
        }
    }
}

```

### 3. 文件上传的进度条实现

**在启动类上加上注解，使用自定义的文件上传解析**

```
package com.filemanager.fm;

import com.filemanager.fm.config.CustomMultipartResolver;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.context.annotation.Bean;
import org.springframework.web.multipart.MultipartResolver;

import java.io.IOException;

//注意取消自动Multipart配置，否则可能在上传接口中拿不到file的值
@SpringBootApplication
@EnableAutoConfiguration(exclude = { MultipartAutoConfiguration.class })
public class FmApplication extends SpringBootServletInitializer{

    //注入自定义的文件上传处理类
    @Bean(name = "multipartResolver")
    public MultipartResolver multipartResolver() {
        CustomMultipartResolver customMultipartResolver = new CustomMultipartResolver();
        return customMultipartResolver;
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(FmApplication.class);
    }

    public static void main(String[] args) throws IOException {
        SpringApplication.run(FmApplication.class, args);
        
    }

}


```

**编写自定义文件渲染的配置**

```
package com.filemanager.fm.config;

import com.filemanager.fm.listener.UploadProgressListener;
import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.FileUpload;
import org.apache.commons.fileupload.FileUploadException;
import org.apache.commons.fileupload.servlet.ServletFileUpload;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartException;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Component
public
class CustomMultipartResolver extends CommonsMultipartResolver {

    @Autowired
    private UploadProgressListener progressListener;

    @Override
    protected MultipartParsingResult parseRequest(HttpServletRequest request) throws MultipartException {
        String encoding = super.determineEncoding(request);
        progressListener.setSession(request.getSession());
        FileUpload fileUpload = prepareFileUpload(encoding);
        fileUpload.setProgressListener(progressListener);
        try {
            List<FileItem> fileItemList =  ((ServletFileUpload)fileUpload).parseRequest(request);
            return super.parseFileItems(fileItemList, encoding);
        } catch (FileUploadException e) {
            e.printStackTrace();
        }
        return null;
    }
}


```

**编写监听器, 需实现 ProgressListener 接口**

```
package com.filemanager.fm.listener;


import com.filemanager.fm.model.Progress;
import org.apache.commons.fileupload.ProgressListener;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpSession;

@Component
public class UploadProgressListener implements ProgressListener {

    private HttpSession session;

    public void setSession(HttpSession session){
        this.session = session;
        Progress pe = new Progress();
        session.setAttribute("uploadStatus", pe);
    }


    /**
     *  设置当前已读取文件的进度
     * @param readBytes 已读的文件大小（单位byte）
     * @param allBytes 文件总大小（单位byte）
     * @param index 正在读取的文件序列
     */
    @Override
    public void update(long readBytes, long allBytes, int index) {
        Progress pe = (Progress) session.getAttribute("uploadStatus");
        pe.setReadBytes(readBytes);
        pe.setAllBytes(allBytes);
        pe.setCurrentIndex(index);
    }
}


```

**编写进度条实体类**

```
package com.filemanager.fm.model;

public class Progress {
    private long ReadBytes; //已读取文件的比特数
    private long contentLength;//文件总比特数
    private long items; //正读的第几个文件

    public long getReadBytes() {
        return ReadBytes;
    }

    public void setReadBytes(long readBytes) {
        ReadBytes = readBytes;
    }

    public long getContentLength() {
        return contentLength;
    }

    public void setContentLength(long contentLength) {
        this.contentLength = contentLength;
    }

    public long getItems() {
        return items;
    }

    public void setItems(long items) {
        this.items = items;
    }

    public void setAllBytes(long allBytes) {
    }

    public void setCurrentIndex(int index) {
    }
}


```

### 4. 根据[根目录](https://so.csdn.net/so/search?q=%E6%A0%B9%E7%9B%AE%E5%BD%95&spm=1001.2101.3001.7020)的路径，递归获取改目录下的所有目录

```
/**
 * 获取路径下的所有文件/文件夹
 *
 * @return
 */
public  List<String> getAllDirection(String filePath) {
    if(filePath == null || "".equals(filePath)){
        filePath = dirPath;
    }
    List<String> dirList = getAllDirs(filePath, true);
    return dirList;
}

```

### 5. 根据目录的路径，获取改目录下的所有文件信息详情

```
/**
 * 获取路径下的所有文件
 *
 * @param directoryPath 需要遍历的文件夹路径
 * @return
 */
public  List<Object> getAllFiles(String directoryPath) {
    List<Object> list = new ArrayList<Object>();
    File baseFile = new File(directoryPath);
    if (baseFile.isFile() || !baseFile.exists()) {
        return list;
    }
    File[] files = baseFile.listFiles();
    for (File file : files) {
        Map<String, String> map = new HashMap<>();
        if (file.isFile()) {
            map.put("fileName", file.getName());
            map.put("filePath", file.getAbsolutePath());
            map.put("time", getFileTime(file.getAbsolutePath()));
            map.put("size", getFileSize(file.getAbsolutePath()));
            list.add(map);
        }
    }
    return list;
}

```

**根据文件路径获取该文件的创建时间，即是文件的最后一次修改时间**

```
/**
 * 获取文件最后修改时间
 */
public  String getFileTime(String filePath) {
    File file = new File(filePath);
    long time = file.lastModified();//返回文件最后修改时间，是以个long型毫秒数
    String ctime = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss").format(new Date(time));
    return ctime;
}

```

**把文件的大小转成 KB，MB，GB 等字符串**

```
/**
 * 获取文件大小,整数
 */
public  String getFileSize(String path) {
    File file = new File(path);
    long length = file.length();
    return FormetFileSize(length);
}

/**
 * 转换文件大小，带单位的字符串
 *
 * @param fileLength 文件大小值
 * @return String 文件大小
 * @Title: FormetFileSize
 * @author projectNo
 */
public  String FormetFileSize(long fileLength) {
    DecimalFormat df = new DecimalFormat("#.00");
    DecimalFormat d = new DecimalFormat("#");
    String fileSizeString = "";
    if (fileLength < 1024) {
        fileSizeString = d.format((double) fileLength) + "B";
    } else if (fileLength < 1048576) {
        fileSizeString = df.format((double) fileLength / 1024) + "KB";
    } else if (fileLength < 1073741824) {
        fileSizeString = df.format((double) fileLength / 1048576) + "MB";
    } else {
        fileSizeString = df.format((double) fileLength / 1073741824) + "GB";
    }
    return fileSizeString;
}

```

### 6. 创建文件夹，名字重复就在后面加 name(1)，name（2）采用 windows 文件重命名方法

```
/**
 * 创建多级目录
 *
 * @param path
 */
public static boolean makeDirs(String path) {
    File file = new File(path);
    boolean flag = false;
    try {
        // 如果文件夹不存在则创建
        if (!file.exists() && !file.isDirectory()) {
            file.mkdirs();
            flag = true;
        }else{
            int i = 1;
            String name = path.substring(path.lastIndexOf("/")+1,path.length());
            while(file.exists()) {
                String newFilename = name+"("+i+")";
                String parentPath = file.getParent();
                file = new File(parentPath+ File.separator+newFilename);
                i++;

            }
            file.mkdirs();
            flag = true;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return flag;
}

```

### 7. 修改文件夹名称，也支持修改文件名

```
/**
 * 通过文件路径直接修改文件名,支持文件，目录
 *
 * @param filePath    需要修改的文件的完整路径
 * @param newFileName 需要修改的文件的名称
 * @return
 */
public String FixFileName(String filePath, String newFileName) {
    File f = new File(filePath);
    if (!f.exists()) { // 判断原文件是否存在（防止文件名冲突）
        return null;
    }
    newFileName = newFileName.trim();
    if ("".equals(newFileName) || newFileName == null) // 文件名不能为空
        return null;
    String newFilePath = null;
    if (f.isDirectory()) { // 判断是否为文件夹
        int i = filePath.lastIndexOf("\\");
        //windows系统使用\\分割，linux使用/分割
        newFilePath = filePath.substring(0, filePath.lastIndexOf("/")) + "/" + newFileName;
    } else {
        newFilePath = filePath.substring(0, filePath.lastIndexOf("/")) + "/" + newFileName
                + filePath.substring(filePath.lastIndexOf("."));
    }
    File nf = new File(newFilePath);
    try {
        f.renameTo(nf); // 修改文件名
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
    return newFilePath;
}

```

### 8. 递归删除文件夹及其子文件

```
/**
 * 删除目录及其子文件夹
 *
 * @param rootFilePath 根目录
 * @return boolean
 */
public boolean deleteFile(File rootFilePath) {
    if (rootFilePath.isDirectory()) {
        for (File file : rootFilePath.listFiles()) {
            deleteFile(file);
        }
    }
    return rootFilePath.delete();
}

```

### 9. 文件下载

```
/**
 * 文件下载
 *
 * @param response
 * @param filePath 被下载的文件在服务器中的路径
 * @return
 * @throws UnsupportedEncodingException
 */
@RequestMapping(value = "/download", method = RequestMethod.POST)
private String download(HttpServletResponse response, String filePath) throws UnsupportedEncodingException {
    File file = new File(filePath);
    String fileName = filePath.substring(filePath.lastIndexOf("/") + 1);//被下载文件的名称
    if (file.exists()) {
        response.setContentType("application/force-download");// 设置强制下载不打开
        response.setHeader("Content-Disposition", "attachment;file
                + URLEncoder.encode(fileName, "UTF-8"));

        byte[] buffer = new byte[1024];
        FileInputStream fis = null;
        BufferedInputStream bis = null;
        try {
            fis = new FileInputStream(file);
            bis = new BufferedInputStream(fis);
            OutputStream outputStream = response.getOutputStream();
            int i = bis.read(buffer);
            while (i != -1) {
                outputStream.write(buffer, 0, i);
                i = bis.read(buffer);
            }

            return "下载成功";
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (bis != null) {
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    return "下载失败";
}

```

### 10.githup 完整代码下载

[https://github.com/00-xiaoYu/FileManager](https://github.com/00-xiaoYu/FileManager)

### 11. 结束语

**支持开源，少搬砖头**