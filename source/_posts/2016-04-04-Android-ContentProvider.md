---
title: Content Provider on Android
author: YalesonChan
profile_picture: http://www.famousbirthdays.com/faces/laurel-stan-image.jpg
short_bio: If anyone at my funeral has a long face, I'll never speak to him again.
author_site: https://github.com/cyw3
---
## 前言

Content Provider是Android组件中最基本也是最为常见用的四大组件之一。

主要用于对外共享数据，也就是通过Content Provider把应用中的数据共享给其他应用访问，其他应用可以通过Content Provider对指定应用中的数据进行操作。Content Provider分为系统的和自定义的，系统的也就是例如联系人，图片等数据。

Android中对数据操作包含有：

file, sqlite3, SharedPreferences, ContectResolver与ContentProvider。前三种数据操作方式都只是针对本应用内数据，程序不能通过这三种方法去操作别的应用内的数据。Android中提供ContectResolver与ContentProvider来操作别的应用程序的数据。

使用方式：

一个应用实现ContentProvider来提供内容给别的应用来操作

一个应用通过ContentResolver来操作别的应用数据，当然在自己的应用中也可以。

以下这段是Google Doc中对ContentProvider的大致概述：

内容提供者将一些特定的应用程序数据供给其它应用程序使用。内容提供者继承于Content Provider基类，为其它应用程序取用和存储它管理的数据实现了一套标准方法。然而，应用程序并不直接调用这些方法，而是使用一个 ContentResolver对象，调用它的方法作为替代。ContentResolver可以与任意内容提供者进行会话，与其合作来对所有相关交互通讯进行管理。

## 目录
1. 为什么使用Content Provider？
2. Content Provider的关键类
3. ContentProvider的使用
4. 监听ContentProvider中数据的变化

## 一、为什么使用Content Provider？

关于数据共享，以前我们学习过文件操作模式，知道通过指定文件的操作模式为Context.MODE_WORLD_READABLE或Context.MODE_WORLD_WRITEABLE同样也可以对外共享数据。那么，这里为何要使用ContentProvider对外共享数据呢？

是这样的，如果采用文件操作模式对外共享数据，数据的访问方式会因数据存储的方式而不同，导致数据的访问方式无法统一，如：采用xml文件对外共享数据，需要进行xml解析才能读取数据；采用sharedpreferences共享数据，需要使用sharedpreferences API读取数据。

使用ContentProvider对外共享数据的好处是统一了数据的访问方式。

## 二、Content Provider的关键类

1、ContentProvider

Android提供了一些主要数据类型的ContentProvider，比如音频、视频、图片和私人通讯录等。可在android.provider包下面找到一些Android提供的ContentProvider。通过获得这些ContentProvider可以查询它们包含的数据，当然前提是已获得适当的读取权限。

主要方法：

```ruby
public boolean onCreate() 在创建ContentProvider时调用；
public Cursor query(Uri, String[], String, String[], String) 用于查询指定Uri的ContentProvider，返回一个Cursor；
public Uri insert(Uri, ContentValues) 用于添加数据到指定Uri的ContentProvider中；
public int update(Uri, ContentValues, String, String[]) 用于更新指定Uri的ContentProvider中的数据；
public int delete(Uri, String, String[]) 用于从指定Uri的ContentProvider中删除数据；
public String getType(Uri) 用于返回指定的Uri中的数据的MIME类型。
```

> public String getType(Uri) 用于返回指定的Uri中的数据的MIME类型

1）如果操作的数据属于集合类型，那么MIME类型字符串应该以vnd.android.cursor.dir/开头。

例如：要得到所有person记录的Uri为content://contacts/person，那么返回的MIME类型字符串为"vnd.android.cursor.dir/person"。

2）如果要操作的数据属于非集合类型数据，那么MIME类型字符串应该以vnd.android.cursor.item/开头。

例如：要得到id为10的person记录的Uri为content://contacts/person/10，那么返回的MIME类型字符串应为"vnd.android.cursor.item/person"。

2、ContentResolver

当外部应用需要对ContentProvider中的数据进行添加、删除、修改和查询操作时，可以使用ContentResolver类来完成，要获取ContentResolver对象，可以使用Context提供的getContentResolver()方法。

