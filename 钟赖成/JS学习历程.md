## JS学习历程——AJAX简述

### 什么是AJAX？

一句话解释：AJAX 是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。而传统的网页（不使用 AJAX）如果需要更新内容，必需重载整个网页面。

比如说我们做一个类似股市的模块，不断的更新数据，如果我们在这个页面再添加一个评论功能的话，我们写字写了一半就会出现整个页面刷新，导致我们的评论白写了。所以说我们急需一门技术来解决这个问题，因此就出现了AJAX这么技术。有了AJAX之后，我们顺利的解决了这个问题。AJAX就是类似一个中介，我们可以把它想成类似淘宝这个工具，前端将数据传给AJAX这个工具，然后我们在我们的界面可以做我们的操作，而数据库和AJAX对象之间进行数据传输，之后数据库把数据传给AJAX，AJAX再传给前端，而且可以传给指定的那个模块，不会出现整个界面刷新。这样就成功的解决了我们的问题。

### AJAX工作原理

![AJAX](http://www.runoob.com/images/ajax.gif)

### 实例解析

**这是一个get请求的异步请求。**

```
function sendXML(){
        var XMLHttp;
        if(window.ActiveXObject){
            //如果是IE
            XMLHttp = new ActiveXObject("Microsoft.XMLHTTP");
        }else{
            //如果是其他的浏览器
            XMLHttp = new XMLHttpRequest();
        };
        //定义一个url
        var url = "register.php?username="+document.getElementById('username').value+"&age="+document.getElementById('age').value;
        //这里用get请求，url是传输的地址，true是异步传输，false同步传输
        XMLHttp.open("get",url,true);
        XMLHttp.onreadystatechange = function(){
            if(XMLHttp.readyState == 4){
                if(XMLHttp.status == 200){
                    var response = XMLHttp.responseText();
                    alert("response");
                }
            }
        }
        XMLHttp.send(null);
    }
```

**这段代码是将username和age的值传给register.php函数，请求方式是get请求，true是异步请求。这是js写的。**

**接下来用jq来写一段AJAX代码：**

```
$.ajax({
        //类型是get方式
        type:"get",
        //发送请求的网址
        url:"register.php",
        //是否是异步请求
        async:"false",
        //请求的数据类型
        dataType:"jsonp",
        //回调函数名
        jsonpCallback:"fly",
        //成功之后调用的函数
        success:function(data){
            alert("");
        },
        //失败调用的函数
        error:function(){
            alert("");
        }
    });
```

**这段代码就是jq里边的AJAX的写法。**

**接下来是POST请求，首先是介绍JS中的代码：**

```
var xmlHttp;        
function S_xmlhttprequest(){ 
 if(window.ActiveXObject){  
  xmlHttp=new ActiveXObject('Microsoft.XMLHTTP');
  }else if(window.XMLHttpRequest){  
   xmlHttp=new XMLHttpRequest();
   }
 }

function funphp100(n){
var data = "text=" +n;　　//多个参数的，往后加
 S_xmlhttprequest();
xmlHttp.open("POST","for.php",true);  
xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
 xmlHttp.onreadystatechange=byphp;
 xmlHttp.send(data);
 }

function byphp(){
var byphp100=xmlHttp.responseText;
document.getElementById("php100").innerHTML=byphp100;
 }
```

下面是JQ里边的post代码：

```
$.ajax({
            //提交数据的类型 POST GET
            type:"POST",
            //提交的网址
            url:"testLogin.aspx",
            //提交的数据
            data:{Name:"sanmao",Password:"sanmaoword"},
            //返回数据的格式
            datatype: "html",//"xml", "html", "script", "json", "jsonp", "text".
            //在请求之前调用的函数
            beforeSend:function(){$("#msg").html("logining");},
            //成功返回之后调用的函数             
            success:function(data){
           $("#msg").html(decodeURI(data));            
            }   ,
            //调用执行后调用的函数
            complete: function(XMLHttpRequest, textStatus){
               alert(XMLHttpRequest.responseText);
               alert(textStatus);
                //HideLoading();
            },
            //调用出错执行的函数
            error: function(){
                //请求出错处理
            }         
         });
```

**这就是基本的ajax方法。今后用的时候可以直接按照上边写的稍微改动一点就行。**