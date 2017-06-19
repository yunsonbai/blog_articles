---
title: 富文本实践
date: 2016-06-08 13:50:05
tags:
    - html
    - 富文本
    - summernote
categories: html
---

## 引言
``` bash
    前段时间趁着项目不忙自己玩了一下富文本，在html页实现富文本。
```
<!--more-->
### 使用的js包
``` bash
summernote
```
### html代码
``` bash
     <!DOCTYPE html>
        <html>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <head>
        <link href="../bootstrap-3.3.5/css/bootstrap.css" rel="stylesheet">
        <link href="font-awesome-4.5.0/css/font-awesome.css" rel="stylesheet">
        <link href="summernote/dist/summernote.css" rel="stylesheet">
        <script src="../jquery-2.1.4.js"></script>
        <script src="../bootstrap-3.3.5/js/bootstrap.js"></script>
        <script src="summernote/dist/summernote.js"></script>
        <script src="mysummer.js"></script>
        <script src="mysummer.css"></script>
        </head>
        <body>
          <div id="summernote">Hello Summernote</div>
          <span onclick='getContext(this)'>预览</span>
        
          <div id="context"></div>
         
        </body>
        </html>
```

### mysummer.js代码
``` bash
    $(document).ready(function() {
      $('#summernote').summernote({
            toolbar: [
                ['style', ['style','fontname', 'bold', 'italic', 'underline','fontsize','color']],
                ['font', ['strikethrough', 'superscript', 'subscript']],
                ['para', ['ul', 'ol', 'paragraph']],
                ['insert', ['link', 'picture', 'table', 'hr']],
                ['height', ['height']],
                ['misc', ['redo','undo','help', 'fullscreen']],
                // ['view', ['codeview']],
          ]
      });
    });
    
    
    function getContext(obj){
         $(document).ready(function(){
              var code = $('#summernote').summernote('code');
              var context = document.getElementById('context');
              context.innerHTML=code;
    
              var data = {"code":$('#summernote').summernote('code')};
            $.ajax({
                url: '/whatdo/test',
                type: 'POST',
                data: data,
                dataType: 'json',
                success: function(data,status){
                     alert(data.status)
                },
            });
    
         })
    }

```

### mysummer.css代码
``` bash
       $('#summernote').summernote({
        /*  height: 700,*/
          minHeight: null,
          maxHeight: null,
          focus: true
             });
        
```

### 效果图
{% img  https://yunsonbai.github.io/images/article_img/summernote.jpg %}