> ContentResolver cr = getContentResolver();

ContentResolver提供的方法和ContentProvider提供的方法对应的有以下几个方法。

```ruby
public Uri insert(Uri uri, ContentValues values) 用于添加数据到指定Uri的ContentProvider中；
public int delete(Uri uri, String selection, String[] selectionArgs) 用于从指定Uri的ContentProvider中删除数据；
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) 用于更新指定Uri的ContentProvider中的数据；
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) 用于查询指定Uri的ContentProvider。
```

3、Uri解析类

Uri指定了将要操作的ContentProvider，其实可以把一个Uri看作是一个网址，我们把Uri分为三部分：

第一部分是"content://"，可以看作是网址中的"http://"；

第二部分是主机名或authority，用于唯一标识这个ContentProvider，外部应用需要根据这个标识来找到它。可以看作是网址中的主机名，如"blog.csdn.net"；

第三部分是路径名，用来表示将要操作的数据。可以看作网址中细分的内容路径。

![Uri.png](/img/Uri.png)

因为Uri代表了要操作的数据，所以我们经常需要解析Uri，并从Uri中获取数据。Android系统提供了两个用于操作Uri的工具类，分别为UriMatcher和ContentUris 。

1）UriMatcher类用于匹配Uri。

1、把你需要匹配Uri路径全部给注册上

```ruby
//常量UriMatcher.NO_MATCH表示不匹配任何路径的返回码
UriMatcher  sMatcher = new UriMatcher(UriMatcher.NO_MATCH);
sMatcher.addURI("com.cyw.provider.personprovider", "person", 1);
sMatcher.addURI("com.cyw.provider.personprovider", "person/#", 2);//#号为通配符
switch (sMatcher.match(Uri.parse("content://com.cyw.provider.personprovider/person/10"))) { 
   case 1
     break;
   case 2
     break;
   default://不匹配
     break;
}
```

2、使用sMatcher.match(uri)方法对输入的Uri进行匹配，如果匹配就返回匹配码，匹配码是调用addURI()方法传入的第三个参数​

2）ContentUris类用于操作Uri路径后面的ID部分，它有两个比较实用的方法：

1、withAppendedId(uri, id)用于为路径加上ID部分：

```ruby
Uri uri = Uri.parse("content://com.cyw.provider.personprovider/person")
Uri resultUri = ContentUris.withAppendedId(uri, 10); 
//生成后的Uri为：content://com.cyw.provider.personprovider/person/10
```

2、parseId(uri)方法用于从路径中获取ID部分：

```ruby
Uri uri = Uri.parse("content://com.cyw.provider.personprovider/person/10")
long personid = ContentUris.parseId(uri);//获取的结果为:10
```


## 三、ContentProvider的使用

1、需要继承ContentProvider并重写方法

2、需要在AndroidManifest.xml使用<provider>对该ContentProvider进行配置，为了能让其他应用找到该ContentProvider ，ContentProvider采用了authorities（主机名/域名）对它进行唯一标识，你可以把ContentProvider看作是一个网站，authorities 就是域名。
​

## 四、监听ContentProvider中数据的变化

1、如果ContentProvider的访问者需要知道ContentProvider中的数据发生变化，可以在ContentProvider发生数据变化时调用getContentResolver().notifyChange(uri, null)来通知注册在此URI上的访问者，例子如下：

```ruby
public class PersonContentProvider extends ContentProvider {
   public Uri insert(Uri uri, ContentValues values) {
      db.insert("person", "personid", values);
   getContext().getContentResolver().notifyChange(uri, null);
   }
}
```

2、如果ContentProvider的访问者需要得到数据变化通知，必须使用ContentObserver对数据（数据采用uri描述）进行监听，当监听到数据变化通知时，系统就会调用ContentObserver的onChange()方法：

```ruby
getContentResolver().registerContentObserver(Uri.parse("content://com.cyw.providers.personprovider/person"),
       true, new PersonObserver(new Handler()));
public class PersonObserver extends ContentObserver{
   public PersonObserver(Handler handler) {
      super(handler);
   }
   public void onChange(boolean selfChange) {
      //此处可以进行相应的业务处理
   }
}
```
