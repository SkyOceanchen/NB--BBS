# BBS表设计

## 项目开发的流程

1. 需求分析
   架构师+产品经理+开发组组长
   在去客户公司谈需求之前，先事先估摸着这个项目应该怎么做
   里面有哪些坑的点 提前想好比较简单的解决方案 
   在跟客户谈的时候 有意识的引导客户朝着你已经想好的方案上去提需求

2. 项目设计
   架构师干的活
   项目的报价(每个程序员按照人头 每天2000+左右)
   语言的选择
   框架的选择
   数据库的选择(主库用什么  缓存库)
   功能划分
   开发部开发组长开会分发任务

3. 分组开发
   架构师和开发组长将项目整体的框架搭建出来
   然后让小组成员各自朝着各个部分填写代码即可

4. 测试
   显而易见的bug如果你自己没有发现，测试部分的如果发现了 你可能就会面临扣绩效的场面
   基本薪资  6000     扣1050
   岗位津贴  4000      
   绩效      2000

   ....

   1. 自己写测试脚本
   2. 测试部分专门测试 
   3. 测试部分一般都是妹纸
   4. 

5. 交付上线
   交给你们公司的运维人员或者是客户公司的运维人员

## 表

#### 用户表                 用户表和个人站点表是一对一的关系

个人站点表

文章标签表             标签与个人站点一对多

文章分类表             分类与个人站点是一对多

#### 文章表                 

文章和个人站点 是一对多
文章与标签是多对多的关系
文章与分类是一对多

#### 点赞点踩表

用来存哪个用户给哪篇文章点了赞还是点了踩
user    一对多用户表
article 一对多文章表
is_up   普通字段

本质:
一张表中的一条数据能否对应另外一张表的多条数据
另外一张表的一条数据能够对应当前的表多条件
user_id        article_id               is_up
1               1                           1
1               2                           0
1               3                           1
2               1                           1

#### 评论表

记录哪个用户给哪篇文章评论了哪些内容
user     一对多用户
article  一对多文章
content 
parent   一对多评论表   自关联  
to='Comment'
to='self'



id     user_id       article_id        parent_id
1        1                   1               
2        2                   1               1
3        3                   1               1





表  七张表

用户表
扩展auth_user表  
phone  电话号码
avatar  用户头像
create_time  注册时间
DateField(auto_now_add=True)

blog    一对一个人站点

个人站点表
站点名称
站点标题
站点样式

文章标签表
标签名字
blog    一对多个人站点

文章分类表
分类名字
blog    一对多个人站点

文章表
文章标题
文章摘要
文章内容
创建时间

#### 外键字段

1.一对多个人站点
2.多对多标签
3.一对多分类

## 数据库优化设计(******)

评论数 点赞数  点踩数 确实可以通过跨表查询得出   走数据库频率太高
评论数   普通字段
点赞数   普通字段
点踩数   普通字段





文章与标签的第三张表
article  一对多
tag      一对多

点赞点踩表
记录哪个用户给哪篇文章点了赞还是点了踩

user        一对多
article     一对多
is_up       普通字段  0/1



评论表
记录哪个用户给哪篇文章评论了上面内容
根评论
子评论

user        一对多
article     一对多
content     普通字段
create_time 
parent  用来表示当前评论是否是跟评论还是子评论  没值就是根评论 有值 值就是根评论的主键值
自关联 
self
数据库同步

avatar = models.FileField(upload_to='avatar/',default='avatar/default.png')

is_up = models.BooleanField()  # 传True/False    自动存成0/1

content = models.TextField()  # 大段html内容

## 数据库优化字段(******)

comment_num = models.BigIntegerField(default=0)
up_num = models.BigIntegerField(default=0)
down_num = models.BigIntegerField(default=0)

parent = models.ForeignKey(to='self',null=True)  # 自关联



# 注册功能

1.注册功能往往都会由很多校验性的需求  所以这里我们用到了forms组件
项目中可能有多个地方需要用到不同的forms组件 为了解耦合 但是创建一个py文件 专门用来存放项目用到的所有的forms组件
校验 用户名 密码 确认密码 邮箱
借助于钩子函数 校验用户名是否存在  密码与确认密码是否一致
2.创建路由 写视图函数
将实例化产生的forms类对象 利用模板语法传递到前端页面
再利用forms组件自动渲染前端获取用户输入的标签
3.用户输入数据后  采用ajax的方式朝后台提交数据
1.考虑到获取用户输入的input框可能会有很多 在同ajax获取的时候 可能会比较繁琐
借助了form标签有一个方法 自动将内部所有的普通键值 打包成一个容器类型

