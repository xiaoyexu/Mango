# Django
* Install Django on CentOS 7.0(use python 2.7)```wget https://bootstrap.pypa.io/get-pip.py
```如果wget没有，用``yum install wget``安装``python get-pip.py``或者用``easy_install pip``安装``pip install Django==1.7`` (version number)
>在ubuntu下安装pip，然后upgrade
>
```sudo apt-get install python-pip python-dev build-essentialsudo pip install --upgrade pip```
查看Django版本，如在python命令行输入以下命令：

```>>>import django>>>django.VERSION(1,4,0,'final',0)```
ubuntu下启动和终止脚本start.sh

```nohup sudo python manager.py runserver 0.0.0.0:80```
stop.sh

```pid=$(ps -ef | grep 'python manage' | awk '{print $2}')sudo kill -9 $pid```
**注意 Django项目文件，相关库都可能和python版本有关，从python2产生的项目可能会在python3上运行出错*** Django, database setting```DATABASES = {     'default':{          'ENGINE': 'django.db.backends.mysql',          'OPTIONS': {               'read_default_file': '/projects/my.cnf',          },     }}
```文件/projects/my.cnf

```#my.cnf#For msql connection[client]host = 192.168.1.101database = DJANGOuser = dbuserpassword = dbuserdefault-character-set = utf8character-set-server=utf8[mysql]default-character-set=utf8[mysqld]#default-character-set=utf8collation-server = utf8_unicode_cicharacter-set-server=utf8init_connect='SET NAMES utf8'```
或者直接指定数据库连接

```DATABASES = {     'default':{          'ENGINE': 'django.db.backends.mysql',          'NAME': '<mysql database name>',          'USER': '<mysql username>',          'PASSWORD': '<mysql user password>',     }}
```* MariaDB 中文字符问题file /projects/my.cnf contains content below: (must set utf8 for Chinese characters)
```## This group is read both both by the client and the server# use it for options that affect everything#[client-server]#default-character-set=utf8## include all files from the config directory#!includedir /etc/my.cnf.d#character_set_server=utf8#default-character-set=utf8[client]default-character-set=utf8[mysql]default-character-set=utf8[mysqld]#default-character-set=utf8collation-server = utf8_unicode_cicharacter-set-server=utf8init_connect='SET NAMES utf8'```* sqlitedb Unicode utf出错，由model的__unicode__方法引起的，可在models.py加入

```import sysreload(sys)sys.setdefaultencoding("utf-8")
```* Mac下安装MySQLdb
使用``easy_install MySQL-python``，注意大小写在/etc/profile中输出路径export DYLD_LIBRARY_PATH=/usr/local/mysql/lib，否则可能报如下错误

```django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module: dlopen(/Users/xuxiaoye/.python-eggs/MySQL_python-1.2.5-py2.7-macosx-10.10-intel.egg-tmp/_mysql.so, 2): Library not loaded: libmysqlclient.18.dylib  Referenced from: /Users/xuxiaoye/.python-eggs/MySQL_python-1.2.5-py2.7-macosx-10.10-intel.egg-tmp/_mysql.so  Reason: image not found```
* 查看django版本```python -c "import django; print(django.get_version())"
```* 在某个目录下创建mysite项目```django-admin startproject mysite```
配置好数据库，如果使用mysql，则需要先建好database(``create database <name>``)，在settings.py里配置好数据库连接信息在mysite创建app``python manage.py startapp myapp``
在mysite/settings.py添加myapp

```INSTALLED_APPS = (    'django.contrib.admin',    'django.contrib.auth',    'django.contrib.contenttypes',    'django.contrib.sessions',    'django.contrib.messages',    'django.contrib.staticfiles',    'myapp',)
```修改myapp/models.py后，用来生成sql文本```python manage.py makemigrations myapp
```>提示会产生诸如0001_initial.py sql文件用sqlmigrate查看更新数据库的sql语句，如果migrate出错，则可能需要手动修改数据库

```python manage.py sqlmigrate myapp 0001
```然后

```python manage.py migratepython manage.py migrate --fake 用来忽略已有冲突```Django shell``python manage.py shell``创建myapp管理员账户

```python manage.py createsuperuser
```使用服务器地址启动服务，开发模式，单线程

```python manage.py runserver 0.0.0.0:8000
```Djange 项目目录下（settings.py一级）的__init__.py指定了MySQLdb

```import pymysqlpymysql.install_as_MySQLdb()
```##Model```class Question(models.Model):    question_text = models.CharField(max_length=200)    pub_date = models.DateTimeField('date published')class Choice(models.Model):    question = models.ForeignKey(Question)    choice_text = models.CharField(max_length=200)    votes = models.IntegerField(default=0)```需要调用model的save()来保存数据```from django.db import modelsclass Book(models.Model):    title = models.CharField(max_length=100)    @classmethod    def create(cls, title):        book = cls(title=title)        # do something with the book        return bookbook = Book.create("Pride and Prejudice")```models.Manager是管理model的父类，即自定义Manager类```class BookManager(models.Manager):    def create_book(self, title):        book = self.create(title=title)        # do something with the book        return bookclass Book(models.Model):    title = models.CharField(max_length=100)    # 将自定义的Manager赋给objects，因为所有model都有.objects这个manager对象    objects = BookManager()book = Book.objects.create_book("Pride and Prejudice")```
class定义中加入ordering

```class Book(models.Model):    …    class Meta:        ordering = ['name']
```
则可使用``Book.objects.order_by(“-name”) ``排序
复合主键```class TeamMember(models.Model):        user = models.ForeignKey('User')        role = models.ForeignKey('ProjectRole')        class Meta:                unique_together = (('user','role'),)```
这张表里的user,projectrole不可重复

```MariaDB [django_test]> desc alltest_teammember;+---------+---------+------+-----+---------+----------------+| Field   | Type    | Null | Key | Default | Extra          |+---------+---------+------+-----+---------+----------------+| id      | int(11) | NO   | PRI | NULL    | auto_increment || role_id | int(11) | NO   | MUL | NULL    |                || user_id | int(11) | NO   | MUL | NULL    |                |+---------+---------+------+-----+---------+----------------+```
Meta中还可以用来指示是否为django管理object，比如``manged=False``，表示无需django做任何数据库修改，通常用在反向从已有数据库生成的对象上，下列表示该model对应的数据库表是table123，并且每次makemigrations，migrate时无需做任何操作```class Meta:    managed = False    db_table = 'table123'```自定义QuerySet在某个自定义Manager class中，返回一个已经过滤完的QuerySet```class PythonManager(models.Manager):    def get_query_set(self):        return super(PythonManager,self).get_query_set().filter(title__icontains="django")```
级联操作
```field = models.ForeignKey(‘Foo’,null=True,blank=True,on_delete=models.SET_NULL)
```表示当field对应的外键记录删除时，本model的field字段置空，否则会造成本model记录被删，默认为on_delete=CASCADE级联删除该属性还有其他值可选：```CASCADE: 默认的，级联删除PROTECT: 通过抛出django.db.models.ProtectedErrordjango.db.models.ProtectedError错误来阻止删除关联的对象 SET_NULL: 设置ForeignKey 为 null; 这个只有设置了null 为 True的情况才能用SET_DEFAULT: 设置 ForeignKey 为默认值; 默认值必须预先设置SET(): 设置为某个方法返回的值DO_NOTHING: 什么都不做，如果数据库设置必须关联则会报IntegrityError错。```
使用verbose_name和Meta来设置admin界面中的表字段显示值，比如

```class TeacherLevel(models.Model):    key = models.CharField(max_length=10, primary_key=True,verbose_name=u"主键")    description = models.CharField(max_length=255,verbose_name=u"描述")    def __unicode__(self):        return "%s %s" % (self.key, self.description)    class Meta:        verbose_name = u"教师级别定义"        verbose_name_plural = u"教师级别定义"
```##FieldsAutoField，自动增长的integerfield，通常不直接用BigIntegerField 64位长整型```class FieldTest(models.Model):        bigIntegerField = models.BigIntegerField()        binaryField = models.BinaryField(blank=True)        booleanField = models.BooleanField(default=False)        charField = models.CharField(max_length=20,default='')        commaSeparatedIntegerField = models.CommaSeparatedIntegerField(max_length=30,default='')        dateField = models.DateField(auto_now_add=True)        #dateField = models.DateField(auto_now=True,default=date.today)        dateTimeField = models.DateTimeField('Datetime:')        decimalField = models.DecimalField(max_digits=5,decimal_places=2)        durationField = models.DurationField()        emailField = models.EmailField()        fileField = models.FileField()        filePathField = models.FilePathField()        floatField = models.FloatField()        imageField = models.ImageField()        integerField = models.IntegerField()        ipAddressField = models.IPAddressField()        genericIPAddressField = models.GenericIPAddressField()        nullBooleanField = models.NullBooleanField()        positiveIntegerField = models.PositiveIntegerField()        positiveSmallIntegerField = models.PositiveSmallIntegerField()        slugField = models.SlugField()        smallIntegerField = models.SmallIntegerField()        textField = models.TextField()        timeField = models.TimeField()        urlField = models.URLField()        uuidField = models.UUIDField()```
对应表结构为

```MariaDB [django_test]> desc alltest_fieldtest;+----------------------------+----------------------+------+-----+---------+----------------+| Field                      | Type                 | Null | Key | Default | Extra          |+----------------------------+----------------------+------+-----+---------+----------------+| id                         | int(11)              | NO   | PRI | NULL    | auto_increment || bigIntegerField            | bigint(20)           | NO   |     | NULL    |                || binaryField                | longblob             | NO   |     | NULL    |                || booleanField               | tinyint(1)           | NO   |     | NULL    |                || charField                  | varchar(20)          | NO   |     | NULL    |                || commaSeparatedIntegerField | varchar(30)          | NO   |     | NULL    |                || dateField                  | date                 | NO   |     | NULL    |                || dateTimeField              | datetime             | NO   |     | NULL    |                || decimalField               | decimal(5,2)         | NO   |     | NULL    |                || durationField              | bigint(20)           | NO   |     | NULL    |                || emailField                 | varchar(254)         | NO   |     | NULL    |                || fileField                  | varchar(100)         | NO   |     | NULL    |                || filePathField              | varchar(100)         | NO   |     | NULL    |                || floatField                 | double               | NO   |     | NULL    |                || integerField               | int(11)              | NO   |     | NULL    |                || ipAddressField             | char(15)             | NO   |     | NULL    |                || genericIPAddressField      | char(39)             | NO   |     | NULL    |                || nullBooleanField           | tinyint(1)           | YES  |     | NULL    |                || positiveIntegerField       | int(10) unsigned     | NO   |     | NULL    |                || positiveSmallIntegerField  | smallint(5) unsigned | NO   |     | NULL    |                || slugField                  | varchar(50)          | NO   | MUL | NULL    |                || smallIntegerField          | smallint(6)          | NO   |     | NULL    |                || textField                  | longtext             | NO   |     | NULL    |                || timeField                  | time                 | NO   |     | NULL    |                || urlField                   | varchar(200)         | NO   |     | NULL    |                || uuidField                  | char(32)             | NO   |     | NULL    |                |+----------------------------+----------------------+------+-----+---------+----------------+```
imgField需要Pillow支持，需根据版本安装必要的包，如ubuntu 14.04上

```apt-get install libtiff5-dev libjpeg8-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python-tk
```再安装

