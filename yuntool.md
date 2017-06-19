---
title: yuntool
date: 2016-09-29 17:41:16
tags:
    - 日常运营工具包
    - orm
    - 绘图
    - 邮件发送
categories: 运营工具包
---

## 引言

``` bashgng
        工作中总会遇到运营人员提出统计某个指标的需求，诸如需要绘图并邮件发送等需求，工作之余总结了一个工具包，包括
    数据库查询（就像使用django那样快速方便的查询）， 图表绘制，邮件发送，其他功能还在持续更新中。这些是我工作遇到
    的，有觉得不合适的地方欢迎提出修改意见。
```

## 项目地址
[yuntool](https://github.com/yunsonbai/yuntool)

<!--more-->
## 项目简介
* 应用：运营数据统计；快熟查询数据库异常数据；快速数据进行增删改查
* 版本：0.4
* 安装：
    * git clone git@github.com:yunsonbai/yuntool.git
    * python setup.py install
* 主要功能和特点:
    * 采用orm方式操作数据库
    * 支持画图
        * 曲线图
        * 柱形图
    * 制表excel
    * 支持邮件发送
        * 纯文字
        * 图加文字


## 部分例子

### 数据查询
```python
    from yuntool.db.field import CharField
    from yuntool.db.models import Model
    from yuntool.chart.sheet import create_sheet
    from yuntool.chart.plot import draw_curve, draw_bar
    from yuntool.email.smtp import send_mail

    def test_get_orm():
        select_term = {
            'label__une': 2,
            # 'type__eq': 100,
            # 'send_time__gte': '2015-09-25 00:00:00',
            # 'send_time__gte': send_time_gte + ' 00:00:00',
            # 'send_time__lte': send_time_gte + ' 23:59:59'
        }
        # print '---------------{0}-------------------'.format(send_time_gte)
        queryset = TestOrm.objects.filter(**select_term)
        count = queryset.count()
        res = queryset.data()
        print(count)
        data = []
        for r in res:
            # only print the fields that are defined in TestOrm
            data.append([r.title, r.label])
            print(r.title)
        print('----------------------')
        res_limit = queryset.limit(1, 2).data()
        for r in res_limit:
            # only print the fields that are defined in TestOrm
            print(r.title)
        # ---------------------
        print('------------------')
        res = TestOrm.objects.filter(label=1, title='test10').data()
        for r in res:
            # only print the fields that are defined in TestOrm
            data.append([r.title, r.label])
            print(r.title)

        title = 'test_sheet'
        hearder_list = ['title', 'label']
        f = create_sheet(title, hearder_list, data)
        new_f = open('text.xlsx', 'wb')
        new_f.write(f.read())
        new_f.close()
        f.close()
```

### 数据更新
```python
    def test_update_orm():
        update_data = {
            'title': 'hello yunsonbai',
            'label': 10
        }
        res = TestOrm.objects.filter(id__in=[1, 2]).data()
        for r in res:
            r.update(**update_data)
        # or
        res = TestOrm.objects.filter(id=3).first().data()
        res.update(**update_data)
```

### 绘图
```python
    def test_bar():
        x = [
            ['2016-06-28', '2016-06-29', '2016-06-30',
             '2016-07-01', '2016-07-02', '2016-07-03', '2016-07-04'
             ],
            ['2016-06-28', '2016-06-29', '2016-06-30', '2016-07-01',
             '2016-07-02', '2016-07-03', '2016-07-04']]
        y = [
            [270, 279, 288, 273, 248, 232, 293],
            [2482, 1890, 2359, 7506, 14561, 14741, 16191]]
        picture = draw_bar(
            x, y, xlabel=['date', 'date'], ylabel=['num', 'num1'], merge=True)
        new_f = open('text_merge.png', 'wb')
        new_f.write(picture.read())
        new_f.close()
        picture.seek(0)
        return picture
```
#### 图片展示
* 柱形图
{% img  https://yunsonbai.github.io/images/yuntool/text.png %}
* 条形图
{% img  https://yunsonbai.github.io/images/yuntool/text_curve.png %}

### 邮件发送
```python
    def test_email(picture):
        # test email
        from_user = 'yunsonbai@sohu.com'
        from_user_passwd = 'xxxxxx'
        mail_server = '192.168.95.xx'
        mail_server_port = 'xx'
        to_users = ['1942893504@qq.com']
        try:
            subject = '关于test'.decode('utf-8')
        except:
            subject = '关于test'
        content = '请回复'
        send_mail(
            from_user, from_user_passwd, to_users,
            subject, content, mail_server,
            mail_server_port=mail_server_port,
            picture=picture.getvalue())
        picture.close()
```