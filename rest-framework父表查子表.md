---
title: rest_framework父表查子表
date: 2016-07-26 10:10:39
tags:
    - rest_framework
    - relation
    - 父表查子表
categories: django
---

## 引言
``` bash
    表A的外键是B表，想在django接口中查询B表把A表的信息携带出来，
    rest_framework可以分方便的实现,另外django配合上rest_framework，
    编写API将是你如鱼得水，写起来非常方便。
```
<!--more-->
## 需求
``` bash
    显示对应Duoshuod的评论的user_id和评论内容, 统计评论总数
```
## 实现
### model设计
``` bash
    class Duoshuo(models.Model):
        create_time = models.DateTimeField(auto_now_add=True)
        text = models.CharField(max_length=128)

        def __unicode__(self):
            return str(self.id)


    class DuoshuoComment(models.Model):
        feed = models.ForeignKey(Duoshuo, related_name='comment_duoshuo')
        user_id = models.CharField(max_length=255)
        content = models.CharField(max_length=255)

        def __unicode__(self):
            return self.content
```

### serializers.py
```bash
    # coding=utf-8
    from app.modele import Duoshuo, DuoshuoComment

    class FeedSerializer(serializers.ModelSerializer):

        duoshuo_comment = serializers.PrimaryKeyRelatedField('
                        source='comment_duoshuo',
                        many=True,
                        read_only=True)
        comment_num = serializers.SerializerMethodField(
                        'comment_total')
        class Meta:
            model = Duoshuo
            fields = ('id', 'duoshuo_comment', 'comment_num')

        def comment_total(self, obj):
             total = DuoshuoComment.objects.filter(feed=obj).count()
            return total
```

## 拓展
```bash
    在serializers.py还可以写相关的函数来整理查询后的结果，可查看官网。
    rest_framework在协助django编写api方面非常方便，有兴趣的可以在官方查询
    如何在django中使用rest_framework。
```
[rest_framework官网](http://www.django-rest-framework.org/)