```
pip install Pillow```
>所有migrations记录会在表django_migrations中，可以清空这张表和文件目录里的migrations目录重新建表，但是其他已存在表可能会报错
>
```truncate table django_migrations```
如果表结构发生改动，导致以下错误，这是因为已有的字段没有默认值

```You are trying to add a non-nullable field 'dateField2' to fieldtest without a default; we can't do that (the database needs something to populate existing rows).Please select a fix: 1) Provide a one-off default now (will be set on all existing rows) 2) Quit, and let me add a default in models.pySelect an option:
```
此时可以选1，输入默认值，比如dateField2是DateField类型，输入``timezone.now()``，将一次性更新表中原有记录的字段为默认值

```Please select a valid option: 1Please enter the default value now, as valid PythonThe datetime and django.utils.timezone modules are available, so you can do e.g. timezone.now()>>> timezone.now()
Migrations for 'alltest':  0002_fieldtest_datefield2.py:    - Add field dateField2 to fieldtest
```会产生migration sql文件，内容如下

```# -*- coding: utf-8 -*-from __future__ import unicode_literalsfrom django.db import models, migrationsimport datetimefrom django.utils.timezone import utcclass Migration(migrations.Migration):    dependencies = [        ('alltest', '0001_initial'),    ]    operations = [        migrations.AddField(            model_name='fieldtest',            name='dateField2',            field=models.DateField(default=datetime.datetime(2015, 5, 11, 5, 56, 57, 954016, tzinf   utc), auto_now=True),            preserve_default=False,        ),    ]```
字段dateField2会被添加到最末

```| uuidField                  | char(32)             | NO   |     | NULL    |                || dateField2                 | date                 | NO   |     | NULL    |                |+----------------------------+----------------------+------+-----+---------+----------------+```
如需要字段可为空，构造时使用null=True，比如```charField = models.CharField(max_length=20,default='',null=True)```

对应表结构
```+----------------------------+----------------------+------+-----+---------+----------------+| Field                      | Type                 | Null | Key | Default | Extra          |+----------------------------+----------------------+------+-----+---------+----------------+| charField                  | varchar(20)          | YES  |     | NULL    |                ```
##Relationship1.	ForeignKey2.	OneToOneField3.	ManyToManyField多个学生有一个老师，多对一

```class Student(models.Model):        name = models.CharField(max_length=50)        teacher = models.ForeignKey('Teacher')class Teacher(models.Model):        name = models.CharField(max_length=50)```
多个小组有一个父小组，多对一，自身多个小组有多个人，多个人参加多个小组一个小组只能有一个老师

```class Group(models.Model):        name = models.CharField(max_length=50)        parentGroup = models.ForeignKey('self',blank=True,null=True)        students = models.ManyToManyField('Student')        teacher = models.OneToOneField('Teacher')```
多对多关系操作，对group来说直接访问students变量

```g.students.all()g.students.add(stu2,stu3)g.students.count()g.students.remove(stu2)
```
对Student来说，访问group_set变量，_set是默认后缀

```stu2.group_set.all() 
```
如果多对多关系应用于自身，则是对称的，可以加入参数取消对称性，通过<model>_set访问

```models.ManyToManyField("self",symmetrical=False)```
反向名称，默认为<model>_set，比如Student查老师为stu.teacher，老师查学生为teacher.student_set，可以指定反向名字，比如

```pub=models.ForeignKey(Publisher,related_name='pub')    authors=models.ManyToManyField(Author,related_name='author')
```
如果不想设置反向关系，设置``related_name``为``+``或者以``+``结束。

```user = models.ForeignKey(User, related_name='+')
```
如果有多个ManyToManyField指向同一个Model,这样反向查询FOO_set的时候就无法弄清是哪个ManyToManyField字段了,可以禁止反向关系:

```users = models.ManyToManyField(User, related_name='u+')referents = models.ManyToManyField(User, related_name='ref+')```
>多对多关系实际上是由django自动生成第三张表，A,B，生成C(id,A.id,B.id)ForeignKey,多对一关系，如是对自身，使用

```
models.ForeignKey(‘self’)
```

在views.py中引入

```from django.db import models```
如果model已经定义，可以直接使用object，Manufacturer,或者对还没定义的model，使用引号的版本

```class Car(models.Model):    manufacturer = models.ForeignKey('Manufacturer')    # ...class Manufacturer(models.Model):    # ...    pass
```可以引用其他app下的model，比如production下的Manufacturer

```class Car(models.Model):    manufacturer = models.ForeignKey('production.Manufacturer')```
带限制的ForeignKey，只能选is_staff为True的User

```staff_member = models.ForeignKey(User, limit_choices_to={'is_staff': True})
```
学生可选教师范围在age > 30

```teacher = models.ForeignKey(Teacher,limit_choices_to=Q(age__gt=30))```
建完model可在``python manager.py shell``里直接引用操作Model如：

```class Teacher(models.Model):        name = models.CharField(max_length=50)class Student(models.Model):        teacher = models.ForeignKey(Teacher)        name = models.CharField(max_length=50)```
shell中操作如

```>>> Teacher.objects.all()[]>>> t = Teacher('Mario')>>> t.nameu''>>> t = Teacher(name = 'Mario')           #要指定参数名>>> t.name'Mario'>>> s = Student(name ='stu')>>> s.name'stu'>>> t.save()>>> s.save()       #报错，因为s.teacher未指定>>> s.teacher = t   #成功>>> s.save()```

数据库里

```MariaDB [django_test]> select * from alltest_student;+----+------+------------+| id | name | teacher_id |+----+------+------------+|  1 | stu  |          1 |+----+------+------------+1 row in set (0.00 sec)MariaDB [django_test]> select * from alltest_teacher;+----+-------+| id | name  |+----+-------+|  1 | Mario |+----+-------+```
默认model构造方法（无参数）

```>>> from alltest.models import *>>> t = Teacher()>>> t.name = 'Han'>>> t.save()>>> s = Student()>>> s.name = 'stu2'>>> s.teacher = Teacher.objects.all()[0]>>> s.save()```使用filter查找，返回集合

```Teacher.objects.filter(id=1)[0].name #返回集合，故取第0个```
使用get查找，只返回一条，否则报错

```Teacher.objects.get(id=1).name #返回一个```count()计数

```>>> Teacher.objects.count()3```
delete()删除```>>> Teacher.objects.get(id=3).delete()>>> Teacher.objects.count()2```##Admin界面python manage.py createsuperuser，创建网页的admin登录用户重置密码，在shell里运行

```from django.contrib.auth.models import User user = User.objects.get(username='admin') user.set_password('new_password') user.save()```并且在admin.py中注册models，比如

```from django.contrib import adminfrom .models import Questionadmin.site.register(Question)```
然后运行runserver自定义Admin界面，admin.py, 调整字段显示顺序

```class TeacherCustAdmin(admin.ModelAdmin):        fields = ['age','name']#admin.site.register(Teacher)admin.site.register(Teacher,TeacherCustAdmin)```
或者把字段分组group```class TeacherCustAdmin(admin.ModelAdmin):        #fields = ['age','name']        fieldsets = [        (None,               {'fields': ['age']}),        ('Name of Teacher', {'fields': ['name']}),        ]```
指定字段是否可扩展

```('Name of Teacher', {'fields': ['name'], 'classes': ['collapse']}),```
列表显示，title是Teacher中的方法models.py

```class Teacher(models.Model):        name = models.CharField(max_length=50)        age = models.IntegerField()        def title(self):                if self.age > 30 :                        return 'Old teacher'                else:                        return 'Young teacher'```
admin.py:

```class TeacherCustAdmin(admin.ModelAdmin):        list_display = ('name','age','title')        #fields = ['age','name']        fieldsets = [        (None,               {'fields': ['age']}),        ('Name of Teacher', {'fields': ['name'],'classes': ['collapse']}),        ]```同样，search_fields指定可搜索字段

```search_fields = ('name','age','title')list_filter 指定可过滤字段，是tuple类型(‘name’,)date_hierarchy 是string类型 date_hierarchy = ‘datefield’ordering 指定排序 ordering = (‘-datefield’,)fields指定要显示的字段 fields = (‘field1’,’field2’)filter_horizontal/filter_vertical产生一个左右/水平式的过滤器 filter_horizontal = (‘field1’,)，只适用于多对多字段raw_id_fields字段搜索框，raw_id_fields =(‘field1‘,)```##Viewapp的目录下，view.py中定义的变量为全局Application级别变量View.py:

```from django.shortcuts import renderfrom django.http import HttpResponse# Create your views here.def index(request):        output = 'Default page'        return HttpResponse(output)```
同级目录下创建urls.py```from django.conf.urls import urlfrom . import viewsurlpatterns = [    url(r'^index$', views.index, name='index'),]```
并且在project目录，比如mysite下的urls.py中加入

```url(r'^alltest/', include('alltest.urls'))
```>意为符合正则^alltest/的url请求通过alltest下的urls.py解析**所有的form 提交必须要有{%csrf_token%}**，如需禁用csrf检查，在views.py中使用

