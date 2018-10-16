# JavaScript-学习
1、脚本位置
因为当浏览器解析到 <script> 标签时，浏览器会停止解析其后的内容，而优先下载脚本文件，并执行其中的代码，
这意味着，其后的 styles.css 样式文件和<body>标签都无法被加载，由于<body>标签无法被加载，导致页面无法渲染，是一片空白。
JavaScript下载过程会阻塞其他资源的下载，比如样式文件和图片。尽管脚本的下载过程不会互相影响，但页面仍然必须等待所有
JavaScript代码下载并执行完成才能继续。
  
推荐：将所有<script>标签尽可能放到<body>标签的底部，以尽量减少对整个页面下载的影响。（优化JavaScript 的首要规则：将脚本放在底部。）

2、组织脚本
因为每遇到一个<script>标签，都会因执行脚本而导致一定的延时，所以减少页面包含的<script>标签数量有助于改善这一情况。这不仅针对外链脚本，内嵌脚本的数量同样也要限制。
  
特别提醒的是，把一段内嵌脚本放在引用外链样式表的<link>之后会导致页面阻塞去等待样式表的下载。这样做是为了确保内嵌脚本在执行时能获得最精确的样式信息。因此，建议不要把内嵌脚本紧跟在<link>标签后面。

3、无阻塞的脚本
无阻塞脚本的秘诀在于，在页面加载完成后才加载 JavaScript 代码。这就意味着在 window 对象的 onload事件触发后再下载脚本。

4、延迟加载脚本
Defer 属性指明本元素所含的脚本不会修改 DOM，因此代码能安全地延迟执行。

 <script type="text/javascript" src="script1.js" defer></script>

带有 defer 属性的<script>标签可以放置在文档的任何位置。对应的 JavaScript 文件将在页面解析到<script>标签时开始下载，但不会执行，直到 DOM 加载完成，即onload事件触发前才会被执行。任何带有 defer 属性的<script>元素在 DOM 完成加载之前都不会被执行，无论内嵌或者是外链脚本都是如此。
  
async。它的作用和 defer 一样，能够异步地加载和执行脚本，不因为加载脚本而阻塞页面的加载。但是有一点需要注意，在有 async 的情况下，JavaScript 脚本一旦下载好了就会执行，所以很有可能不是按照原本的顺序来执行的。如果 JavaScript 脚本前后有依赖性，使用 async 就很有可能出现错误。

5、动态脚本元素

通过标准 DOM 函数创建<script>元素
var script = document.createElement ("script");
   script.type = "text/javascript";
   script.src = "script1.js";
   document.getElementsByTagName("head")[0].appendChild(script);
  
  
通过监听 onload 事件加载 JavaScript 脚本
var script = document.createElement ("script")
script.type = "text/javascript";
//Firefox, Opera, Chrome, Safari 3+
script.onload = function(){
    alert("Script loaded!");
};
script.src = "script1.js";
document.getElementsByTagName("head")[0].appendChild(script);


通过检查 readyState 状态加载 JavaScript 脚本
var script = document.createElement("script")
script.type = "text/javascript";
 
//Internet Explorer
script.onreadystatechange = function(){
     if (script.readyState == "loaded" || script.readyState == "complete"){
           script.onreadystatechange = null;
           alert("Script loaded.");
     }
};
 
script.src = "script1.js";
document.getElementsByTagName("head")[0].appendChild(script);

readyState 有五种取值：
“uninitialized”：默认状态
“loading”：下载开始
“loaded”：下载完成
“interactive”：下载完成但尚不可用
“complete”：所有数据已经准备好
有时<script>元素会得到“loader”却从不出现“complete”，但另外一些情况下出现“complete”而用不到“loaded”。最安全的办法就是在 readystatechange 事件中检查这两种状态，并且当其中一种状态出现时，删除 readystatechange 事件句柄（保证事件不会被处理两次）。


通过函数进行封装
function loadScript(url, callback){
    var script = document.createElement ("script")
    script.type = "text/javascript";
    if (script.readyState){ //IE
        script.onreadystatechange = function(){
            if (script.readyState == "loaded" || script.readyState == "complete"){
                script.onreadystatechange = null;
                callback();
            }
        };
    } else { //Others
        script.onload = function(){
            callback();
        };
    }
    script.src = url;
    document.getElementsByTagName("head")[0].appendChild(script);
}
此函数接收两个参数：JavaScript 文件的 URL，和一个当 JavaScript 接收完成时触发的回调函数。属性检查用于决定监视哪种事件。最后一步，设置 src 属性，并将<script>元素添加至页面。此 loadScript() 函数使用方法如下：
  
loadScript()函数使用方法
  
loadScript("script1.js", function(){
    alert("File is loaded!");
});

通过 loadScript()函数加载多个 JavaScript 脚本

loadScript("script1.js", function(){
    loadScript("script2.js", function(){
        loadScript("script3.js", function(){
            alert("All files are loaded!");
        });
    });
});

此代码等待 script1.js 可用之后才开始加载 script2.js，等 script2.js 可用之后才开始加载 script3.js。虽然此方法可行，但如果要下载和执行的文件很多，还是有些麻烦。如果多个文件的次序十分重要，更好的办法是将这些文件按照正确的次序连接成一个文件。独立文件可以一次性下载所有代码（由于这是异步进行的，使用一个大文件并没有什么损失）。

