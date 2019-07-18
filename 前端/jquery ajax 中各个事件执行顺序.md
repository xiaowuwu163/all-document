# [jquery ajax 中各个事件执行顺序](https://www.cnblogs.com/kelelipeng/p/11045888.html)

## **https://www.cnblogs.com/yangjinwang/p/6513529.html**

**https://www.cnblogs.com/SongHuiJuan/p/8117890.html**

**https://blog.csdn.net/woshiwxw765/article/details/22393619**

## jquery ajax 中各个事件执行顺序如下：

1.ajaxStart(全局事件)

2.beforeSend

3.ajaxSend(全局事件)

4.success

5.ajaxSuccess(全局事件)

6.error

7.ajaxError (全局事件)

8.complete

9.ajaxComplete(全局事件)

10.ajaxStop(全局事件)

## 下面看个例子：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 $("#ACCOUNT_TYPE").bind('click', function() {
          //alert( $(this).val());
          var msg="";
          if($(this).val()=="B134002")//托管
          {
              //msg="托管";
              msg="ACCOUNT_TYPE_COM_T";
          }
          else if($(this).val()=="B134001")//存管
          {
              //msg="存管";
              msg="ACCOUNT_TYPE_COM_C";
          }
          else if($(this).val()=="-")//存管和托管all
          {
              //msg="存管和托管";
              msg="ACCOUNT_TYPE_COM_ALL";
          }
          else
          {
              alert("参数错误！");
              return;
          }
          
          $("#ACCOUNT_TYPE_COMDiv_son").children().remove();//先清除所有子元素
          $.ajax({
              type:"post",              
              url:"${ctx}/Rule/Combination/getdict",
              data:{type:msg},
              dataType:"json",
              success:function(json)
              {
                //$(object).children(selector).remove();  // 删除object元素下满足selector选择器的子元素，不填写则默认删除所有子元素
                 
                  
                  for(var i=0;i<json.length;i++)
                   {                
                     var checkBox=document.createElement("input");
                   //checkBox.setAttribute("type","checkbox");
                   checkBox.setAttribute("type","radio");
                   checkBox.setAttribute("id", json[i].value);
                   checkBox.setAttribute("value", json[i].value);
                   checkBox.setAttribute("name", "ACCOUNT_TYPE_COM");
                   checkBox.setAttribute("checked", true);
                   //checkBox.setAttribute("readonly", true);//
                   var li=document.createElement("span");
                   
                   li.style.display = "block";
                   li.style.width = 23+"%";
                   li.style.float = "left";
                   
                   li.appendChild(checkBox);
                   li.appendChild(document.createTextNode(json[i].label));
                   
                   $("#ACCOUNT_TYPE_COMDiv_son").append(li); 
     
                   }     
              }
              ,              
              beforeSend:function(mes)
              {
                  alert("beforeSend");
              },
              complete:function()
              {
                  alert("complete");
              }
          });
     });
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行这段代码，会先弹出 beforeSend  提示，然后加载success 方法里面的代码，最后弹出 complete  提示，这说明这段代码的几个方法的执行先后顺序是符合上面总结的顺序的！

## 全局事件的例子：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
<html>
<head>
<script type="text/javascript" src="/jquery/jquery.js"></script>
<script type="text/javascript">
$(document).ready(function(){
  $("div").ajaxStart(function(){
      
     $(this).html("<img src='/i/demo_wait.gif' />");
 alert("1.最先的全局方法");
  });
  $("button").click(function(){
    $("div").load("/example/jquery/demo_ajax_load.asp");
  });

  $("div").ajaxSend(function()
   {
     alert("2.发送前");
   });
  $("div").ajaxSuccess(function()
   {
     alert("3.成功后");
   });
 $("div").ajaxComplete(function()
   {
     alert("4.ajaxComplete");
   });
 $("div").ajaxStop(function()
   {
     alert("5.ajaxStop");
   });

});
</script>
</head>

<body>

<div id="txt"><h2>通过 AJAX 改变文本</h2></div>
<button>改变内容</button>

</body>
</html>
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

每次点击 "改变内容" 的这个按钮，都会先加载一次demo_wait.gif 这个图片，然后执行 ajaxSend，然后执行 ajaxSuccess，然后执行ajaxComplete，然后执行 ajaxStop ，这个执行顺序也是符合上面总结的顺序的！

 ____________________________________________________________________________________

**一、.ajaxStart()——ajax请求开始时触发** 

**描述：**ajax请求开始时触发 .ajaxStart()的回调函数，全局的，所有的ajax都可以用

**写法：**元素.ajaxStart(function({ajax请求开始时触发的代码})

```
$("#loading").ajaxStart(function () {
    $(this).show();  //ajax请求开始时#loading元素显示
});
```

**作用域：**全局

**某个ajax不受全局方法的影响：**

方法：将$.ajax({})的global设为false

```
$.ajax({
    url: "test.html",
    global:false
});
```

 

**二、.ajaxStop()——ajax请求结束时触发** 

**描述：**ajax请求结束时触发 .ajaxStop()的回调函数，全局的，所有的ajax都可以用

**写法：**元素.ajaxStop(function({ajax请求结束时触发的代码})

```
$("#loading").ajaxStop(function () {
    $(this).hide();   //ajax请求结束时#loading元素隐藏
});
```

**作用域：**全局

**某个ajax不受全局方法的影响：**

方法：将$.ajax({})的global设为false

```
$.ajax({
    url: "test.html",
    global:false
});
```