```python
$('#myform').serializeArray()
2.你需要提交的数据不单单只有普通键值 还有文件 可以借助 内置对象 FormData
如何获取文件对象
$('#myfile')[0].files[0]  # forms组件渲染错误的信息    
```

4.用户头像前端动态展示
利用内置对象 文件阅读器   new FileReader
等待什么什么加载完毕再执行  onload

```python
window.onload
fileReader.onload = function(){
文件阅读器加载完毕之后 再执行的操作
}
```



5.针对ajax提交的数据 后端在返回信息的时候  通常都是一个字典
在创建数据的时候利用**直接打散字典的形式 传输数据

6.将错误的信息传递给前端
前端渲染错误信息的时候 遇到了一个问题  如何将对应的信息渲染到对应的input框下面的span标签中
研究forms组件渲染的input框的id值的特点  id_字段名
发现后端传过来的字典的key就是一个个的字段名  自己拼接处id值

```js
<script>
    $('#myfile').change(function () {
        // 文件阅读器
        //1 产生一个文件阅读器对象
        var fileReader = new FileReader();
        //2 获取用户上传的文件
        var fileObj = $(this)[0].files[0];
        //3 让文件阅读器读取该文件
        fileReader.readAsDataURL(fileObj);  // 这一步是异步 提交完之后直接运行下一行
        //4 利用文件阅读器将文件展示出来
        fileReader.onload = function () {
            $('#myimg').attr('src', fileReader.result)
        }
    });

    $('#commit').click(function () {
        // 由于你要发送的数据即有普通的键值 又有文件 所以考虑使用内置对象FormData
        var formDataObj = new FormData();
        // 1 朝对象中添加普通的键值对
        // $('#myform').serializeArray()自动获取到内部所有的普通键值对
        $.each($('#myform').serializeArray(),function (index,obj) {
            formDataObj.append(obj.name,obj.value)
        });
        // 2 手动添加文件数据
        formDataObj.append('avatar',$('#myfile')[0].files[0]);

        // 发送ajax请求
        $.ajax({
            url:'',
            type:'post',
            data:formDataObj,

            // 发送文件 需要修改两个参数
            contentType:false,
            processData:false,

            success:function (data) {
                if (data.code == 1000){
                    window.location.href = data.url
                }else{
                    $.each(data.msg,function (index,obj) {
                        // index就是一个个的报错字段名  obj就是数组 里面是报错信息
                        // 手动拼接对应的input框的id值
                        var targetId = '#id_' + index;
                        // $('#id_username') $('#id_password')
                        $(targetId).next().text(obj[0]).parent().addClass('has-error')
                    })
                }
            }

        })

    });
    $('input').focus(function () {
        $(this).next().text('').parent().removeClass('has-error')
    })

</script>
```



# 登录功能

1.前端页面搭建
图片验证码
2.图片验证码 
img标签 src可以接收的数据
1.图片的具体路径
2.图片的二进制数据
3.后端url(自动朝该url发送get请求)

3.后端代码推导
操作图片的模块pillow

4.完成后跳转到网站首页
展示网站所有的博客内容
主页
用户没有登录导航条展示 登录 注册按钮
用户登录的情况下 展示用户的用户名和更多操作



图片验证码的实现

```python
from PIL import Image,ImageDraw,ImageFont
import random
from io import BytesIO,StringIO
"""
BytesIO,  能够存储数据 并以二进制的格式再返回给你
StringIO  能够存储数据 并以字符串的格式再返回给你
"""
"""
Image,  产生图片的
ImageDraw,  产生画笔的
ImageFont  控制字体样式
"""
def get_random():
    return random.randint(0,255),random.randint(0,255),random.randint(0,255)
```



```
# 图片验证码相关
def get_code(request):
    
    # 推到思路4(终极步骤)  图片上写字
    img_obj = Image.new('RGB',(310,35),get_random())
    img_draw = ImageDraw.Draw(img_obj)  # 生成一个画笔对象
    img_font = ImageFont.truetype('static/font/111.ttf',30)  # 字体的样式
    """
    所有的字体样式都是由.ttf结尾的文件控制的
    """
    # 随机生成验证码  a~z  A~Z  0~9
    code = ''
    for i in range(5):
        random_upper = chr(random.randint(65,90))
        random_lower = chr(random.randint(97,122))
        random_int = str(random.randint(0,9))
        temp = random.choice([random_upper,random_lower,random_int])
        # 将产生的随机字符写到图片上
        img_draw.text((i*45+45,0),temp,get_random(),img_font)
        code += temp
    print(code)
    # 将随机验证码存储取来  以便其他函数调用
    request.session['code'] = code

    io_obj = BytesIO()
    img_obj.save(io_obj,'png')
    return HttpResponse(io_obj.getvalue())
```