```from djnago.views.decorators.csrf import csrf_exempt…@csrf_exemptdef nocsrf(request):	…	…
```	
>这样post url http://domain/nocsrf将不会作csrf检查HttpRequest>当请求一张页面时，Django把请求的metadata数据包装成一个HttpRequest对象，然后Django加载合适的view方法,把这个HttpRequest 对象作为第一个参数传给view方法。任何view方法都应该返回一个HttpResponse对象。HttpRequest代表一个来自uesr-agent的HTTP请求。请求中使用的HTTP方法的字符串表示。全大写表示。例如:```if request.method == 'GET':    do_something()elif request.method == 'POST':    do_something_else()
```GET>包含所有HTTP GET参数的类字典对象。参见QueryDict 文档。POST>包含所有HTTP POST参数的类字典对象。参见QueryDict 文档。服务器收到空的POST请求的情况也是有可能发生的。也就是说，表单form通过HTTP POST方法提交请求，但是表单中可以没有数据。因此，不能使用语句if request.POST来判断是否使用HTTP POST方法；应该使用if request.method == "POST" (参见本表的method属性)。注意: POST不包括file-upload信息。参见FILES属性。REQUEST>为了方便，该属性是POST和GET属性的集合体，但是有特殊性，先查找POST属性，然后再查找GET属性。借鉴``PHP’s $_REQUEST``。例如，如果GET = {"name": "john"} 和POST = {"age": '34'},则 REQUEST["name"] 的值是"john", REQUEST["age"]的值是"34".强烈建议使用GET and POST,因为这两个属性更加显式化，写出的代码也更易理解。COOKIES>包含所有cookies的标准Python字典对象。Keys和values都是字符串。参见第12章，有关于cookies更详细的讲解。FILES>包含所有上传文件的类字典对象。FILES中的每个Key都是``<input type="file" name="" />``标签中name属性的值. FILES中的每个value 同时也是一个标准Python字典对象，包含下面三个Keys:filename: 上传文件名,用Python字符串表示content-type: 上传文件的Content typecontent: 上传文件的原始内容注意：只有在请求方法是POST，并且请求页面中<form>有enctype="multipart/form-data"属性时FILES才拥有数据。否则，FILES 是一个空字典。META>包含所有可用HTTP头部信息的字典。 例如:

```CONTENT_LENGTHCONTENT_TYPEQUERY_STRING: 未解析的原始查询字符串REMOTE_ADDR: 客户端IP地址REMOTE_HOST: 客户端主机名SERVER_NAME: 服务器主机名SERVER_PORT: 服务器端口META 中这些头加上前缀HTTP_最为Key, 例如:HTTP_ACCEPT_ENCODINGHTTP_ACCEPT_LANGUAGEHTTP_HOST: 客户发送的HTTP主机头信息HTTP_REFERER: referring页HTTP_USER_AGENT: 客户端的user-agent字符串HTTP_X_BENDER: X-Bender头信息```
user>是一个django.contrib.auth.models.User 对象，代表当前登录的用户。如果访问用户当前没有登录，user将被初始化为django.contrib.auth.models.AnonymousUser的实例。你可以通过user的is_authenticated()方法来辨别用户是否登录：

```if request.user.is_authenticated():    # Do something for logged-in users.else:    # Do something for anonymous users.
```>只有激活Django中的AuthenticationMiddleware时该属性才可用
关于认证和用户的更详细讲解,参见第12章。session>唯一可读写的属性，代表当前会话的字典对象。只有激活Django中的session支持时该属性才可用。 参见第12章。raw\_post\_data>原始HTTP POST数据，未解析过。 高级处理时会有用处。Request对象也有一些有用的方法,见Table H-2： Table H-2. HttpRequest Methods
Method|Description
---|---\_\_getitem\_\_(key)|返回GET/POST的键值,先取POST,后取GET。如果键不存在抛出 KeyError。这是我们可以使用字典语法访问HttpRequest对象。例如,request["foo"]等同于request.POST["foo"] 然后 request.GET["foo"]的操作。
has\_key()|检查request.GET or request.POST中是否包含参数指定的Key。
get\_full\_path()|返回包含查询字符串的请求路径。例如， "/music/bands/the_beatles/?print=true"is\_secure()|如果请求是安全的，返回True，就是说，发出的是HTTPS请求。 QueryDict对象>在HttpRequest对象中, GET和POST属性是django.http.QueryDict类的实例。 QueryDict类似字典的自定义类，用来处理单键对应多值的情况。因为一些HTML form元素，例如，<selectmultiple="multiple">, 就会传多个值给单个键。QueryDict对象是immutable(不可更改的),除非创建它们的copy()。这意味着我们不能直接改变request.POST and request.GET的属性。QueryDict实现所有标准的字典方法。还包括一些特有的方法，见Table H-3.Table H-3. How QueryDicts Differ from Standard Dictionaries.
Method|Differences from Standard dict Implementation
---|---\_\_getitem\_\_|和标准字典的处理有一点不同，就是，如果Key对应多个Value，__getitem__()返回最后一个value。\_\_setitem\_\_|设置参数指定key的value列表(一个Python list)。注意：它只能在一个mutable QueryDict 对象上被调用(就是通过copy()产生的一个QueryDict对象的拷贝).get()|如果key对应多个value，get()返回最后一个value。update()|参数可以是QueryDict，也可以是标准字典。和标准字典的update方法不同，该方法添加字典 items，而不是替换它们:
items()|和标准字典的items()方法有一点不同,该方法使用单值逻辑的\_\_getitem\_\_():
values()|和标准字典的values()方法有一点不同,该方法使用单值逻辑的\_\_getitem\_\_():

update() 例子

```>>> q = QueryDict('a=1')>>> q = q.copy() # to make it mutable>>> q.update({'a': '2'})>>> q.getlist('a') ['1', '2']>>> q['a'] # returns the last['2']
```

items() 例子

```>>> q = QueryDict('a=1&a=2&a=3')>>> q.items()[('a', '3')]
```此外, QueryDict也有一些方法，见Table H-4. H-4. 额外的 (非字典的) QueryDict 方法
Method|Description
---|---copy()|返回对象的拷贝，内部实现是用Python标准库的copy.deepcopy()。该拷贝是mutable(可更改的)— 就是说，可以更改该拷贝的值。getlist(key)|返回和参数key对应的所有值，作为一个Python list返回。如果key不存在，则返回空list。It’s guaranteed to return a list of some sort.setlist(key,list\_)|设置key的值为list\_ (unlike \_\_setitem\_\_()).appendlist(key,item)|添加item到和key关联的内部list.setlistdefault(key,list)|和setdefault有一点不同，它接受list而不是单个value作为参数。lists()|和items()有一点不同, 它会返回key的所有值，作为一个list
urlencode()|返回一个以查询字符串格式进行格式化后的字符串(e.g., "a=2&b=3&b=5").

lists() 例子

```>>> q = QueryDict('a=1&a=2&a=3')>>> q.lists()[('a', ['1', '2', '3'])]
```
A Complete Example例如, 下面是一个HTML form:

```<form action="/foo/bar/" method="post"><input type="text" name="your_name" /><select multiple="multiple" name="bands">    <option value="beatles">The Beatles</option>    <option value="who">The Who</option>    <option value="zombies">The Zombies</option></select><input type="submit" /></form>
```
如果用户在your_name域中输入"JohnSmith"，同时在多选框中选择了“The Beatles”和“The Zombies”，下面是Django请求对象的内容:

```>>> request.GET{}>>> request.POST{'your_name': ['John Smith'], 'bands': ['beatles', 'zombies']}>>> request.POST['your_name']'John Smith' >>> request.POST['bands']'zombies'>>> request.POST.getlist('bands')['beatles', 'zombies']>>> request.POST.get('your_name', 'Adrian')'John Smith'>>> request.POST.get('nonexistent_field', 'Nowhere Man')'Nowhere Man'
```
HttpResponse>对于HttpRequest 对象来说，是由Django自动创建, 但是，HttpResponse对象就必须我们自己创建。每个View方法必须返回一个HttpResponse对象。>HttpResponse类在django.http.HttpResponse。构造HttpResponses一般地, 你可以通过给HttpResponse的构造函数传递字符串表示的页面内容来构造HttpResponse对象:```>>> response = HttpResponse("Here's the text of the Web page.")>>> response = HttpResponse("Text only, please.", mimetype="text/plain")```
但是如果想要增量添加内容, 你可以把response当作filelike对象使用:```>>> response = HttpResponse()>>> response.write("<p>Here's the text of the Web page.</p>")>>> response.write("<p>Here's another paragraph.</p>")```也可以给HttpResponse传递一个iterator作为参数，而不用传递硬编码字符串。如果你使用这种技术, 下面是需要注意的一些事项:·         iterator应该返回字符串.·         如果HttpResponse使用iterator进行初始化，就不能把HttpResponse实例作为filelike 对象使用。这样做将会抛出异常。最后，再说明一下，HttpResponse实现了write()方法, 可以在任何需要filelike对象的地方使用HttpResponse对象。
设置Headers你可以使用字典语法添加，删除headers:```>>> response = HttpResponse() >>> response['X-DJANGO'] = "It's the best.">>> del response['X-PHP']>>> response['X-DJANGO']"It's the best."
```
你也可以使用has_header(header)方法检测某个header是否存在。不要手动设置Cookie headers
HttpResponse子类>Django包含很多HttpResponse子类，用来处理不同的HTTP响应类型(见Table H-5). 和HttpResponse一样, 这些子类在django.http中.Table H-5. HttpResponse SubclassesClass|Description
---|---HttpResponseRedirect|构造函数接受单个参数：重定向到的URL。可以是全``URL (e.g., 'http://search.yahoo.com/')``或者相对``URL(e.g., '/search/')``. 注意：这将返回HTTP状态码302。HttpResponsePermanentRedirect|同HttpResponseRedirect一样，但是返回永久重定向(HTTP 状态码 301)。HttpResponseNotModified|构造函数不需要参数。Use this to designate that a page hasn’t been modified since the user’s last request.HttpResponseBadRequest|返回400 status code。HttpResponseNotFound|返回404 status code.HttpResponseForbidden|返回403 status code.HttpResponseNotAllowed|返回405 status code. 它需要一个必须的参数：一个允许的方法的list (e.g., ['GET','POST']).HttpResponseGone|返回410 status code.HttpResponseServerError|返回500 status code. 当然，你也可以自己定义不包含在上表中的HttpResponse子类。返回Errors在Django中返回HTTP错误码是很容易的。上面介绍了HttpResponseNotFound, HttpResponseForbidden, HttpResponseServerError等一些子类。View方法中返回这些子类的实例就OK了,例如:

```def my_view(request):        #  ...       if foo:                return HttpResponseNotFound('<h1>Page not found</h1>')        else:                 return HttpResponse('<h1>Page was found</h1>')```>当返回HttpResponseNotFound时,你需要定义错误页面的HTML:
```
return HttpResponseNotFound('<h1>Page not found</h1>')
```因为404错误是最常使用的HTTP错误, 有一个更方便的方法处理它。为了方便，而且整个站点有一致的404错误页面也是友好的,Django提供一个Http404异常类。如果在一个View方法中抛出Http404,Django将会捕获它并且返回标准错误页面,同时返回错误码404。```from django.http import Http404def detail(request, poll_id):        try:                p = Poll.objects.get(pk=poll_id)        except Poll.DoesNotExist:                raise Http404        return render_to_response('polls/detail.html', {'poll': p})
```
    为了使用Http404异常, 你需要创建一个模板文件。当异常抛出时，就会显示该模板文件。该模板文件的文件名是404.html,在模板根目录下创建。
自定义404 (Not Found) View方法当抛出Http404异常时, Django会加载一个特殊的view方法处理404错误。默认地, 它是django.views.defaults.page\_not\_found,负责加载和渲染404.html模板文件。这意味着我们必须在模板根目录定义404.html模板文件，该模板文件应用于所有的404异常。该page\_not\_found view方法应该可以应对几乎99%的Web App，但是如果想要重载该view方法时, 你可以在URLConf文件中指定handler404为自定义的404 error view方法, 像这样:

```from django.conf.urls.defaults import *urlpatterns = patterns('',        ...)handler404 = 'mysite.views.my_custom_404_view'
```
Django就是通过在URLConf文件中查找handler404来决定404 view方法的。默认地, URLconfs包含下面一行代码:

```from django.conf.urls.defaults import *
```
在 django/conf/urls/defaults.py模块中, handler404赋值为 'django.views.defaults.page\_not\_found'。**注意:**· 如果请求URL没有和Django的URLConf文件中的任何一个正则表示式匹配，404 view方法就会被调用。· 如果没有定义自己的404 view — 使用默认地404 view，你仍然有一个工作要做：在模板根目录创建404.html模板文件。默认地404 view将使用该模板处理所有404错误。· 如果 DEBUG设置为True (在setting模块中)，404 view方法不会被使用，取而代之的是，traceback信息。
自定义500 (Server Error) View方法同样地，当view代码发生运行时错误时，Django也会产生特殊行为。如果view方法产生异常，Django将会调用默认地view方法django.views.defaults.server\_error, 该方法加载渲染500.html模板文件。这意味着我们必须在模板根目录定义500.html模板文件，该模板文件应用于所有的服务器错误。该server\_error view方法应该可以应对几乎99%的Web App，但是如果想要重载该view方法时, 你可以在URLConf文件中指定handler500为自定义的server error view方法, 像这样:

```from django.conf.urls.defaults import *   urlpatterns = patterns('',            ...   )    handler500 = 'mysite.views.my_custom_error_view'```Template在View.py中使用template

