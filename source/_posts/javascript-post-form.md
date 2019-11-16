---
title: javascript模拟post表单提交
categories:
    - Javascript
tags:
    - Javascript
    - csrf
date: 2018-11-11 00:00:00
---


# 背景
今天遇到一个问题，我在使用spring-security的时候，配置了登出链接：`<logout logout-url="/logout" logout-success-url="/" />`.但是登出的button每当点击的时候就会报异常：服务器没有对应的方法（/logout）。

<!-- more -->

# 分析
因为在spring-security中默认开启了csrf，所以在提交表单的时候需要用一个隐藏button提交token，就以注册页面举例：
```xml
<form role="form" action="${basePath}/register" method="post" id="form">
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
<div class="form-group">
<label for="username">用户名</label>
<input type="text" class="form-control" id="username" name="username" placeholder="用户名">
</div>
<div class="form-group">
<label for="password">密码</label>
<input type="password" class="form-control" id="password" name="password" placeholder="密码">
</div>
<button type="submit" class="btn btn-default">注册</button>
</form>
```
所以对于登出接口也是一样，需要以post方式提交，并且加入token。

# 解决
专门写一个form表单，映射到退出的button上，这样不是不可以，但是不美观（对于前端技术，没有深入研究）。
所以我就想写一个js脚本，提交post请求，这个button就来调一下这个js函数就行了。
那么我的js脚本如下：
```javascript
function postRequest(path, params, method) {
    method = method || "post";

    var form = document.createElement("form");
    form.setAttribute("method", method);
    form.setAttribute("action", path);

    for(var key in params) {
        if(params.hasOwnProperty(key)) {
            var hiddenField = document.createElement("input");
            hiddenField.setAttribute("type", "hidden");
            hiddenField.setAttribute("name", key);
            hiddenField.setAttribute("value", params[key]);

            form.appendChild(hiddenField);
         }
    }

    document.body.appendChild(form);
    form.submit();
}
```

在需要登出的地方，进行如下调用即可：
```xml
<a href="javascript:postRequest('${basePath}/logout', {'${_csrf.parameterName}': '${_csrf.token}'});">退出</a>
```

OK，搞定。
如有不对之处，还请各位赐教。
