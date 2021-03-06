## SSO单点登陆

*最后更新 2018/4/21*

### 简介
应用端自身维护应用自身的登录状态，如果未登录，跳转至用户系统的登录页面，用户在用户系统进行登录操作。
用户登录完成后，登陆页面会下发一次性验证码和用户UID，并将用户**在前端**重定向至应用端的回调接口。
应用在收到用户前端发来的一次性验证码和UID后，在服务器访问用户系统的验证接口，验证用户发来的验证码是否为真，验证通过后会返回用户信息，应用端应据此创建登录状态。
不建议在数据库中存储用户详情（若存储应每次更新用户信息，或使用缓存服务存储）。

### 需要在用户系统申请
name: 应用名称
appid：应用ID，四位数字
appkey：应用与用户系统连接时验证密钥，不应泄露
callback：应用回调接口，应当接受GET请求，请求类似
（如果非网页应用，直接调用登录接口，可直接获得验证信息，不需要callback）
```
test.fyscu.com/callback?uid={32位uid}&token={32位token}
```
权限：用户系统设计了RBAC权限模型，可以返回用户的权限代码。

### 页面

#### /{appid}
若请求登录，将用户定向到此页面。登录成功后前端跳转至callback。

### 接口

#### /api/challenge
应用服务器访问，验证/获得用户信息。

请求：
```
POST https://sso.fyscu.com/api/challenge

appid={应用ID}&appkey={应用密钥}&uid={收到的用户uid}&token={收到的用户token}
```

返回：
```
{"code":200,"data":{"id":"{用户内部ID}","user_id":"{32位uid}","name":"{姓名}","cell":"{手机号}","email":"{邮箱 可选}","qq":"{QQ，尚未用}","reg_time":"{注册时间，有时为null}","last_update":"{资料更新时间}","permissions":[四位权限代码,可能有多个]}}
```

错误码：
```
401 应用/token验证失败
404 token不存在
408 token过期
444 信息不全
```