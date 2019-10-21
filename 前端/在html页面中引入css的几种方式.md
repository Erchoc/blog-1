### css主要有三种引用方法：
#### 引用外部CSS文件
用<link>标签引用外部的css文件，将样式引用到html文档来。
```
<link rel="stylesheet" type="text/css" href="style.css">
```
#### 内部定义Style样式
在html文档中在<style>标签里面写的css样式
```
<style>
  bdoy{font-size:14px;} 
</style>
```
#### 内联样式
在标签中使用style属性将当前的标签样式改变。
```
<div style="color:red"></div>
```

### 优先级

内联样式 > 内部定义Style样式 > 引用外部CSS文件