```def index(request):        output = 'Default page'        template = loader.get_template('alltest/index.html')        context = RequestContext(request,        {        'str':'String'        })        return HttpResponse(template.render(context))
```
此时必须在view的同级目录下有templates目录，其中有alltest/index.html文件在工程的settings.py中指定目录，表示从当前项目目录manage.py所在目录，及app目录（view.py所在目录）templates下找，也就是template可在manage.py同一层，或者在alltest下

```TEMPLATES = [    {        'BACKEND': 'django.template.backends.django.DjangoTemplates',        'DIRS': [os.path.join(BASE_DIR, 'templates')],        'APP_DIRS': True,        'OPTIONS': {            'context_processors': [                'django.template.context_processors.debug',                'django.template.context_processors.request',                'django.contrib.auth.context_processors.auth',                'django.contrib.messages.context_processors.messages',            ],        },    },]```
当然可以在其中指定其他目录

```'DIRS': [os.path.join(BASE_DIR, 'templates'), '/var/www/html/'],```此时view中若``loader.get_template('index.html')``将获得``/var/www/html/``下的``index.html``如果目录结构相同，看起来/var/www/html/的优先级高>{%%} 为表达式{{}}为值

```{% if latest_question_list %}<ul>    {% for question in latest_question_list %}<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>    {% endfor %}</ul>{% else %}<p>No polls are available.</p>{% endif %}```
view.py中返回404错，如

```try:    question = Question.objects.get(pk=question_id)except Question.DoesNotExist:    raise Http404("Question does not exist")
```或者简洁方式```question = get_object_or_404(Question, pk=question_id)
```
同样有get_list_or_404() url，这里的index是view.py里的方法

```<a href="{% url 'index' %}">{{ str }}</a>
```如果url需要带参数，showurl是view里的name，’abc’为参数

```<a href="{% url 'showurl' 'abc'%}">{{ str }}</a>
```
此时show接受两个参数

```def show(request,parameter):    output = 'Parameter is %s' % parameter    return HttpResponse(output)
```
参数的定义在urls.py中

```url(r'^show/(?P<parameter>.*)$', views.show, name='showurl'),```这里表示符合…/show/abc的url，abc为参数存放在parameter变量中又如
```url(r'^show/(?P<parameter>.*)/end$', views.show, name='showurl'),http://192.168.56.102/alltest/show/134/end```
可以指明url全名，比如```<a href="{% url 'alltest:showurl' 'abc'%}">{{ str }}</a>```
页面继承可使用block，比如```<td>{% block content %}{% endblock %}</td>```
子html使用``{% extends "framework/layout/layout.html" %}``，并用替换父页面的内容

```{% block content %}{{ block.super }}Anything here{% endblock %}```
``block.super``表示父页面中已有的blockinclude只会把渲染好的页面包进来，因此父页面include的文件里的block是无法被子页面使用的直接将url map到html文件，不写views逻辑

```fromdjango.conf.urlsimport urlfromdjango.views.genericimport TemplateViewurlpatterns = [    url(r'^about/', TemplateView.as_view(template_name="about.html")),]```或者写一个继承于TemplateView的类

```# some_app/views.pyfrom django.views.generic import TemplateViewclass AboutView(TemplateView):    template_name ="about.html"
```然后在urls中指定```# urls.pyfrom django.conf.urls import urlfrom some_app.views import AboutViewurlpatterns = [    url(r'^about/', AboutView.as_view()),]```
根据需要修改http response header```from django.conf.urls import urlfrom books.views import BookListViewurlpatterns = [    url(r'^books/$', BookListView.as_view()),]```
这里修改了Last-Modified,使得book在未更新情况下，客户端无需多次访问服务器

```from django.http import HttpResponsefrom django.views.generic import ListViewfrom books.models import Bookclass BookListView(ListView):    model = Bookdef head(self, *args, **kwargs):        last_book =self.get_queryset().latest('publication_date')        response = HttpResponse('')# RFC 1123 date format        response['Last-Modified'] = last_book.publication_date.strftime('%a, %d %b %Y %H:%M:%S GMT')return response```##Tag在app目录下（models.py, views.py）建``templatetags``目录，必须为这个名字，在其中建Tag py文件，比如``TestTag.py``

```from django import templateregister = template.Library()class TestTagOne(template.Node):        def __init__(self, nav_path, nav_displaytext):                self.path = nav_path.strip('"')                self.text = nav_displaytext.strip('"')        def render(self, context):                output = '<a href="%s">%s</a><br>' % (self.path, self.text)                return output@register.tag(name='TestTagOnPage')    #注册Tag，使用名是TestTagOnPage，处理方法是testTagOnedef testTagOne(parse, token):        try:                tag_name, nav_path, nav_text = token.split_contents()        except ValueError:                raise template.TemplateSyntaxError, \                "%r tag requires exactly two arguments: path and text" % \                token.split_contents[0]        return TestTagOne(nav_path, nav_text)   #返回Tag对象`````token.split_contents()``会返回页面上所有字符，包括引号，所以用``strip``去掉引号，或者数组``[1:-1]``来去掉首尾字符，``token.split_contents()[1][1:-1]``templatetags下touch一个空的``__init__.py``以表示该目录为python包目录```{% load TestTag %}  这个TestTag是py文件名[{% TestTagOnPage  / "Menu1" %}]  TestTagOnPage是注册名字```
带头尾Tag比如 FormTag EndFormTag```{% FormTag %}…{% EndFormTag %}
```
FormTag.py

```from django import templateregister = template.Library()class FormTag(template.Node):	def __init__(self, nodeList, form_name, form_action):		self.nodeList = nodeList		self.name = form_name.strip('"')		self.action = form_action.strip('"')	def render(self, context):		innerContent = self.nodeList.render(context)		output = """		<form name="%s" action="%s" method="POST"><input id="navForm_pageAction" type="hidden" name="pageAction" value=""><input id="navForm_pageParams" type="hidden" name="pageParams" value="">%s</form>""" % (self.name, self.action, innerContent)		return output@register.tag(name='FormTag')def formTag(parse, token):	nodeList = parse.parse(('EndFormTag',))	parse.delete_first_token()	try:		tag_name, form_name, form_action = token.split_contents()	except ValueError:		raise template.TemplateSyntaxError, \		"%r tag requires exactly 2 arguments: path and text" % \		token.split_contents[0]	return FormTag(nodeList,form_name,form_action)```
需要注意的是```nodeList = parse.parse(('EndFormTag',))parse.delete_first_token()```
过滤器，可不带class，用reigster.filter替代register.tag，比如

```from django import templateregister = template.Library()@register.filter(name="cut")def myCut(value,arg):    return value.replace(arg,"!")```用法：

```{{value | cut 'o'}}```
带参数解析的tag，比如``{% Tag abc %}``，此时希望abc是一个变量，可以这样做在tag处理方法里，用``parse.compile_filter``来编译参数，返回一个``django.template.base.FilterExpression``类型的对象```tag_name, appId, phraseId = token.split_contents()phraseId = parse.compile_filter(phraseId)  #返回的phraseId类型不是str或unicode```
在Tag类的render方法里，判断类型，用.resolve(context)方法来还原具体值，要注意的是在render方法里，**不能用还原后的值修改成员变量，否则会造成循环体中的第二个tag无法解析，因为第一次被解析后这个self.name被覆盖了**

```phraseId = parseFilter(self.phraseId,context)return getPhrase(context['request'],self.appId,phraseId)
```比如，如果已经是str或unicode字符串，直接返回，否则使用resolve方法解析，如果解析为空，则直接返回原值（保证诸如`` {% Tag abc %}``在``abc``没有定义情况下，为字符串``'abc'``）

```def parseFilter(filter,context):        if type(filter) == str or type(filter) == unicode:                v = filter        else:                v = filter.resolve(context)                if v == '':                        v = filter        return v```
Django Tag

```{% for item in todo_list %}<p>{{ forloop.counter }}: {{ item }}</p>{% endfor %}forloop.counter0forloop.revcounter forloop.revcounter0 {% for object in objects %}    {% if forloop.first %}<li class="first">{% else %}<li>{% endif %}    {{ object }}</li>{% endfor %}{% for link in links %}{{ link }}{% if not forloop.last %} | {% endif %}{% endfor %}{% ifequal user currentuser %}<h1>Welcome!</h1>{% endifequal %}{% ifequal section 'sitenews' %}<h1>Site News</h1>{% endifequal %}{% ifequal section "community" %}<h1>Community</h1>{% endifequal %}```##FormAttribute/method|Description|Example
---|---|---request.path|The full path, not including the domain but including the leading slash.|"/hello/"request.get_host()|The host (i.e., the “domain,” in common parlance).|"127.0.0.1:8000" or"www.example.com"request.get_full_path()|The path, plus a query string (if available).|"/hello/?print=true"request.is_secure()|True if the request was made via HTTPS. Otherwise, False.|True or False

```<h1>{{question.question_text}}</h1>{%iferror_message%}<p><strong>{{error_message}}</strong></p>{%endif%}<formaction="{%url'polls:vote'question.id%}"method="post">{%csrf_token%}{%forchoiceinquestion.choice_set.all%}<inputtype="radio"name="choice"id="choice{{forloop.counter}}"value="{{choice.id}}"/><labelfor="choice{{forloop.counter}}">{{choice.choice_text}}</label><br/>{%endfor%}<inputtype="submit"value="Vote"/></form>```
在提交端获得提交内容``request.POST['choice']``如果choice不存在会报错用``request.POST.get(‘key’,’default value’)``来获得数据，给出键不存在时的默认值对于``http://domain/test/?query=123``，用``request.GET[‘query’]``，``request.GET.get(‘query’,None)``获取``{%csrf_token%}``将在form内生成类似隐藏字段``<input type='hidden' name='csrfmiddlewaretoken' value='aa5PrILWccDV7aAyedZurkTqTn1HrSHD' />``使用django formmodels.py

```from django.forms import ModelForm, Form# Formclass TestForm(Form):        text = forms.CharField(max_length=100,label='Text:')# Model Formclass TestModelForm(ModelForm):        class Meta:                model = Settings                fields = ('propertyName','propertyValue')```使用form，views.py:

```def formsubmit(request):        if request.method == 'POST':                #form = TestModelForm(request.POST)                form =TestForm(request.POST)                if form.is_valid():                        return HttpResponseRedirect('/alltest/home')        else:                #form = TestModelForm()                form = TestForm()        #return render_to_response('alltest/formtest.html',{'form':form})        template = loader.get_template('alltest/formtest.html')        context = RequestContext(request,{        'form':form        })        return HttpResponse(template.render(context))```注意因为form需要``csrf_token``，用``render_to_response``则``{%csrf_token%}``无法生成隐藏字段，即必须用``tempload loader``和``context``，以下无效

```return render_to_response('alltest/formtest.html', {                'form': form,        })```
##Statics在settings.py中指定static目录，比如```STATIC_URL = '/static_cust/'```在html中使用``{% load staticfiles %}``及 static标签``{% static "test/test.jpg" %}``如
```
<img src="{% static "test/test.jpg" %}" alt="My image">
```将会生成url

```<img src="/static_cust/test/test.jpg" alt="My image">```
在settings中指定static查找目录，比如``static_cust``，``BASE_DIR``是有``manager.py``的那一层，如果如下指定

```STATICFILES_DIRS = (    os.path.join(BASE_DIR, "static_cust"),)
```则``static_cust``和``mysite``,``manage.py``,``alltest``同一层如果用include包进来的文件，都需要用 ``load staticfileds`` 加载标签##Logging在Settings.py中配置，一个logger可以有多个handler

