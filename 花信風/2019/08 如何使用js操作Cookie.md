[pixiv: 004]: # 'https://i.loli.net/2019/06/10/5cfdc5728ffa786026.jpg'

第一次使用Cookie保存记录。以下是操作Cookie的一些方法。
## 设置Cookie

```js
//expires是cookie在浏览器上的超时时间
function setCookie(name,value,expires) {
    if(expires) {
        var exp = new Date();
        exp.setTime(exp.getTime() + expires * 1000);
        document.cookie = name + "=" + escape(value) + ";expires=" + exp.toGMTString();
    } else {
        document.cookie = name + "=" + escape(value) + ";";
    }
}
```
## 获取指定name的cookie值

```js
function getCookie(name) {
    var result = document.cookie.match(new RegExp("(^| )" + name + "=([^;]*)(;|$)"));
    if (result != null) {
        return unescape(result[2]);
    }
    return null;
}
```
## 删除Cookie

```js
//删除cookie
function delCookie(name)
{
    var exp = new Date();
    exp.setTime(exp.getTime() - 1);
    var cval=getCookie(name);
    if(cval!=null)
        document.cookie= name + "="+cval+";expires="+exp.toGMTString();
}
```
## 将JSON数据存入Cookie,从Cookie获取JSON数据

```js
var obj={name: '123',age: 18};
setCookie("user",JSON.stringify(obj),null);
var jsonStr=getCookie("user");
if(jsonStr != null){
	console.log(JSON.parse(jsonStr));
}
```
