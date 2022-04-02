---

layout: post
title:  "Django中views数据查询使用locals()函数进行优化"
categories: 经验总结
tags: Django 经验总结
author: 303Donatello

---


* content
{:toc}
# 优化场景
>利用视图函数（views）查询数据之后可以通过上下文context、字典、列表等方式将数据传递给HTML模板，由template引擎接收数据并完成解析。但是通过context传递数据可能就存在在不同的视图函数中使用重复的查询语句，所以可以通过将重复查询语句设置全局变量，配合locals()函数进行数据查询与传递。





# 优化前
```
def index(request):
    threatname = '威胁情报展示'
    url = 'www.testtip.com'
    allthreat = Threat.objects.all()
    #推荐位的威胁情报
    rec = Threat.objects.filter(rec__id=1)[:3]
    #情报标签
    threat_tags = Tag.objects.all()
    #将上述数据封装至上下文中
    context = { 
            'threatname': threatname,
            'url': url,
            'allthreat': allthreat,
            'rec':rec,
            'threat_tags':threat_tags,
    }
    #通过render传递上下文至模板templates
    return render(request,'index.html',context)
def threatshow(request,tid):
    allthreat = Threat.objects.all()
    #推荐位的威胁情报
    rec = Threat.objects.filter(rec__id=1)[:3]
    #情报标签
    threat_tags = Tag.objects.all()
    # 热门情报数据
    hot_threat = Threat.objects.filter(hot__id=x)[:6]
    #将sitename&url&allarticle封装至上下文中
    context = { 
            'allthreat': allthreat,
            'rec':rec,
            'threat_tags':threat_tags,
            'hot_threat':hot_threat,
    }
    return render(request, 'threatshow.html',context)
```
上面可以看到 `views` 里面有 `index()`和 `threatshow()`两个视图函数，在这两个视图函数中有三个相同的数据查询语句：
```
    allthreat = Threat.objects.all()
    #推荐位的威胁情报
    rec = Threat.objects.filter(rec__id=1)[:3]
    #情报标签
    threat_tags = Tag.objects.all()
```

# 优化后
## 设置全局变量
```
    # 全局定义常用查询数据参数
def global_variable(request):
    allthreat = Threat.objects.all()
    #推荐位的威胁情报
    rec = Threat.objects.filter(rec__id=1)[:3]
    #情报标签
    threat_tags = Tag.objects.all()
    return locals()
```
 `views`中定义上述全局变量后，通过locals()函数优化如下：
```
def index(request):
    threatname = '威胁情报展示'
    url = 'www.testtip.com'
    #通过render传递上下文至模板templates
    return render(request,'index.html',locals())
def threatshow(request,tid):
    # 热门情报数据
    hot_threat = Threat.objects.filter(hot__id=x)[:6]
    return render(request, 'threatshow.html',locals())
```
`Python`中的`locals()`函数会以字典类型返回当前位置的全部局部变量，也就是返回当前`index()`、`threatshow()`视图函数中定义的局部数据查询结果，加上全局变量当中已经完成了其他剩余数据查询，所以在满足数据查询需求的基础上完成了视图函数优化。