```LOGGING = {    'version': 1,    'disable_existing_loggers': True,    'formatters': {        'standard':{            'format': '%(levelname)s %(asctime)s %(message)s'        }    },    'filters': {},    'handlers': {        'default':{            'level':'ERROR',            'class':'logging.handlers.RotatingFileHandler',            'filename':'error_log',            'formatter':'standard',        }    },    'loggers':{        'default':{            'handlers':['default'],            'level':'INFO',            'propagate': False,        }    }}
```
在views.py中使用log

```import logging…log=logging.getLogger('default')log.error('Something is wrong')```##缓存测试1000个连接如``ab -n 1000 http://192.168.1.108/sales``创建缓存表``python manage.py createcachetable mycache``在setting中配置``CACHE_BACKEND = 'db://mycache'``文件缓存，``file://``是协议，后面是路径如``/usr/local/tmp``

```CACHE_BACKEND = 'file:///usr/local/tmp'
```本地缓存

```CACHE_BACKEND='locmem:///'```
页面缓存

```from django.views.decorators.cache import cache_page```在view方法前加@cache_page(60 * 15)，表示15分钟(60*15秒)后过期##中间件gzip压缩``django.middleware.gzip.GZipMiddleware``反向代理中间件``django.middleware.http.SetRmoteAddrFormForwardedFor`````__init__process_requestprocess_viewprocess_responseprocess_exception```
自定义中间件

```class MyMiddleware(object):    def __init__(self):        pass    def process_view(self,request,func,args,kwargs):        pass```
导入到setting如

```'test.MyMiddleware.MyMiddleware'```##Q

``from django.db.models import Q``model的filter里，字段之间都是AND关系，要用OR，就需要Q```and --> XX.objects.filter(Q(f=1),Q(f=2)) # 肯定木有结果 f == 1 and f == 2 or --> XX.objects.filter(Q(f=1) | Q(f=2)) # f ==1 | f == 2 not --> XX.objects.filter(~Q(f=1),Q(f=2)) # f != 1 and f == 2```
判断某字段是否为null

```_tasks = tasks.exclude(group__isnull=True)```
filter里没有``!=``操作符，需要用``direct_comment = _tasks.filter(~Q(user=1))``，非user等于1##Q 动态 filter如果给filter的参数不全是Q对象，则要如下使用

```kwargs = { 'deleted_datetime__isnull': True }args = ( Q( title__icontains = 'Foo' ) | Q( title__icontains = 'Bar' ) )entries = Entry.objects.filter( *args, **kwargs )```
在全是Q对象的情况下，动态拼接假设搜索条件字段以如下方式存在dictionary conditions中

```conditions {  'field1':[            {'opt':'eq','low':'value1'},            {'opt':'eq','low':'value2'},           ],  'field2':[{'opt':'eq','low':'value2'}]}```

动态获得Q对象

```q = Q()for k,v in conditions.items():    if len(v)==1:        # Only one condition, AND        opt=v[0]['opt']        low=v[0]['low']        high=v[0]['high']        buildQobject(q,k,opt,low,high,Q.AND)    else:        qor = Q()        for c in v:            opt = c['opt']            low = c['low']            high = c['high']            buildQobject(qor, k, opt, low, high, Q.OR)        q.add(qor, Q.AND)def buildQobject(q,fieldname,opt,low,high,ao):    if not low:        return    conKey=fieldname    # Operator mapping    if opt == 'cs':        conKey = ''.join([fieldname, '__icontains'])        q.add(Q(**{conKey: low}), ao)    elif opt == 'nc':        conKey = ''.join([fieldname, '__icontains'])        q.add(~Q(**{conKey: low}), ao)    elif opt == 'eq':        q.add(Q(**{conKey: low}), ao)    elif opt == 'ne':        q.add(~Q(**{conKey: low}), ao)    elif opt == 'lt':        conKey = ''.join([fieldname, '__lt'])        q.add(Q(**{conKey: low}), ao)    elif opt == 'le':        conKey = ''.join([fieldname, '__lte'])        q.add(Q(**{conKey: low}), ao)    elif opt == 'gt':        conKey = ''.join([fieldname, '__gt'])        q.add(Q(**{conKey: low}), ao)    elif opt == 'ge':        conKey = ''.join([fieldname, '__gte'])        q.add(Q(**{conKey: low}), ao)    elif opt == 'bt':        conKey = ''.join([fieldname, '__gte'])        rq = Q()        q1 = Q(**{conKey: low})        conKey = ''.join([fieldname, '__lte'])        q2 = Q(**{conKey: high})        rq.add(q1,Q.AND)        rq.add(q2,Q.AND)        q.add(rq, ao)```##F``from django.db.models import F``判断一个model里的两个字段是否相等，``user``和``assigned_user``属同一个model```
direct_comment = _tasks.filter(user=F('assigned_user'))
```group_by

```def group_by(query_set, group_by):     '''util:django 获取分类列表'''     assert isinstance(query_set, QuerySet)    django_groups = query_set.values(group_by).annotate(Count(group_by))     groups = []     for dict_ in django_groups:         groups.append(dict_.get(group_by))     return groups ```
例如: 

```assign_to = _tasks.exclude(user=F('assigned_user')) groups = group_by(assign_to, 'group') 
```取出的是一个列表groups = [1L, 3L, 4L]
##Model Q对象使用实例简单博客models实体对象```from django.db import models class Blog(models.Model):     name = models.CharField(max_length=100)     tagline = models.TextField()     pub_date =DateField()     def __str__(self):         return self.name class Author(models.Model):     name = models.CharField(max_length=50)     email = models.EmailField()     def __str__(self):         return self.name  class Entry(models.Model):     blog = models.ForeignKey(Blog)     headline = models.CharField(max_length=255)     body_text = models.TextField()     pub_date = models.DateTimeField()     authors = models.ManyToManyField(Author)     def __str__(self):         return self.headline ```
1. Save() 
保存到数据库，方法没有返回值```>>> from mysite.blog.models import Blog >>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.') >>> b.save() ```
2. 自增主键每个数据库模型都会添加一个自增主键字段，即 id ```>>> b2 = Blog(name='Cheddar Talk', tagline='Thoughts on cheese.') >>> b2.id     # Returns None, because b doesn't have an ID yet. None >>> b2.save() >>> b2.id     # Returns the ID of your new object. 14 ```3. 保存对对象做修改
在后台执行了 UPDATE 这一SQL语句```>>> b5.name = 'New name' >>> b5.save() ```4. 获取对象
当你从数据库中获取对象的时候，你实际上用 Manager 模块构造了一个 QuerySet ，这个 QuerySet 知道怎样去执行SQL语句并返回你想要的对象。```>>> blogs = Blog.objects.filter(author__name__contains="Joe")   【过滤数据获取】>>> blogs = Blog.objects.get(id=1)  【获取单个对象】>>> blogs = Blog.objects.all()  【获取所以对象】```
5. 缓存与查询集：为了减少数据库访问次数，每个 QuerySet 包含一个缓存，要写出高效的代码，理解这一很重要。```queryset = BLog.objects.all() print [p.name for p in queryset] # Evaluate the query set. print [p.pub_date for p in queryset] # Reuse the cache from the evaluation. ```6. 过滤对象
用 filter() 和 exclude() 方法可以实现这样的功能```>>> y2006 = Entry.objects.filter(pub_date__year=2006) >>> not2006 = Entry.objects.exclude(pub_date__year=2006) ```7. 级联过滤器：

``` >>> qs = Entry.objects.filter(headline__startswith='What') >>> qs = qs.exclude(pub_date__gte=datetime.datetime.now()) >>> qs = qs.filter(pub_date__gte=datetime.datetime(2005, 1, 1)) ```
8. 限量查询集：

```数据切片的语法来限定 QuerySet 的结果数量，这和SQL中的 LIMIT 和 OFFSET 语句是一样的。>>> Entry.objects.all()[:5]  【返回前五个条目（ LIMIT 5 ）】>>> Entry.objects.all()[5:10] 【返回第六到第十个条目（ OFFSET 5 LIMIT 5 ）】>>> Entry.objects.all()[:10:2] 【回前十个对象中的偶序数对象的列表时】>>> Entry.objects.order_by('headline')[0]  【单个 对象而不是一个列表时（SELECT foo FROM bar LIMIT 1）】```
9. 删除对象

``` b = Blog.objects.get(id=2) b.delete() ```
返回新的 QuerySets 的 查询方法 1. filter(**lookup) 返回一个新的 QuerySet ，包含匹配参数lookup的对象。

```>>>Entry.objects.filter(pub_date__year=2005).```
2. exclude(**kwargs) 返回一个新的 QuerySet ，包含不匹配参数kwargs的对象。

```>>> Entry.objects.exclude(pub_date__year=2005). ```
3. order_by(*fields) 默认情况下，会返回一个按照models的metadata中的``ordering``选项排序的``QuerySet``。你可以调用``order_by()``方法按照一个特定的规则进行排序以覆盖默认的行为：

```>>> Entry.objects.filter(pub_date__year=2005).order_by('-pub_date', 'headline') ```4. distinct() 这消除了查询结果中的重复行,就像使用”SELECT DISTINCT”在SQL查询一样

```>>>Entry.objects.filter(pub_date__year=2005).distinct()```
5. values(*fields) 返回一个特殊的QuerySet相当于一个字典列表，而不是model的实例

```>>> Blog.objects.filter(name__startswith='Beatles').values() [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}] ```
6. dates(field, kind, order) : 参数一：字段名称；参数二：year/month/day；参数三：排序 order="DESC/ASE" 返回QueryDet，指定是DateField或DateTimeField字段名称，日期数据。 
```Kind ="Year"返回所以年份值的数据列表 Kind="Mouth"访问所以月份值的数据列表 Kind="Day"访问所以月份值的数据列表```7. select_related() 在取实体同时取到其外键对应实体

```>>> e = Entry.objects.select_related().get(id=5)# Doesn't hit the database, because e.blog has been prepopulated # in the previous query. >>> b = e.blog  ```
 8. extra() Django查询语法本身没有复杂Where子句，用extra函数就可以使用Where查询子句。
```>>>Entry.objects.extra(select={'is_recent': "pub_date > '2006-01-01'"}) 执行SQL字符串 >>> subq = 'SELECT COUNT(*) FROM blog_entry WHERE blog_entry.blog_id = blog_blog.id' >>> Blog.objects.extra(select={'entry_count': subq}) Where 条件查询   中的使用IN>>> Entry.objects.extra(where=['id IN (3, 4, 5, 20)']) Where条件中使用参数方式一>>>Entry.objects.extra(where=['headline=%s'], params=['Lennon']) Where条件使用参数方式二>>>Entry.objects.extra(where=["headline='%s'" % name]) Where条件中使用参数方式三>>>Entry.objects.extra(where=['headline=%s'], params=[name]) ```
返回另外一个QuerySet对象1. Create(**kwargs)   这个快捷的方法可以一次性完成创建并保证对象 常用保存对象```>>>p = Person(first_name="Bruce", last_name="Springsteen") >>> p.save() 一行保存对象，简介方式保存对象>>>p = Person.objects.create(first_name="Bruce", last_name="Springsteen")  ```
2. get_or_create(**kwargs) 根据参数获取一个对象，如果对象存在则返回对象，如果对象不操作，则创建对象。

```>>> (object,Created)=Enty.object.get_or_create(**kwargs) ```
返回参数中：Object是获取或添加的对象，Created是Boolean值返回是否新创建的对象。

