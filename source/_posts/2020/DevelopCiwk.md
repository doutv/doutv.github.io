---
title: TeaBreak网站后端笔记
top: true
cover: true
toc: true
mathjax: true
tags:
  - Backend
  - Django
  - Database
  - Project
categories:
  - Backend
date: 2020-09-02 08:46:51
summary:
---

# TeaBreak网站后端笔记
[TeaBreak](http://lguwelcome.online/)是一个类知乎的校内问答网站，目前提供问答与文章功能。

我负责的是网站的后端，框架是`django`。我之前从未用过`django`，那就现学呗。
把[django tutorial](https://docs.djangoproject.com/zh-hans/3.0/intro/tutorial01/)过一遍，了解了里面的一些概念和整个开发的流程。

## 设计数据库
- 目标是做一个类知乎的问答网站
- 先画ER Diagram，设计了`user`,`question`,`answer`,`comment`这几个表
- 那评论别人的评论该怎么做呢？
- 我上知乎查了一下这个问题，[发现了解决方案](https://www.zhihu.com/question/38959595)，只要在`comment`中加一个外键`reply_comment_id`就好了
- 在这个问题下面，我发现了另外一个问题：如何设计数据库使每个用户只能点赞/踩同一个回答/评论一次呢？
- 参考了别人的回答，添加`answer_vote`来存储某个用户对某条回答的投票信息，`comment_vote`来存储某个用户对某条评论的投票信息。
- 初步的数据库设计：
- ![ER Diagram](https://raw.githubusercontent.com/doutv/Picbed/master/img/DevelopCiwk-2020-07-10-09-05-06)
        

## django中建立ORM模型
django中的数据库在`models.py`中定义，用的是ORM。

遇到的一些问题及解决方案：
- 用于分类的字段，如`gender`只有两个取值，该如何表示？可以使用[Choices](https://docs.djangoproject.com/zh-hans/3.0/ref/models/fields/#choices)来显式定义:
    ```python
    class Gender(models.TextChoices):
        MAN = 'M'
        WOMAN = 'W'
    gender = models.CharField(max_length=1, choices=Gender.choices)
    ```
    Choices此处在纯后端的django中其实只起到了代码注释的作用，在生成的数据库语句中不会产生任何不同。也就是说，`gender="A"`也是可以的。
- 外键（foreign key）中的`on_delete`该如何设置？
  - 这是一个有关设计哲学的问题
  - 对于`on_delete`的6种模式介绍，[stackoverflow](https://stackoverflow.com/questions/38388423/what-does-on-delete-do-on-django-models)上面有精彩的回答
  - `CASCADE`是最严谨的模式，例如一个用户被删除了，那么他所有的回答和评论都会被删除，这样能最好地保证数据库的完整性（integrity）
  - `SET_NULL`能最好地保护数据不丢失，哪怕误操作删掉了一个用户，他所有的回答和评论都保存在数据库中，不会丢失。知乎上有时会看到“已注销用户”就是这个道理，虽然他的个人信息可能被永久删除了，但是他的回答和评论都保存在数据库中。（当然我不确定知乎是否采用了这种模式）
  - 我最后选择了`SET_NULL`：
    ```python
    user_id = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, db_column="user_id")
    ```

## API Document
+ [API Blueprint 一门类markdown的api设计语言](https://github.com/apiaryio/api-blueprint)
+ [一个在线编辑，预览，测试API的网站](https://apiary.io/)

生成出的API文档十分美观  
![API Document Example](https://raw.githubusercontent.com/doutv/Picbed/master/img/DevelopCiwk-2020-07-10-10-17-33)

## django前后端分离实践
django是自带前端界面的，但在我们的项目中，django只负责后端部分，前端用react完成。

由于我们项目较小，核心开发人员只有三人，我负责后端部分，另外两位同学负责前端，所以我们是一边完善API文档一边写程序的。后端只需要提供API，根据相应的请求返回对应的json。下面讲一些常用的函数/功能：
- `body_dict = json.loads(request.body.decode('utf-8'))` 读取POST中的字段
- `dict.get` python字典的get方法
- `models.save()` 更改某个model后保存
- `get_object_or_404` 获取单个符合条件的model
- `models.objects.filter` 返回一个`queryset`包含符合条件的所有model


### Convert Django Model object to dict
[StackOverflow上面对这个问题的解答](https://stackoverflow.com/questions/21925671/convert-django-model-object-to-dict-with-all-of-the-fields-intact)

我在项目中主要采用了2种方法：
1. `model_to_dict`：返回特定列且不需要返回外键的场景中。
2. StackOverflow上的custom function`to_dict`：返回model的所有字段。

`to_dict`也可以只返回model的特定列，写法如下：

```python
def to_dict(instance, except_fields=[]):
    opts = instance._meta
    d = {}
    for f in chain(opts.concrete_fields, opts.private_fields):
        if f.name in except_fields:
            continue
        d[f.name] = f.value_from_object(instance)
    for f in opts.many_to_many:
        if f.name in except_fields:
            continue
        d[f.name] = [i.id for i in f.value_from_object(instance)]
    return d
```

## 用户登录系统设计
用户成功登录后，用`set_cookie`方法将`token`（一串随机生成的字符串作为用户身份标识）放在用户的cookie中。在需要验证用户身份的场景中，只需要验证用户cookie中的token与数据库中该user_id对应的token是否相等即可，省去了每次登录的麻烦。token有过期时间，如果token过期了就相当于无效，用户需要重新登录获取新生成的token。

由于我并没有采用django中的身份验证模块，因此以下操作都需手动进行：生成token和过期日期expired_date，存储到数据库中，进行token比对。为了减少重复代码，我使用了装饰器来进行token比对：
```python
def post_token_auth_decorator(api):
    @wraps(api)
    def token_auth(request):
        body_dict = json.loads(request.body.decode('utf-8'))
        user = get_object_or_404(User, user_id=body_dict.get("user_id"))
        if request.COOKIES.get("token") != user.token:
            raise Exception("token incorrect")
        if user.expired_date < datetime.now():
            raise Exception("token expire")
        return api(request)
    return token_auth
```
在需要验证用户身份的场景中，如所有的POST操作，只需要在API前加上`@post_token_auth_decorator`即可。事实上，POST的API使用了2个装饰器（注意先后顺序）：
```python
@require_http_methods(["POST"])
@post_token_auth_decorator
def alter_user_info(request):
```
P.S. [装饰器教程，注意看下面的第一篇笔记](https://www.runoob.com/w3cnote/python-func-decorators.html)

使用secrets库中的`token_urlsafe`方法生成token并`set_cookie`：
```python
from secrets import token_urlsafe

user.token = token_urlsafe(TOKEN_LENGTH)
user.expired_date = datetime.now() + timedelta(days=TOKEN_DURING_DAYS)
response = HttpResponse({"message": "User register successfully"}, content_type='application/json')
response.set_cookie("token", user.token)
```
## 后端项目规范
随着我们团队开发人员的增多（前端2人+后端2人），提高协作效率显得尤为重要，为此我定义了后端项目规范。

### 后端工作流程workflow
+ 流程：`API Document` -> `git checkout feature`切换到`feature`分支进行开发 -> `models.py` -> `views.py` -> `urls.py` -> `postman testing on localhost` -> `git commit & sync` -> `postman testing on cloud server`这一步可以省略，一般在本地通过测试以后，服务器上应该也不会出现问题 -> 通过测试后将`feature`分支合并到`master` -> 上线部署 -> `API Document`
+ `models.py`数据库设计
  + 一般情况不需要更改，更改过多时迁移`migrate`会遇到错误，遇到错误的解决方法见`Database`。
+ `views.py`API设计:
  + 大部分的工作会在这里完成。最好是先在API文档中定义好request和response。
  + 查django官方文档或者google解决，也可参考之前的代码，POST和GET两种接口的写法几乎是固定的。
+ `urls.py`后端路由:
  + 将`views.py`里面的函数映射到某个url上，一般在完成`views.py`后修改。
  + 小心两个url发生冲突。例如：`api/handbook/<str:category>/<int:order>/`与`api/handbook/category/<str:category>/`会发生冲突
+ 注意API文档要与`views.py`，`urls.py`同步更新。


### git使用规范
+ 双分支管理：`master+feature`分支，`master`是服务器上跑的稳定版本，`feature`是开发中的不稳定版本，`feature`分支需要经过测试以及前后端沟通后才能发布到`master`分支。
+ commit信息要写完整，写详细，解决了某某需求，修复了某某bug，建议用中文来写，这样容易表达出自己的意思，如：`API: 修改get_suggested_questions接口，按照问题下的回答的点赞总数从大到小排序`。
+ 单次commit最好对应`confluence`中的一个需求，或是解决了一个bug。
+ 除了紧急修复运行中的bug，其他情况不要直接在服务器上面改代码，网站容易崩。

### 数据库注意事项
- ⚠尽量不要直接操作数据库，而是通过已经写好的API来操作。
- tips：下载`mysql workbench`来可视化管理数据库，进行敏感操作（更改/删除）时要小心。
- `/ciwkbe/settings.py`里面选好数据库的`.cnf`文件
- `python manage.py makemigrations qa`
- `python manage.py migrate qa`
- `migrate`迁移过程中发生错误怎么办？
  - 首先查看错误信息，常见的有：将某个字段改为`unique`时，由于该字段已经存在重复的数据，不满足`unique constraint`，数据库会报错。
  - 有些错误可以通过django的引导来更改，比如提供一个`default`值。
  - 而像上面所提到的错误则需要手动更改重复的数据，使该列满足`unique constraint`。
  - 有时候`migrate`进行了一部分，剩下另一部分由于出错而没有进行，或已经直接对数据库进行了相应更改。此时可以找到对应的迁移文件`/qa/migrations/xxxx_auto_yyyy.py`，手动将已经迁移的部分删除，再进行`migrate`。为了避免下次`migrate`因`migrate`不完整而出错，我的解决方法是：`migrate`全部完成后，将对应的迁移文件`/qa/migrations/xxxx_auto_yyyy.py`删除，再`python manage.py makemigrations qa`重新生成完整的迁移文件，然后通过该命令跳过该迁移`python manage.py migrate qa xxxx --fake`（`xxxx`是当次迁移文件的编号）