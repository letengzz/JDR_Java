# 文件上传

文件上传要求form表单的请求方式必须为post，并且添加属性`enctype="multipart/form-data"`

SpringBoot中将上传的文件封装到MultipartFile对象中，通过此对象可以获取文件相关信息

> index.html

```html
<form th:action="@{/upload}" method="post" enctype="multipart/form-data">
    <!-- 单文件上传 -->
    <input type="file" name="file"><br>
    <!-- 多文件上传 -->
    <input type="file" name="photos" multiple><br>
    <input type="submit" value="提交">
</form>
```

> FileController.java

```java
@Slf4j
@Controller
public class FileController {

    @PostMapping("/upload")
    public String upload(HttpSession session, @RequestPart("file")MultipartFile file,
                         @RequestPart("photos")MultipartFile[] photos) throws IOException {
        log.info("上传的信息:file={},photos={}",file.getSize(),photos.length);
        if (!file.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = file.getOriginalFilename();
            //获取上传的文件的后缀名
            assert originalFilename != null;
            String suffixName = originalFilename.substring(originalFilename.lastIndexOf("."));
            //使用UUID防止重名 将UUID作为文件名
            String uuid = UUID.randomUUID().toString().replaceAll("-", "");
            //将uuid和后缀名拼接后的结果作为最终的文件名
            String fileName = uuid + suffixName;
            ServletContext servletContext = session.getServletContext();
            String path = servletContext.getRealPath("/");
            File filePath = new File(path,"img\\");
            if (!filePath.exists()){
                filePath.mkdirs();
            }
            file.transferTo(new File(filePath+fileName));
        }
        //上传多个文件
        if (photos.length>0){
            for (MultipartFile photo :
                    photos) {
                String originalFilename = photo.getOriginalFilename();
                //获取上传的文件的后缀名
                assert originalFilename != null;
                String suffixName = originalFilename.substring(originalFilename.lastIndexOf("."));
                //使用UUID防止重名 将UUID作为文件名
                String uuid = UUID.randomUUID().toString().replaceAll("-", "");
                //将uuid和后缀名拼接后的结果作为最终的文件名
                String fileName = uuid + suffixName;

                ServletContext servletContext = session.getServletContext();
                String path = servletContext.getRealPath("/");
                File filePath = new File(path,"img\\");
                if (!filePath.exists()){
                    filePath.mkdirs();
                }
                System.out.println(filePath+fileName);
                photo.transferTo(new File(filePath,fileName));
            }
        }
        return "index";
    }
}
```