```obj, created = Person.objects.get_or_create( first_name = 'John', last_name  = 'Lennon', defaults   = {'birthday': date(1940, 10, 9)} ) ```
3. count ：统计实体对象集合的记录```>>>Entry.objects.count() >>> Entry.objects.filter(headline__contains='Lennon').count() ```4. in_bulk(id_list)：根据实体对象中的主键ID集合，返回该ID集合中的数据集合。```>>> Blog.objects.in_bulk([1]) {1: Beatles Blog} >>> Blog.objects.in_bulk([1, 2]) {1: Beatles Blog, 2: Cheddar Talk} >>> Blog.objects.in_bulk([]) {} 
```
5. latest(field_name=None)：返回最新、最后添加的对象。参数ield_name：列名```>>>Entry.objects.latest('pub_date') ```6. __exact ：精确匹配```>>>Entry.objects.get(headline__exact="Man bites dog")  >>>Blog.objects.get(id__exact=14) # Explicit form        >>> Blog.objects.get(id=14)      # 等价上面一种方式```
7. __iexact:精确匹配（大小写无关）```>>> Blog.objects.get(name__iexact='lhj588 blog') ```可以把“Lhj588 Blog"、"lhj588 blog"、"LHJ588 BLOG"匹配出来8. __contains：执行严格区分大小写的内容包含检测：```>>>Blog.objects.get(name__contains='lhj') ```可以匹配Blog name包含"lhj"区分大小写 9. __icontains执行一个忽略大小写的内容包含检测：```>>>Blog.objects.get(name__icontains='lhj') ```可以匹配Blog name包含"lhj"不区分大小写10. __gt:筛选大于```>>>Entry.objects.filter(id__gt=5)```
11. __gte:筛选大于等于```>>>Entry.objects.filter(id__gte=10)```
12. __lt: 筛选小于

```>>>Entry.objects.filter(id__lt=15)```
13. __lte筛选小于等于

```>>>Entry.objects.filter(id__lte=20)```
14. __in筛选出包含在给定列表中的数据```>>>Entry.objects.filter(id__in=[1,4,8]) ```这会返回所有ID为1，4，或8的记录对象。 15. __startswith：开头匹配，区分大小写```>>>Blog.objects.filter(name__startswith='Lhj') ```返回标题Lhj588 blog和Lhj888 blog，但是返回lhj588 blog 和lhj888 blog  16. __istartswith:开头匹配，不区分大小写```>>>Blog.objects.filter(name__istartswith='Lhj') ```返回标题Lhj588 blog和Lhj888 blog，同时返回lhj588 blog 和lhj888 blog 17. __endswith：末尾匹配，区分大小写```
>>>Blog.objects.filter(name__endswith='Blog') ```返回标题Lhj588 Blog和Lhj888 Blog，但是返回lhj588 blog 和lhj888 blog 18. __iendswith：末尾匹配，不区分大小写```>>>Blog.objects.filter(name__iendswith='Blog')```返回标题Lhj588 'Blog'和Lhj888 'Blog'，同时返回lhj588 blog 和lhj888 blog 19. __range:范围检测, 就像SQL中的BETWEEN and  ```>>> start_date = datetime.date(2005, 1, 1) >>> end_date = datetime.date(2005, 3, 31) >>> Blog.objects.filter(pub_date__range=(start_date, end_date)) ```
20. __year, __month,and __day：对date/datetime类型字段严格匹配年、月或日 ```>>>Blog.objects.filter(pub_date__year=2005) #查找年份数据>>> Blog.objects.filter(pub_date__month=12) #查找月份数据>>> Blog.objects.filter(pub_date__day=3) #日查找数据 #查找12月25日数据>>> Blog.objects.filter(pub_date__month=12, pub_date_day=25)```21. __isnull:使用``True``或``False``，则分别相当于SQL语句中的``IS NULL``和``IS NOT NULL`` 

```>>>Blog.objects.filter(pub_date__isnull=True)```
22. 主键查询快捷方式：```>>>Blog.objects.get(id__exact=14) # Explicit form >>> Blog.objects.get(id=14) # __exact is implied  >>> Blog.objects.get(pk=14) # pk implies id__exact  >>>Entry.objects.filter(blog__id__exact=3) # Explicit form >>> Entry.objects.filter(blog__id=3) # __exact is implied  >>> Entry.objects.filter(blog__pk=3)# __pkimplies __id__exact ```
23. 使用Q对象做联合查找``filter()``等语句的参数都是取AND运算。如果想要执行更多的联合语句（如``OR``语句），你可以使用 ``Q``对象。  Q 对象 (django.db.models.Q ) 是一个用来囊括参数间连接的对象。这些参数会放在指定的域查询的位置。

```  Q(name__startswith='Lhj')    同SQL： WHERE name LIKE 'Lhj'%'   Q(name__startswith='Lhj') | Q(name__startswith='loker')  同SQL：WHERE name LIKE 'Lhj%' OR name LIKE 'loker%'  你可以用运算符``&``和 |``连接``Q 对象组成任意复杂的语句。你也可以使用附加组。Blog.objects.get(      Q(name__startswith='Lhj'),      Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)) )  同SQL：SELECT * FROM blog WHERE nameLIKE 'Lhj%'      AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06') ```
关系对象对象关系型模型：一对一、一对多、多对多1. 跨越关系查找```>>>Entry.objects.filter(blog__name__exact=’Lhj588 Blog')    >>>Blog.objects.filter(entry__headline__contains='Lennon')     ```
2. 外键关系如果一个模型里面有一个 ForeignKey 字段，那么它的实例化对象可以很轻易的通过模型的属性来访问与其关联的关系对象

```e = Entry.objects.get(id=2)  e.blog = some_blog e.save()  e = Entry.objects.select_related().get(id=2) print e.blog  e = Entry.objects.get(id=2) print e.blog
```
  3. 外键的反引用关系外键关系是自动对称反引用关系的,这可由一个外键可以指向另一个模型而得知.
>如果一个源模型含有一个外键,那么它的外键模型的实例,可以利用”Manager”返回这个源模型的所有实例.默认的这个”Manager”叫做”FOO_set”,这个”FOO”是源模型的名字,小写字母,这个”Manager”将返回”QuerySets”,对这个QuerySets进行过滤和操作,就像在检索对象章节中介绍的.  

```b =Blog.objects.get(id=1)  b.entry_set.all()   #返回到博客相关的所有入口对象。b.entry_set.filter(headline__contains='Lennon') b.entry_set.count()  通过在”ForeignKey()”中定义related_name参数,你可以重载”FOO_set”名字.举例,如果把”Entry”模型修改为”blog =ForeignKey(Blog, related_name=’entries’)”, b= Blog.objects.get(id=1)  b.entries.all() # 返回到博客相关的所有入口对象。b.entries.filter(headline__contains='Lennon') b.entries.count() ```
4. 关系 (连接)
>当你在 model 中定义了一个关系字段(也就是,一个ForeignKey, OneToOneField, 或 ManyToManyField). Django 使用关系字段的名字为 model 的每个实例添加一个描述符. 在访问对象或关联对象时, 这个描述符就象一个常规属性. 
>举例来说, mychoice.poll 会返回 Choice 实例对象关联的 Poll 对象.  通过下面的关系,连接可以以非显式的方式进行: ``choices.objects.filter(poll__slug="eggs") ``得到一个 Choice 对象列表, 这些对象关联的 Poll 对象的 slug 字段值为 eggs. 允许多级连接.  通过一个对象实例的便利函数(convenience functions)就可直接查询该对象的关联对象. 举例来说, 如果 p 是一个 Poll 实例, p.choice_set() 将返回所有关联的 Choice 对象列表. 聪明的读者会注意到它等价于 ``choices.objects.filter(poll__id=p.id)``, 只是更加清晰.  每一种关系类型会为关系中的每个对象自动创建一系列便利方法(类似 ``choice_set()`` 这样的方法).这些方法被双向创建, 这样被关联的对象就不需要明确的定义反向关系, 这一切都是自动完成的.  5. 一对一（One-to-one relationsone-to-one 关系中的每个对象拥有一个 get_relatedobjectname() 方法. 举例来说: 

```class Place(meta.Model):     # ...   class Restaurant(meta.Model):     # ...      the_place = meta.OneToOneField(places.Place)  ```在上面的例子里, 每个 Place 会自动拥有一个 get_restaurant() 方法, 且每个 Restaurant 会自动拥有一个 get_the_place() 方法. 6. 多对一（``Many-to-one relations``）在 ``many-to-one`` 关系中, 关联对象(Many)会自动拥有一个 ``get_relatedobject()`` 方法. 被关联的对象(one)会自动拥有 ``get_relatedobject()``, ``get_relatedobject_list()``, 和 ``get_relatedobject_count()`` 方法 (功能与模块级的 ``get_object()``, ``filter()``, 和 ``get_count()`` 相同).  在上面的民意测试例子里, 一个 Poll 对象 p 自动拥有下列方法: 

```p.get_choice() p.get_choice_list()  p.get_choice_count()  ```Choice 对象 c 则自动拥有下面的方法: ``c.get_poll()``7. 多对多关系（Many-to-Many）
>在多对多关系的两端，都可以通过相应的API来访问另外的一端。 API的工作方式跟前一节所描述的反向一对多关系差不多。唯一的不同在于属性的命名：定义了``ManyToManyField``的model的实例使用属性名称本身，另外一端的model的实例则使用model名称的小写加上``_set``来活得关联的对象集（就跟反向一对多关系一样）

```e = Entry.objects.get(id=3)  e.authors.all() # 返回所有Author对象e.authors.count()  e.authors.filter(name__contains='John') a = Author.objects.get(id=5)  a.entry_set.all() #返回该作者所有入境的对象。```额外方法
1. get_FOO_display()
>设置字段隐藏值，如：性别，M=Male，F=Female ```GENDER_CHOICES = (('M','Male'),('F','Female'),)  class Person(models.Model):          name = models.CharField(max_length=20)      gender = models.CharField(max_length=1, choices=GENDER_CHOICES) 
```

```>>> p = Person(name='lhj', gender='M') >>> p.save() >>>p.gender 'M'  >>>p.get_gender_display() 'Male'   ```2. get_object_or_404():  获取对象，如果对象不存在它将引发 Http404，

