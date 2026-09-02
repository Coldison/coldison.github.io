---
title: ffmpeg使用(未完)
date: 2022-07-18 22:20:07
tags:
- ffmpeg
- 视频
categories:
- 教程
- 视频编辑
---
一直想建立一个视频搬运的工作流，比如在b站上看到什么视频了，复制下链接，然后自动抓取转码和上传到推特或者油管，推特机器人账号需要自己有服务器，所以再议。
本文主要是收集一些ffmpeg的使用，视频剪辑软件对于我的需求来说太重了。比如我可能需要的是剪切，转码，音频提取（接翻译接口），字幕生成之类的，这些事情脚本做还是比较方便的。
然后因为还是参考别人的博客也没有自己读文档，先在这里记录一下地址。

### 转码

---

<div id="article_content" class="article_content clearfix">
        <link rel="stylesheet" href="https://csdnimg.cn/release/blogv2/dist/mdeditor/css/editerView/ck_htmledit_views-bbac9290cd.css">
                <div id="content_views" class="htmledit_views">
                    <h1><a name="t0"></a>1. 准备</h1> 
<h3><a name="t1"></a>1.1 下载<a href="https://so.csdn.net/so/search?q=ffmpeg&spm=1001.2101.3001.7020" target="_blank" class="hl hl-1" data-report-click="{"spm":"1001.2101.3001.7020","dest":"https://so.csdn.net/so/search?q=ffmpeg&spm=1001.2101.3001.7020"}" data-tit="ffmpeg" data-pretit="ffmpeg">ffmpeg</a></h3> 
<p>进入ffmpeg官网<a href="https://www.ffmpeg.org/download.html" title="Download FFmpeg">Download FFmpeg</a>，根据自己的系统下载相应封装，这里以Windows为例。</p> 
<p style="text-align:center;"><img alt="" src="https://img-blog.csdnimg.cn/8cae5f53974c47e1a5dbe7f09e85c638.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6IuP5pmTa8SBb-mUruebmA==,size_20,color_FFFFFF,t_70,g_se,x_16"></p> 
<p>选择篮框中任意一项进行下载。</p> 
<p style="text-align:center;">以下是选择第一项后的截图<img alt="" src="https://img-blog.csdnimg.cn/bc35a21c4c3d4d3c9c15ec70674ca02d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6IuP5pmTa8SBb-mUruebmA==,size_20,color_FFFFFF,t_70,g_se,x_16"> </p> 
<p>下载合适的release，解压，将bin目录下的exe文件全部复制到目录<span style="color:#38d8f0;">C:\Windows\System32</span>下。</p> 
<h3><a name="t2"></a>1.2 cmd基础</h3> 
<p><strong>1.2.1 打开cmd</strong></p> 
<p>通过win+R，或 右键“开始”选择“运行”可进入cmd。</p> 
<p><strong>1.2.2 进入指定文件夹</strong></p> 
<p>①进入某个磁盘，直接盘符代号：如D：，然后回车，到D盘下（不用CD 命令切换）</p> 
<p>②输入dir，可以看到d盘下的各个文件名称</p> 
<p>③进入除根录以外的文件夹 ：  cd  文件夹路径（cd  xxx\xxx\xxx）回车</p> 
<p>④进入上一层目录 ： cd ../</p> 
<p>⑤返回D盘：cd\  </p> 
<p>⑥返回C盘：直接输入c: ，回车</p> 
<p>注： 不能在一打开cmd的时候运行cd  d:\xxx\xxx，需要先进入磁盘</p> 
<p>以进入<span style="color:#38d8f0;">E:\Videos\S</span>为例。</p> 
<p>在cmd中输入磁盘符<span style="color:#38d8f0;">E：</span><span style="color:#0d0016;">，回车。这一步不用cd命令。</span></p> 
<p><span style="color:#0d0016;">然后输入</span><span style="color:#38d8f0;">cd Videos\S</span><span style="color:#0d0016;">，回车即可。</span></p> 
<h1><a name="t3"></a>2. 文件转换</h1> 
<h3><a name="t4"></a>2.1 单个文件</h3> 
<pre data-index="0"><code class="hljs language-sql">ffmpeg <span class="hljs-operator">-</span>i "input.flv" <span class="hljs-operator">-</span>c <span class="hljs-keyword">copy</span> "output.mp4"</code><div class="hljs-button {2}" data-title="复制" onclick="hljs.copyCode(event)"></div></pre> 
<p>将这里的input改为你的文件名，output改为你想得到的文件名即可。</p> 
<h3><a name="t5"></a>2.2 批量转换</h3> 
<pre data-index="1"><code class="hljs language-perl"><span class="hljs-keyword">for</span> %i in (*.flv) <span class="hljs-keyword">do</span> ffmpeg -i <span class="hljs-string">"%i"</span> -c copy <span class="hljs-string">"%~ni.mp4"</span></code><div class="hljs-button {2}" data-title="复制" onclick="hljs.copyCode(event)"></div></pre> 
<p>这时新生成的mp4文件会沿用原文件名。</p> 
<h3><a name="t6"></a>2.3 某些flv文件转换成mp4时会报错，这时可尝试以下代码：</h3> 
<pre data-index="2"><code class="hljs language-r">ffmpeg <span class="hljs-operator">-</span>i filename.flv <span class="hljs-operator">-</span><span class="hljs-built_in">c</span><span class="hljs-operator">:</span>v libx264 <span class="hljs-operator">-</span>crf <span class="hljs-number">19</span> <span class="hljs-operator">-</span>strict experimental filename.mp4</code><div class="hljs-button {2}" data-title="复制" onclick="hljs.copyCode(event)"></div></pre> 
<p>第一个filename改为需要转换的文件名，第二个filename改为相应的输出文件名。</p> 
<h3><a name="t7"></a>*2.4 flv/mp4文件的合并</h3> 
<p>有时通过某些下载工具得到的flv/mp4文件被分为多个片段，但我们希望将它们合并。</p> 
<p>以合并5个mp4文件：</p> 
<p>文件1.mp4<br> 文件2.mp4<br> 文件3.mp4<br> 文件4.mp4<br> 文件5.mp4</p> 
<p>为例。</p> 
<p>新建一个txt文件，把需要合并的mp4文件的文件名依序写在txt文件中并保存，格式如下：</p> 
<pre data-index="3"><code class="hljs language-javascript"><ol class="hljs-ln" style="width:100%"><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="1"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">file <span class="hljs-string">'文件1.mp4'</span></div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="2"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">file <span class="hljs-string">'文件2.mp4'</span></div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="3"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">file <span class="hljs-string">'文件3.mp4'</span></div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="4"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">file <span class="hljs-string">'文件4.mp4'</span></div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="5"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">file <span class="hljs-string">'文件5.mp4'</span></div></div></li></ol></code><div class="hljs-button {2}" data-title="复制" onclick="hljs.copyCode(event)"></div></pre> 
<p>注：这里txt文件被命名为<span style="color:#38d8f0;"><strong>combine.txt</strong></span></p> 
<p>把上述需要合并的mp4文件和这个txt文件放在同一个文件夹下，然后在cmd中进入该文件夹，再输入以下命令：</p> 
<pre data-index="4"><code class="hljs language-lua">ffmpeg -f <span class="hljs-built_in">concat</span> -safe <span class="hljs-number">0</span> -i combine.txt -c copy <span class="hljs-built_in">output</span>.mp4</code><div class="hljs-button {2}" data-title="复制" onclick="hljs.copyCode(event)"></div></pre> 
<p>回车。即可得到一个完整的mp4文件。</p> 
<p>合并多个flv文件的方法类似。</p> 
<p></p> 
<p>输出的文件与原文件在同一文件夹中。</p>
                </div><div><div></div></div>
        </div>

---