# admin后台管理(******)

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143342756-1339311345.png)

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143442400-1400530425.png)

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143454953-442389893.png)

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143510832-1666080110.png)

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143526698-163332684.png)

能够对注册了的模型表 生成增删改查起码四个页面

http://127.0.0.1:8000/admin/app01/userinfo/  查看用户
http://127.0.0.1:8000/admin/app01/userinfo/1/change/  编辑用户
http://127.0.0.1:8000/admin/app01/userinfo/add/  添加用户
http://127.0.0.1:8000/admin/app01/userinfo/2/delete/  删除用户

http://127.0.0.1:8000/admin/app01/blog/  查看个人站点
http://127.0.0.1:8000/admin/app01/blog/1/change/  编辑个人站点
http://127.0.0.1:8000/admin/app01/blog/add/  添加个人站点
http://127.0.0.1:8000/admin/app01/blog/2/delete/  删除个人站点



用户头像的渲染
网站默认的静态文件资源 默认放在static文件夹下
用户上传的静态文件资源 也应该单独存放在某一个文件夹

# 规定用户上传的静态文件资源全部放到某一个指定的文件夹下

MEDIA_ROOT = os.path.join(BASE_DIR,'media')

from django.views.static import serve
from BBS import settings

# 暴露给外界的后端文件资源

url(r'^media/(?P<path>.*)',serve,{'document_root':settings.MEDIA_ROOT})

settings中的配置

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143552619-1568160862.png)



urls中的配置

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143554750-58749354.png)

# 图片防盗链

请求来的时候 会检测当前请求的源地址是否来自于网站自己的
refer  放在请求头里面的数据   用来表示你是属于哪个网站

如果还想用 最简单粗暴的方式就是先下载到本地(写爬虫自动下)



# 个人站点页面搭建

侧边栏展示

## 日期归档

日期归档      content                   create_time             month
1           111                     2019-11-1              2019-11 
1           111                     2019-11-25             2019-11
1           111                     2019-11-10             2019-11
1           111                     2019-10-1              2019-10

官方提供的文档

```python
from django.db.models.functions import TruncMonth
-官方提供
from django.db.models.functions import TruncMonth
Article.objects
.annotate(month=TruncMonth('create_time'))  # Truncate to month and add to select list
.values('month')  # Group By month
.annotate(c=Count('id'))  # Select the count of the grouping
.values('month', 'c')  # (might be redundant, haven't tested) select month and count
```

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143643348-785791378.png)

当你写的url再访问的时候发现没有命中
那么你应该去urls.py中查看是否被前面的url中途拦截了  最省力的解决方式 就是调换位置



# 自定义inclusion_tag，区域需要经常被使用 并且需要传参数才能够渲染

## 侧边栏筛选功能



当页面上的某一块区域需要经常被使用 并且需要传参数才能够渲染出来 
那么你可以考虑使用inclusion_tag

自定义inclusion_tag的步骤 跟自定义过滤器和标签是一样的
1.创建一个名字必须交templatetags文件夹
2.新建任意一个py文件
3.py文件内 写固定的两行代码
from django.template import Library
register = Library()

工作原理 类似于 函数的调用过程
你在使用inclusion_tag的时候 可以为其传参   它会将参数传递给一个html页面

该页面会利用传递过来的参数  渲染页面   然后将渲染好的页面 放在调用inclusion_tag的地方

```python
from django.template import Library
from app01 import models
from django.db.models.functions import TruncMonth
from django.db.models import Count

register = Library()


@register.inclusion_tag('left_menu.html')
def index(username):
    user_obj = models.UserInfo.objects.filter(username=username).first()
    blog = user_obj.blog
    # 1.查询当前用户所有的分类及分类下的文章数
    category_list = models.Category.objects.filter(blog=blog).annotate(count_num=Count('article__pk')).values_list(
        'name', 'count_num', 'pk')

    # 2.查询当前用户所有的标签及标签下的文章数
    tag_list = models.Tag.objects.filter(blog=blog).annotate(count_num=Count('article__pk')).values_list('name',
                                                                                                         'count_num',
                                                                                                         'pk')

    # 3.按照年月统计文章数
    date_list = models.Article.objects.filter(blog=blog).annotate(month=TruncMonth('create_time')).values(
        'month').annotate(count_num=Count('pk')).order_by('-month').values_list('month', 'count_num')
    return locals()
    # return {'username':username,'blog':blog}
```