```e = get_object_or_404(Entry, pk=3)  a = get_object_or_404(e.authors, name='Fred') e = get_object_or_404(Entry.recent_entries, pk=3)   ```
3. get_list_or_404()  et_list_or_404 行为与 get_object_or_404() 相同，但是它用 filter() 取代了 get() 。如果列表为空，它将引发 Http404 。4. get_next_by_FOO(**kwargs) 和get_previous_by_FOO(**kwargs)
>不存在 null=True 的每个 DateField 和 DateTimeField 自动拥有 get_next_by_FOO() 和 get_previous_by_FOO() 方法, 此处的 FOO 是字段的名字. 它们分别返回该字段的上一个对象和下一个对象. 如果上一对象或下一对象不存在,则抛出 *DoesNotExist 异常. 这两个方法均接受可选关键字参数, 这些参数应该遵循上文中 "Field 查询" 中提到的格式.  注意如果遇到相同值的对象, 这些方法会使用 ID 字段进行检查. 这保证了没有一条记录会被跳过或重复记数.参阅 lookup API sample model_ ,那里有一个完整的例子. 5. get_FOO_filename()
>对一个 FileField 对象来说, 它自动拥有一个 get_FOO_filename() 方法. 这里 FOO 是字段名,它根据你的 MEDIA_ROOT 设置返回一个完整的路径名称.  注意 ImageField 技术上是 FileField 的一个子类, 因此每个有 ImageField 的 model 自动拥有此方法. 6. get_FOO_url()
>含有 FileField 字段的每个对象自动拥有一个 get_FOO_url() 方法,这里的 FOO 是字段的名字. 该方法根据你的 MEDIA_URL 设置返回该文件的完整 URL ,如果 MEDIA_URL 设置为空, 该方法返回一个空的字符串. 7. get_FOO_size()
>含有 FileField 字段的每个对象自动拥有一个 get_FOO_filename() 方法, 这里的 FOO 是字段的名字. 该方法返回文件的长度(字节数).(在后台, Django 使用 os.path.getsize.)  8. save_FOO_file(filename, raw_contents)
>含有 FileField 字段的每个对象自动拥有一个 get_FOO_filename() 方法, 这里的 FOO 是字段的名字. 该方法使用给定的文件名将文件保存到文件系统.. 如果同目录下已经有同名文件存在,Django 会在文件名后添加一个下划线(扩展名之前)直到文件名有效为止(也就是如果加了下划线还重名,就再加....直到没有重名为止). 9. get_FOO_height() 和 get_FOO_width()
>含有 ImageField 字段的每个对象自动拥有 get_FOO_height() 和 get_FOO_width() 方法, 这里的 FOO 是字段的名字. 它们返回相应的图片高度和宽度(整数,以像素计).简单登录逻辑1.	定义用来存登录信息的Model，如用email/password登录，同时这张表一对一关联到User表(存用户昵称等其他信息)

```class UserLogin(models.Model):  email = models.CharField(max_length=30)  password = models.CharField(max_length=50)  user = models.OneToOneField(‘User’)  def __unicode__(self):    return “%s %s %s” % (self.email,self.user.nickName,self.user.realName)```
2.	Admin.py加入```class UserLoginAdmin(admin.ModelAdmin):  list_display=(‘email’,’password’)admin.site.register(UserLogin, UserLoginAdmin)```
3.	Views登录，注册，已登录页面，将用户是否登录的信息存在session中

```from django import forms# 注册表单class UserRegForm(forms.Form):        email = forms.CharField()        password = forms.CharField(widget = forms.PasswordInput)        nickName = forms.CharField()        realName = forms.CharField()# 登录表单class UserLoginForm(forms.Form):        email = forms.CharField()        password = forms.CharField(widget = forms.PasswordInput)def register(request):        if request.method == 'POST':                uf = UserRegForm(request.POST)                if uf.is_valid():                        email = uf.cleaned_data['email']                        password = uf.cleaned_data['password']                        nickName = uf.cleaned_data['nickName']                        realName = uf.cleaned_data['realName']                        u = User.objects.create(nickName=nickName,realName=realName)                        UserLogin.objects.create(email=email,password=password,user=u)                        return HttpResponseRedirect('login')        else:                uf = UserRegForm()        template = loader.get_template('alltest/register.html')        context = RequestContext(request,{        'uf':uf        })        return HttpResponse(template.render(context))        #return render_to_response('alltest/register.html',{'uf':uf})def login(request):        if request.method == 'POST':                uf = UserLoginForm(request.POST)                if uf.is_valid():                        email=uf.cleaned_data['email']                        password=uf.cleaned_data['password']                        userLogin = UserLogin.objects.filter(email__exact=email,password__exact=password)                        if userLogin:                                request.session['email'] = email                                return HttpResponseRedirect('ind')                        else:                                return HttpResponseRedirect('login')        else:                uf = UserLoginForm()        template = loader.get_template('alltest/login.html')        context = RequestContext(request,{        'uf':uf        })        return HttpResponse(template.render(context))        #return render_to_response('alltest/login.html',{'uf':uf})# ind即in.html是登录/未登录都可访问的页面，根据session值判断用户是否登录def ind(request):        email = request.session.get('email','no login')        return render_to_response('alltest/in.html',{'email':email})def logout(request):        session=request.session.get('email',False)        if session:                del request.session['email']                return render_to_response('alltest/logout.html',{'email':session})        else:                return HttpResponse('Please login')```
4.	Urls.py 加入url

```url(r'^ind$',views.ind, name='ind'),url(r'^login$',views.login, name='login'),url(r'^logout$',views.logout, name='logout'),url(r'^register$',views.register, name='register')```5.	Htmllogin.html

```{% extends 'framework/layout/layout.html' %}{% block wheight%}1000{%endblock%}{% load staticfiles %}{% block workarea%}<form method="POST" action="login" enctype="multipart/form-data">{% csrf_token %}{{uf.as_p}}<input type="submit" value="login"></form>{% endblock%}```logout.html

```{% extends 'framework/layout/layout.html' %}{% block wheight%}1000{%endblock%}{% load staticfiles %}{% block workarea%}<h3>Good bye </h3>    {% if email %}    <a href="login">login</a>    {% endif %}{% endblock%}```
in.html```{% extends 'framework/layout/layout.html' %}{% block wheight%}1000{%endblock%}{% load staticfiles %}{% block workarea%}<h3>Welcome to this site !</h3> <p>hello  <span>{{ email }}</span><p>{% if email != 'no login' %}<a href="logout">logout</a>{% else %}<a href="login">login</a>{% endif %}{% endblock%}```
register.html```{% extends 'framework/layout/layout.html' %}{% block wheight%}1000{%endblock%}{% load staticfiles %}{% block workarea%}<form method="post" enctype="multipart/form-data">{% csrf_token %}{{uf.as_p}}<input type="submit" value="register"></form>{% endblock%}```
有效使用Django的QuerySet对象关系映射 (ORM) 使得与SQL数据库交互更为简单，不过也被认为效率不高，比原始的SQL要慢。要有效的使用ORM，意味着需要多少要明白它是如何查询数据库的。本文我将重点介绍如何有效使用 Django ORM系统访问中到大型的数据集。Django的queryset是惰性的Django的queryset对应于数据库的若干记录（row），通过可选的查询来过滤。例如，下面的代码会得到数据库中名字为‘Dave’的所有的人:```person_set = Person.objects.filter(first_name="Dave")```上面的代码并没有运行任何的数据库查询。你可以使用person_set，给它加上一些过滤条件，或者将它传给某个函数，这些操作都不会发送给数据库。这是对的，因为数据库查询是显著影响web应用性能的因素之一。要真正从数据库获得数据，你需要遍历queryset:```for person in person_set:    print(person.last_name)```Django的queryset是具有cache的当你遍历queryset时，所有匹配的记录会从数据库获取，然后转换成Django的model。这被称为执行（evaluation）。这些model会保存在queryset内置的cache中，这样如果你再次遍历这个queryset，你不需要重复运行通用的查询。例如，下面的代码只会执行一次数据库查询：
```pet_set = Pet.objects.filter(species="Dog")# The query is executed and cached.for pet in pet_set:    print(pet.first_name)# The cache is used for subsequent iteration.for pet in pet_set:    print(pet.last_name)```if语句会触发queryset的执行queryset的cache最有用的地方是可以有效的测试queryset是否包含数据，只有有数据时才会去遍历：
```restaurant_set = Restaurant.objects.filter(cuisine="Indian")# `if`语句会触发queryset的执行。if restaurant_set:    # 遍历时用的是cache中的数据    for restaurant in restaurant_set:        print(restaurant.name)```如果不需要所有数据，queryset的cache可能会是个问题有时候，你也许只想知道是否有数据存在，而不需要遍历所有的数据。这种情况，简单的使用if语句进行判断也会完全执行整个queryset并且把数据放入cache，虽然你并不需要这些数据！```city_set = City.objects.filter(name="Cambridge")# `if`语句会执行queryset.。if city_set:    # 我们并不需要所有的数据，但是ORM仍然会获取所有记录！    print("At least one city called Cambridge still stands!")```为了避免这个，可以用``exists()``方法来检查是否有数据：```tree_set = Tree.objects.filter(type="deciduous")# `exists()`的检查可以避免数据放入queryset的cache。if tree_set.exists():    # 没有数据从数据库获取，从而节省了带宽和内存    print("There are still hardwood trees in the world!")```遍历第一个元素会触发数据库操作``for e in Entry.Objects.all():pass````list(Entry.Objects.all()) ``这种不会触发数据库操作分片会在sql上加上limit，不会返回所有数据``Entry.Objects.all()[2:10]``当queryset非常巨大时，cache会成为问题处理成千上万的记录时，将它们一次装入内存是很浪费的。更糟糕的是，巨大的queryset可能会锁住系统进程，让你的程序濒临崩溃。要避免在遍历数据的同时产生queryset cache，可以使用iterator()方法来获取数据，处理完数据就将其丢弃。
```star_set = Star.objects.all()# `iterator()`可以一次只从数据库获取少量数据，这样可以节省内存for star in star_set.iterator():    print(star.name)```当然，使用iterator()方法来防止生成cache，意味着遍历同一个queryset时会重复执行查询。所以使用iterator()的时候要当心，确保你的代码在操作一个大的queryset时没有重复执行查询。如果查询集很大的话，if 语句是个问题如前所述，查询集缓存对于组合 if 语句和 for 语句是很强大的，它允许在一个查询集上进行有条件的循环。然而对于很大的查询集，则不适合使用查询集缓存。最简单的解决方案是结合使用exists()和iterator(), 通过使用两次数据库查询来避免使用查询集缓存。```molecule_set = Molecule.objects.all()# One database query to test if any rows exist.if molecule_set.exists():    # Another database query to start fetching the rows in batches.    for molecule in molecule_set.iterator():        print(molecule.velocity)
```        一个更复杂点的方案是使用 Python 的“ 高级迭代方法 ”在开始循环前先查看一下 iterator() 的第一个元素再决定是否进行循环。```atom_set = Atom.objects.all()# One database query to start fetching the rows in batches.atom_iterator = atom_set.iterator()# Peek at the first item in the iterator.try:    first_atom = next(atom_iterator)except StopIteration:    # No rows were found, so do nothing.    passelse:    # At least one row was found, so iterate over    # all the rows, including the first one.    from itertools import chain    for atom in chain([first_atom], atom_set):        print(atom.mass)
```        
        防止不当的优化queryset的cache是用于减少程序对数据库的查询，在通常的使用下会保证只有在需要的时候才会查询数据库。使用exists()和iterator()方法可以优化程序对内存的使用。不过，由于它们并不会生成queryset cache，可能会造成额外的数据库查询。所以编码时需要注意一下，如果程序开始变慢，你需要看看代码的瓶颈在哪里，是否会有一些小的优化可以帮到你。由于这种cache机制，似乎无法缓存一个旧model，比如带ForeignKey的model teacher, 

```
teacher.student_set.all()oldTeacher = Teacher.Objects.get(id=4)newTeacher = Teacher.Objects.get(id=4)newTeacher.student_set.add …  // 修改关联表值，即数据库值newTeacher.save()```如果此时调用oldTeacher.student_set.all()，那实际上获得的是含有新数据的models集合，因为此刻会执行sql语句，也就表示这种关联条件下，save之前的cache是会丢掉的合并多个QuerySet1) 用chain

```#coding:utf-8from itertools import chaina = [1,2,"aaa",{"name":"roy","age":100}]b = [3,4]c = [5,6]#items = a + b + citems = chain(a,b,c)for item in items:    print item```2) 用 | 符号

