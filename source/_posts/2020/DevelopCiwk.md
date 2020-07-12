---
title: 学校迎新网站设计
top: false
cover: false
toc: true
mathjax: true
tags:
  - Backend
  - Django
  - Database
categories:
  - Backend
date: 2020-07-10 08:46:51
summary:
---

# 学校迎新网站设计
又是一天早上，我坐在电脑前摸鱼。突然，杨同学发来了消息，邀请我参与学校迎新网站（CUHKSZ Welcome Wall）的设计。
其实我一直都想做一个网站，就是懒而已，需要一个项目来驱动。既然他都主动邀请我了，那肯定上啊。

我负责的是网站的后端，框架是`django`。我之前从未用过`django`，那就现学呗。
把[django tutorial](https://docs.djangoproject.com/zh-hans/3.0/intro/tutorial01/)过一遍，了解了里面的一些概念和整个开发的流程。

## 设计数据库
- 目标是做一个类知乎的问答网站
- 先画ER Diagram，设计了`user`,`question`,`answer`,`comment`这几个表
- 那评论别人的评论该怎么做呢？
- 我上知乎查了一下这个问题，[发现了解决方案](https://www.zhihu.com/question/38959595)，只要在`comment`中加一个外键`reply_comment_id`就好了
- 在这个问题下面，我发现了另外一个问题：如何设计数据库使每个用户只能点赞/踩同一个回答/评论一次呢？
- 参考了别人的回答，添加`answer_vote`来存储某个用户对某条回答的投票信息，`comment_vote`来存储某个用户对某条评论的投票信息。
- 最终的数据库设计：
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
- 外键（foreign key）中的`on_delete`该如何设置？
  - 这是一个有关设计哲学的问题
  - 对于`on_delete`的6种模式介绍，[stackoverflow](https://stackoverflow.com/questions/38388423/what-does-on-delete-do-on-django-models)上面有精彩的回答
  - `CASCADE`是最严谨的模式，例如一个用户被删除了，那么他所有的回答和评论都会被删除，这样能最好地保证数据库的完整性（integrity）
  - `SET_NULL`能最好地保护数据不丢失，哪怕误操作删掉了一个用户，他所有的回答和评论都保存在数据库中，不会丢失。知乎上有时会看到“已注销用户”就是这个道理，虽然他的个人信息可能被永久删除了，但是他的回答和评论都保存在数据库中。（当然我不确定知乎是否采用了这种模式）
  - 我最后选择了`SET_NULL`：
    ```python
    user_id = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, db_column="user_id")
    ```

# API Document
+ [API Blueprint 一门类markdown的api设计语言](https://github.com/apiaryio/api-blueprint)
+ [一个在线编辑，预览，测试API的网站](https://apiary.io/)

生成出的API文档十分美观  
![API Document Example](https://raw.githubusercontent.com/doutv/Picbed/master/img/DevelopCiwk-2020-07-10-10-17-33)