**需求：**项目方需要上传的文件在网页上能够直接预览。

# Vue

直接把pdfjs解压到static目录下，然后新增一个预览函数

```javascript
export function Preview(url){
  if (url == "" || url == null) {
    Message.error("请刷新后重试");
    return;
  }
  url = encodeURIComponent("http://xxx/api/file/preview?fileAddress="+ url);
  window.open("/static/plugins/pdfjs/web/viewer.html?file="+url);
}
```

# Spring

## 预览接口

```java
    package com.xxx.controller;
	@GetMapping("preview")
    @ApiOperation(value="预览文件", notes = "pdf预览接口")
    public void preview(HttpServletRequest request, HttpServletResponse response,
                        @RequestParam(value = "fileAddress") String fileAddress){
        try{
            fileTool.previewFile(request, response, fileAddress);
        }catch(IOException | AllException e){
            log.error(Arrays.toString(e.getStackTrace()));
        }
    }
```

## 返回二进制流以提供预览

```java
    package com.xxx.tools;
	public void previewFile(HttpServletRequest request,
                            HttpServletResponse response,
                            String fileAddress) throws IOException, AllException{
        File file = new File(fileAddress);
        if (file.exists()) {
            byte[] data = null;
            FileInputStream input=null;
            input= new FileInputStream(file);
            data = new byte[input.available()];
            input.read(data);
            response.getOutputStream().write(data);
            if(input!=null) {
                input.close();
            }
        } else {
            throw new AllException(EmAllException.BAD_REQUEST, "请求文件不存在");
        }
    }
```

## 在Security Config中允许无认证访问

```java
package com.xxx.security.config;
.antMatchers("/file/preview/**").permitAll()
```