```html
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <h3 class="panel-title">文章分类</h3>
                </div>
                <div class="panel-body">
                    {% for category in category_list %}
                        <p><a href="/{{ username }}/category/{{ category.2 }}/">{{ category.0 }}({{ category.1 }})</a></p>
                    {% endfor %}
                </div>
            </div>
            <div class="panel panel-warning">
                <div class="panel-heading">
                    <h3 class="panel-title">文章标签</h3>
                </div>
                <div class="panel-body">
                    {% for tag in tag_list %}
                        <p><a href="/{{ username }}/tag/{{ tag.2 }}">{{ tag.0 }}({{ tag.1 }})</a></p>
                    {% endfor %}
                </div>
            </div>
            <div class="panel panel-danger">
                <div class="panel-heading">
                    <h3 class="panel-title">日期归档</h3>
                </div>
                <div class="panel-body">
                    {% for xxx in date_list %}
                        <p><a href="/{{ username }}/archive/{{ xxx.0|date:'Y-m' }}/">{{ xxx.0|date:'Y年-m月' }}({{ xxx.1 }})</a></p>
                    {% endfor %}
                </div>
            </div>
```



# bbs主页搭建

django admin后台管理
你只需要在对应的应用下的admin.py注册你想要操作的模型类即可
你需要创建一个超级用户才能够加入admin后台管理
createsuperuser
admin展示配置
在admin中有一些默认要求必填 你可以给字段加参数
blank=True
表名展示中文
class Meta:
verbose_name_plural = '表名'
对象展示注释信息
def __str__(self):
return self.name  # 该方法的返回值必须是字符串类型

admin扩展功能

```
class ArticleConfig(admin.ModelAdmin):
list_display = ['title','create_time']  # 配置展示字段
list_display_links = ['title','create_time']  # 指定多个跳转标签
search_fields = ['title']  # 指定查询  多个字段默认是or的关系

def patch_init(self,queryset):
pass
patch_init.short_description = '批量更新'
actions = [patch_init,]  # 自定义批量处理函数

list_filter = ['tags','category']  # 定义外键字段的过滤

```

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191111145335757-1318783305.png)

## 手动录入数据

用户头像展示
1.网站默认的静态文件资源统一放在static文件夹 
而用户上传的静态文件资源也应该单独放在某个文件夹

2.如何让用户上传的所有的静态文件全部自动放在某个文件夹下
配置文件中
MEDIR_ROOT = os.path.join(BASE_DIR,'某个文件夹的名字')
media
avatar 

## 用户头像文件

3.要想访问后端资源 必须实现开设该资源的接口
手动在urls.py中开设接口
from django.views.static import serve
from bbs import settings

url(r'^media/(?P<path>.*)',serve,{'document_root':settings.MEDIR_ROOT})
"""该方法能够做到暴露后端任何资源  在使用是要谨慎"""

个人站点
左侧的侧边栏前端样式渲染
三道orm查询题目
日期格式的分组查询
利用django 官网提供的方法

左侧的侧边栏筛选功能
url由三条优化成一条

### 筛选功能的本质 :就是对已经存在了的数据 再按照条件进行进一步的筛选

当你的url过多的时候可能会造成混乱 url被其他url拦截了
1.换位置
2.改正则表达式

多个路由公用一个视图函数
1.对视图函数的参数做扩展性设计

根据字典参数的不同对已经存在的queryset对象做进一步的筛选



# 文章详情页

如何再多个页面上使用同一个样式  并且改页面样式 是需要参数的

自定义inclusion_tag
与自定义过滤器 标签 准备工作是一模一样的
三步走
1.templatetags
2.任意名称py文件
3.两行代码

工作原理:类似于给函数传参并调用函数

拆分代码是很简单的  你只需要把代码先拷贝过去
然后哪个地方缺什么就补什么

评论
跟评论
页面渲染
临时渲染

子评论
用户点击回复按钮 发生了几件事
1.自动获取点击的那一条评论的评论人用户名   @人名
2.将拼接好的人名自动放入textarea框中 并且自动换行
3.textarea框自动聚焦



1.如何取出自带的@人名
2.当你回复了一个子评论之后 如果不刷新页面  后续的评论都会变成子评论