```#coding:utf-8from itertools import chainfrom yihaomen.common.models import Articlearticles1 = Article.objects.order_by("autoid").filter(autoid__lt = 16).values('autoid','title')articles2 = Article.objects.filter(autoid = 30).values('autoid','title')articles = articles1 | articles2 # 注意这里采用的方式。如果 Model相同，而且没有用切片，并且字段一样时可以这样用print articles1print articles2print articles```3) Django内用chain函数

```#coding:utf-8from itertools import chainfrom yihaomen.common.models import Article, UserIDarticles1 = Article.objects.order_by("autoid").filter(autoid__lt = 16).values('autoid','title')users = UserID.objects.all()items = chain(articles1, users)for item in items:    print item```
外部py脚本使用django model

```import os,djangoos.environ['DJANGO_SETTINGS_MODULE']='whsales.settings'django.setup()from sales.models import *from sales.common import *print BP.objects.all()```
在model中使用sqlQuerySet的query可查看sqlqueryset = MyModel.objects.all()print queryset.querySELECT"myapp_mymodel"."id", ... FROM"myapp_mymodel"使用raw直接运行sql，比如```>>> from account.models import person            #导入person类>>> raw_sql = 'select * from Person'             #原始sql语句>>> raw_querySet = person.objects.raw(raw_sql)   #xx.objects.raw()执行原始sql>>> print raw_querySet<RawQuerySet: 'select * from Person'>            #RawQuerySet对象>>> for obj in raw_querySet:                     #迭代循环...     print obj...张三李四王五```
直接使用底层sql

```def sql(request):    """    ----------------------------------------------    Function:    执行原始的SQL    DateTime:    2013/x/xx    ----------------------------------------------    """    from django.db import connection,transaction    cursor = connection.cursor()            #获得一个游标(cursor)对象    #更新操作    cursor.execute('update other_other2 set name ="李四" where id=%s',[3])    #执行sql语句    transaction.commit_unless_managed()     #提交到数据库    #查询操作    cursor.execute('select * from other_other2 where id>%s' ,[1])    raw = cursor.fetchone()                 #返回结果行 或使用 #raw = cursor.fetchall()        #如果连接多个数据库则使用django.db.connections    from django.db import connections    _cursor = connections['other_database'].cursor()    #如果执行了更新、删除等操作    transaction.commit_unless_managed(using='other_databases')     return render_to_response('other/sql.html',{'raw':raw})```
反向从已有数据库生成models
开启一个新工程，在settings.py里设置数据库连接使用下列命令将生成的models输出到文件``python manage.py inspectdb > models.py``这样就可以使用新的models文件，然后用``python manage.py syncdb``或者复制到其他项目里使用Ajax POST CSRF问题在Django html页面提交时需要用``{%csrf_token%}``，可以在view的方法上加``@csrf_exempt`` 去掉如果希望Ajax仍使用该csrf token，可以这样，加入js文件，或引入script如下

```function getCookie(name) {  var cookieValue = null;  if (document.cookie && document.cookie != '') {    var cookies = document.cookie.split(';');    for (var i = 0; i < cookies.length; i++) {        var cookie = jQuery.trim(cookies[i]);        // Does this cookie string begin with the name we want?        if (cookie.substring(0, name.length + 1) == (name + '=')) {            cookieValue = decodeURIComponent(cookie.substring(name.length + 1));            break;        }    }  }  return cookieValue;}  function csrfSafeMethod(method) {  // these HTTP methods do not require CSRF protection  return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));}$.ajaxSetup({beforeSend: function(xhr, settings) {    if (!csrfSafeMethod(settings.type) && !this.crossDomain) {        var csrftoken = getCookie('csrftoken');        xhr.setRequestHeader("X-CSRFToken", csrftoken);    }}});```
然后使用ajax，如某个click事件

```eventClick: function(calEvent, jsEvent, view) {  $.post("home",   { pageAction: '', pageApp: 'svr',pageParams:'' },    function (data) {    var resultCollection = jQuery.parseJSON(data);    alert(resultCollection[0]['key'])   })},```
##Django REST framework使用pip安装

```sudo pip install djangorestframework
```
在django settings.py中加入以下段，表示使用原有的admin验证机制```REST_FRAMEWORK = {    # Use Django's standard `django.contrib.auth` permissions,    # or allow read-only access for unauthenticated users.    'DEFAULT_PERMISSION_CLASSES': [        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'    ]}```在INSTALLED_APPS里加入``rest_framework``在app的目录下，类似models.py，创建一个serializers.py，里面是已有model的序列化转化设置，比如```from rest_framework import serializersfrom mytest.models import Roleclass RoleSerializer(serializers.Serializer):    description = serializers.CharField(max_length=50)    level = serializers.IntegerField()    def create(self, validated_data):        """        Create and return a new `Snippet` instance, given the validated data.        """        return Role.objects.create(**validated_data)    def update(self, instance, validated_data):        """        Update and return an existing `Snippet` instance, given the validated data.        """        instance.description = validated_data.get('description', instance.title)        instance.level = validated_data.get('level', instance.code)        instance.save()        return instance```
和Form/ModelForm类似，可以使用ModelSerializer，如

```class RoleSerializer(serializers.ModelSerializer):    class Meta:        model = Role        fields = ('description', 'level')```在view中设置url对应的处理，比如

```@csrf_exemptdef role_list(request):    """    List all code snippets, or create a new snippet.    """    if request.method == 'GET':        roles = Role.objects.all()        serializer = RoleSerializer(roles, many=True)        return JSONResponse(serializer.data)    elif request.method == 'POST':        data = JSONParser().parse(request)        serializer = RoleSerializer(data=data)        if serializer.is_valid():            serializer.save()            return JSONResponse(serializer.data, status=201)        return JSONResponse(serializer.errors, status=400)```
当然需要app的urls.py加入``url(r'^roles/$', views.role_list)``对于单个实体访问，比如``/role/1``，类似地在urls.py里加入```
url(r'^snippets/(?P<pk>[0-9]+)/$', views.snippet_detail),
```并给出一个view里的逻辑实现

```def role_detail(request, pk):    """    Retrieve, update or delete a snippet instance.    """    try:        role = Role.objects.get(pk=pk)    except Role.DoesNotExist:        return JSONResponse(None,status=status.HTTP_404_NOT_FOUND)    if request.method == 'GET':        serializer = RoleSerializer(role)        return JSONResponse(serializer.data)    elif request.method == 'PUT':        serializer = RoleSerializer(role, data=request.data)        if serializer.is_valid():            serializer.save()            return JSONResponse(serializer.data)        return JSONResponse(serializer.errors, status=status.HTTP_400_BAD_REQUEST)    elif request.method == 'DELETE':        role.delete()        return JSONResponse(status=status.HTTP_204_NO_CONTENT)```

django shell 里使用
```from rest_framework.renderers import JSONRendererfrom rest_framework.parsers import JSONParser// 正常的django model创建snippet = Snippet(code='foo = "bar"\n')snippet.save()// 将model包装成Serializer对象，通过.data获得用python基本结构（list，dict）表示的形式serializer = SnippetSerializer(snippet)serializer.data// 使用JSONRenderer渲染成json格式，”(引号)被渲染成\\”content = JSONRenderer().render(serializer.data)// 从json字符串转换成对象from django.utils.six import BytesIO// 使用BytesIO获得输入stream = BytesIO(content)// 转换成python基本格式data = JSONParser().parse(stream)// 类似Django Form一样验证有效性serializer = SnippetSerializer(data=data)serializer.is_valid()# Trueserializer.validated_data# OrderedDict([('title', ''), ('code', 'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])// 直接操作serializer对象，而不是model对象serializer.save()# <Snippet: Snippet object>// 如果是集合，用many=True参数serializer = SnippetSerializer(Snippet.objects.all(), many=True)serializer.data# [OrderedDict([('pk', 1), ('title', u''), ('code', u'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('pk', 2), ('title', u''), ('code', u'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('pk', 3), ('title', u''), ('code', u'print "hello, world"'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]```使用APIView，每一个实体对应一个List view和detail view类

```from snippets.models import Snippetfrom snippets.serializers import SnippetSerializerfrom django.http import Http404from rest_framework.views import APIViewfrom rest_framework.response import Responsefrom rest_framework import statusclass SnippetList(APIView):    """    List all snippets, or create a new snippet.    """    def get(self, request, format=None):        snippets = Snippet.objects.all()        serializer = SnippetSerializer(snippets, many=True)        return Response(serializer.data)    def post(self, request, format=None):        serializer = SnippetSerializer(data=request.data)        if serializer.is_valid():            serializer.save()            return Response(serializer.data, status=status.HTTP_201_CREATED)        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)class SnippetDetail(APIView):    """    Retrieve, update or delete a snippet instance.    """    def get_object(self, pk):        try:            return Snippet.objects.get(pk=pk)        except Snippet.DoesNotExist:            raise Http404    def get(self, request, pk, format=None):        snippet = self.get_object(pk)        serializer = SnippetSerializer(snippet)        return Response(serializer.data)    def put(self, request, pk, format=None):        snippet = self.get_object(pk)        serializer = SnippetSerializer(snippet, data=request.data)        if serializer.is_valid():            serializer.save()            return Response(serializer.data)        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)    def delete(self, request, pk, format=None):        snippet = self.get_object(pk)        snippet.delete()        return Response(status=status.HTTP_204_NO_CONTENT)```
对应的urls.py变为

```urlpatterns = [    url(r'^snippets/$', views.SnippetList.as_view()),    url(r'^snippets/(?P<pk>[0-9]+)/$', views.SnippetDetail.as_view()),]```
使用mixin
python支持多继承，相当于如SnippetList类继承generics.GenericAPIView，并实现接口mixins.ListModelMixin，mixins.CreateModelMixin，然后把get，post方法代理给mixins类的方法```from snippets.models import Snippetfrom snippets.serializers import SnippetSerializerfrom rest_framework import mixinsfrom rest_framework import genericsclass SnippetList(mixins.ListModelMixin,                  mixins.CreateModelMixin,                  generics.GenericAPIView):    queryset = Snippet.objects.all()    serializer_class = SnippetSerializer    def get(self, request, *args, **kwargs):        return self.list(request, *args, **kwargs)    def post(self, request, *args, **kwargs):        return self.create(request, *args, **kwargs)class SnippetDetail(mixins.RetrieveModelMixin,                    mixins.UpdateModelMixin,                    mixins.DestroyModelMixin,                    generics.GenericAPIView):    queryset = Snippet.objects.all()    serializer_class = SnippetSerializer    def get(self, request, *args, **kwargs):        return self.retrieve(request, *args, **kwargs)    def put(self, request, *args, **kwargs):        return self.update(request, *args, **kwargs)    def delete(self, request, *args, **kwargs):        return self.destroy(request, *args, **kwargs)```
当然更简单的方式是使用``generics.ListCreateAPIView``和``generics.RetrieveUpdateDestroyAPIView`````from snippets.models import Snippetfrom snippets.serializers import SnippetSerializerfrom rest_framework import genericsclass SnippetList(generics.ListCreateAPIView):    queryset = Snippet.objects.all()    serializer_class = SnippetSerializerclass SnippetDetail(generics.RetrieveUpdateDestroyAPIView):    queryset = Snippet.objects.all()    serializer_class = SnippetSerializer

```

