[pixiv: 002]: # 'https://i.loli.net/2019/06/10/5cfdc4504f6b866775.jpg'

# 什么是contenteditable？
HTML中的contentEditable的属性可以打开某些元素的可编辑状态．也许你没用过contentEditable属性．甚至从未听说过．contentEditable的作用相当神奇．可以让div或整个网页，以及span等等元素设置为可写。我们最常用的输入文本内容便是input与textarea，使用contentEditable属性后，可以在div,table,p,span,body,等等很多元素中输入内容．
如果想要整个网页可编辑，请在body标签内设置contentEditable
contentEditable已在html5标准中得到有效的支持。
在IE8下设置表格可写不支持，其他元素没有问题。在Firefox运行一切正常。谷歌浏览器运行一切正常。
# 如何实现可编辑的Div换行（Div + contenteditable）
1. 制作一个可编辑的Div

```html
<div id="editor"  placeholder="请输入您的内容" contenteditable="true" style="overflow-y:scroll;">
</div>
```

2. 给editor绑定键盘监听方法，回车发送消息，shift + 回车文本换行
```js
//绑定键盘监听事件
    $("#editor").bind('keydown',function(e){
        if(e.keyCode == 13 && !e.shiftKey){
            //按下了enter键，发送消息
            sendmsg();
            e.preventDefault();
        }
        if(e.shiftKey && e.keyCode == 13){
            按下了shift + enter组合键，文本换行
            var content=$("#editor").html();
            $("#editor").html(content+"<div><br/></div>");
            //e.preventDefault();
            //加上e.preventDefault(); shift+ enter效果会有问题，换行符号加上了，但是光标不会跑到下一行，而是回到原来行的行首
        }
        return;
    });
```
# 如何实现textarea换行
1. 制作一个文本域

```html
<textarea class="msgedit" cols="82" id="send_msg_text"  rows="5"></textarea>
```
2. 给文本域绑定键盘监听事件，回车发送消息，shift + 回车文本换行

```js
//绑定键盘按下时，文本域触发的事件
    $("#send_msg_text").bind('keydown',function(e){
        if(e.keyCode == 13 && !e.shiftKey){
            //按下了enter键，发送消息
            onSendMsg();
            e.preventDefault();
        }
        if(e.shiftKey && e.keyCode == 13){
            //按下了shift + enter组合键，文本换行
            var content=$("#send_msg_text").val();
            $("#send_msg_text").val(content+"\n");
            e.preventDefault();
        }
        return;
    });
```