# 点赞点踩

1.直接参考博客园 拷贝前端点赞点踩样式
1.拷贝的时候 要靠完全了 html和css都要拷
2.图片防盗链 将需要用到的图片下载到本地
2.给点赞点踩图标绑定点击事件
1.为了区分用户到底是点击的赞还是踩
我们给赞和踩都设置了一个共同的类属性
然后给这个共同的类属性绑定点击事件
然后再利用赞和踩本身由各自独有的类属性
利用this变量指代的是当前被点击(操作)对象
再利用jQuery   hasClass()判断是否有对应的类属性
2.朝后端发ajax请求
由于处理点赞点踩的业务逻辑稍微有点复杂
推荐使用专门的视图函数进行业务逻辑处理
article_id
is_up
3.后端点赞点踩业务逻辑
1.先判断当前点赞点踩用户是否登录
request.user.is_authenticated()
2.判断当前文章是否是当前改用户自己写的
通过article_id获取用户想要点的文章对象
再利用文章对象获取对应的用户对象与当前登录用户对象request.user进行比对
3.判断当前用户是否已经给当前这篇文章点过赞或者点过踩了
利用article_obj和request.user直接去点赞点踩表中过滤数据
如果两者and查询有数据  说明当前用户已经给当前文章点过了
4.操作数据库  需要注意 有一个普通字段需要同步更新
is_up = json.loads(is_up)  # 转成后端python的布尔值类型
1.根据is_up决定到底是up_num加还是down_num加
2.直接操作点赞点踩表 完成记录的添加
5.再去补全错误的逻辑代码
"""
在写代码时候  建议遵循先把正确的逻辑全部写完之后 再去考虑错误的逻辑
"""

4.前端代码优化
用户点了赞或者踩 在结果一系列校验ok的情况下
你应该将点赞点踩的数字在原来的基础之上加一
在加的时候 千万注意 不要弄成了字符串的加   
利用Number转成数字再相加



# 评论

1.根评论

```
1.前端样式搭建
直接参考bbs书写或者拷贝即可
如何清除浮动带来的影响
clearfix

2.点击评论按钮  发送ajax请求
article_id
content

3.后端专门开设评论的视图函数  处理业务逻辑
直接获取数据
普通字段需要同步更新      利用事务
comment_num
Comment

4.展示评论
评论楼的渲染

5.当用户点击提交评论按钮
1.先将textarea清空
2.将用户评论的内容 先临时渲染到评论楼中
利用esc6新语法  模板字符串的替换
再利用标签查找及内部元素的添加  append()
```

2.子评论   
1.切入点就是回复按钮
点击回复按钮应该做的事
1.拼接人名   @人名\n
2.将拼接好的内容添加到textarea框中
3.将textarea聚焦     focus()
2.提交跟评论个提交子评论的区别
如何区分根评论和子评论
提交子评论的时候  需要拿到评论的主键值 ’
利用标签可以自定义任何属性的特点
给回复按钮 添加了用户名和主键值两个属性

```
$(this).attr('username')
$(this).attr('comment_id')
```

3.一个函数中的变量需要在林外一个函数中使用
最简便的方式就是将改变量定义成全局变量
1.定义全局的commend_id
2.后端parent字段可以为空的 也就意味着我在给parent参数传值的时候 可以传空

4.当提交一次子评论之后 需要将comment_id再次清空
还需要解决 用户提交子评论中    @人名\n

5.前端子评论的渲染
{{ comment.parent.user.username }}

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191111145332321-155288874.png)

# url连接合成

![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191107143744223-1396219487.png)

# 针对按日期分组报日期错误的情况

需要修改一下时区问题

```
from django.db.models.functions import TruncMonth

-官方提供
			from django.db.models.functions import TruncMonth
			Sales.objects
			.annotate(month=TruncMonth('timestamp'))  # Truncate to month and add to select list
			.values('month')  # Group By month
			.annotate(c=Count('id'))  # Select the count of the grouping
			.values('month', 'c')  # (might be redundant, haven't tested) select month and count
时区问题报错








```

```
TIME_ZONE = 'Asia/Shanghai'
USE_TZ = True

```

# 后台设计

## admin后台变中文

下面的配置改了admin后台变中文

```

LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True


```



![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191111145338491-2086375232.png)
![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191111145340989-1379870418.png)
![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191111145345113-1708614291.png)
![](https://img2018.cnblogs.com/blog/1739658/201911/1739658-20191111145351528-1772911546.png)

