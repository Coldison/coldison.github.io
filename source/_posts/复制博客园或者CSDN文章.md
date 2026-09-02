---
title: 复制博客园或者CSDN文章
date: 2022-07-19 01:13:43
tags:
- 网页
- 博客
- blog
categories:
- 教程
- 博客设置
---
在转载博客的时候，出于对于站点的不信任，还是倾向于将对应的博文保存到本地。

参考了如下方法，可以直接提取CSDN或者博客园中的博文内容，并将html直接粘贴到markdown中，实现复制博文的效果，其他站点应该也类似，不过显然图床的问题也并没有解决，会随时挂掉。

以下内容为直接粘贴html元素：

---



<div id="article_content" class="article_content clearfix">
        <link rel="stylesheet" href="https://csdnimg.cn/release/blogv2/dist/mdeditor/css/editerView/ck_htmledit_views-bbac9290cd.css">
                <div id="content_views" class="markdown_views prism-atom-one-dark">
                    <svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
                        <path stroke-linecap="round" d="M5,0 0,2.5 5,5z" id="raphael-marker-block" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path>
                    </svg>
                    <p><strong>复制csdn或者博客园文章时，图片无法直接粘贴过来解决办法。</strong></p> 
<p>1、csdn 文章页面，打开浏览器开发者工具</p> 
<p>2、找到文章正文对应的 html 元素，按ctrl+f输入标签头关键字 （含 "article_content"标签头(csdn文章) ，如果是博客园文章，则标签头是“cnblogs_post_body”）</p> 
<p><img src="https://img-blog.csdnimg.cn/img_convert/264838c821805a1acf1fdd5cc54e7268.png" alt="img"></p> 
<p>3、在该元素源代码上右键 “Copy”->“Copy element”</p> 
<p><img src="https://img-blog.csdnimg.cn/img_convert/bd91c1b2f1b0994e12545d15ad1d7dc9.png" alt="img"></p> 
<p>4、新建一个 txt 文件，将后缀改为 .html ，把刚复制的 源代码 粘贴到文件中，浏览器打开，此时复制全文，到博客园 添加新随笔，粘贴。<br> 5、或者复制全文到markdown，到cadn 添加新随笔导入markdown。</p> 
<blockquote> 
 <p>解决方法参考：https://jingyan.baidu.com/article/0964eca24e159c8285f53618.html</p> 
</blockquote>
                </div><div><div></div></div>
                <link href="https://csdnimg.cn/release/blogv2/dist/mdeditor/css/editerView/markdown_views-3fd7f7a902.css" rel="stylesheet">
                <link href="https://csdnimg.cn/release/blogv2/dist/mdeditor/css/style-49037e4d27.css" rel="stylesheet">
        </div>
