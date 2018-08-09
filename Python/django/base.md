* 新建项目
  django-admin startproject $PROTECT_NAME
* 运行项目
  python manage.py runserver $HOST:$PORT(默认127.0.0.1:8000)
* 新建app
  python manage.py startapp $APP_NAME
  * setting.py 中install app (以armo命名的app为例)
  INSTALLED_APPS = [ 'armo.apps.ArmoConfig',]
* 请求处理顺序
  client --> urls --> views --> static template / db
* urlpatterns = []
  url正则匹配规则,可以使用include引入其他app的url
* 数据库的使用
  * SQLite
  * MySQL(MariaDB)
  * PostgreSQL
  * Oracle
  Python 3.5以上版本不支持 MySQLdb的驱动，可选
    * pymysql，性能低处理速度慢
    * mysqlclient 支持Python3.3+,django推荐使用
```
MySQL DB API Drivers
MySQL has a couple drivers that implement the Python Database API described in PEP 249:

mysqlclient is a native driver. It’s the recommended choice.
MySQL Connector/Python is a pure Python driver from Oracle that does not require the MySQL client library or any Python modules outside the standard library.
These drivers are thread-safe and provide connection pooling.

In addition to a DB API driver, Django needs an adapter to access the database drivers from its ORM. Django provides an adapter for mysqlclient while MySQL Connector/Python includes its own.

mysqlclient
Django requires mysqlclient 1.3.7 or later.

# settings.py
DATABASES = {
  'default': {
      'ENGINE': 'django.db.backends.mysql',
      'NAME':	'db_name',
      'USER':	'user',
      'PASSWORD':	'password',
      'HOST':	'127.0.0.1',
      'PORT':	3306,
  }
}
```
