---
layout: post
title: بەڕێوەبردن لە بنکەدراوی MySQL
tags: [mysql]
date : "2018-08-14 05:35:45"
---

### پێشەکی

لەم فێرکارییە لەگەڵ چەن کردار کە بریتین لە جێبەجیکردن، چوونەدەرەوە، ریکخستنە پێویستییەکان و کردارە ئەسڵییەکان لە MySQL ئاشنا دەبین.

### جێبەجیکردن و چوونەدەرەوە لە MySQL

سەرەتا دەبێ ئاگادار بین کە MySQL server چالاکە واتە خەریک ئیشکردنە یان نە؛ بۆ ئەم مە بەستە دەتوانینی لە فەرمانی خوارەوە سوود بگرین:

```sql
ps -ef | grep mysqld
```

گەر MySql ، چالاک بێت ئێوە دەتوانن تەواو پرۆسەی پێرستکراوی MySql لە دەرئەنجامی فەرمانەکە ببینن. بەڵام گەر ڕاژەی MySql ناچالاک بێت دەتوانن بە فەرمانی خوارەوە جێبەجێ و وەگڕی بخەن

```sql
root@host# cd /usr/bin
./safe_mysqld &
```



ئێستا گەر هەرەکتانە کە ڕاژەی MySql کە لەحاڵی چالاکی و ئیشە ، بیوێستێنن دەتوانن لە فەرمانەکەی خوارەوە سوود بگرن:

```sql
root@host# cd /usr/bin
./mysqladmin -u root -p shutdown
Enter password: ******
```



### سازدانی هەژمارەی بەکارهێنەر

بۆ زیادکردنی بەکارهێنەری نوێ بە MySQL ،تەنها دەبێ زانیاریێکمان بۆ بۆ خشتەی بەکارهێنەران نیازە. لە فەرمانەکانی خوارەوە بەکارهێنەرێکمان بە ناوی(qezwan) نوێ بە فەرمانەکانی SELECT, INSERT و UPDATE و تێپەڕوشەی (qezwan2020)، زیاد کردووە. ئەم بەم SQL query جۆرە دەبێ

```sql
root@host# mysql -u root -p
Enter password:*******
mysql> use mysql;
Database changed
 
mysql> INSERT INTO user 
   (host, user, password, 
   select_priv, insert_priv, update_priv) 
   VALUES ('localhost', 'qezwan', 
   PASSWORD('qezwan2020'), 'Y', 'Y', 'Y');
Query OK, 1 row affected (0.20 sec)
 
mysql> FLUSH PRIVILEGES;
Query OK, 1 row affected (0.01 sec)
 
mysql> SELECT host, user, password FROM user WHERE user = 'qezwan';
+-----------+---------+------------------+
|    host   |   user  |     password     |    
+-----------+---------+------------------+
| localhost |  qezwan  | 6f8c114b58f2ce9e |
+-----------+---------+------------------+
1 row in set (0.00 sec)
```



هەروا کە لە نمونەی سەرەوە دەیبینن تێپەڕ وشە بە ۶f8c114b58f2ce9e حشاردراوە. سەرنج بدەنە کۆدی FLUSH PRIVILEGES، کە بە ڕاژە دەڵێتکە خشتەکانی grant بار بکات. گەر ئێوە لە کۆدی سەرەوە سوود نەبینن،ناتوانن تا کاتێک کە ڕاژە دووبارە وەگڕدەخرێتەوە بە هەژمارەی نوێ، بە MySQL پەیوەندی بگرن. هەروەها دەتوانن دەستپێگەیشتنەکانی دیکە بە جیاتی پیتی'Y' بە بەکارهێنەری نوێ کە لێرە qezwanــە بدەن.هەروەهەدا دەتوانن دوایی بە فەرمانی ' UPDATE ' بەڕۆژی بکەنەوە.

- Select_priv
- Insert_priv
- Update_priv
- Delete_priv
- Create_priv
- Drop_priv
- Reload_priv
- Shutdown_priv
- Process_priv
- File_priv
- Grant_priv
- References_priv
- Index_priv
- Alter_priv

شێوازەکانی دیکە بۆ زیادکردنی هەژمارەی بەکاهێەری نوێ سوود گرتن لە فەرمانەکانی GRANT SQL ـە. نمونەیێکی دیکە : بەکارهێنەری **brwa** بە تێپەڕ وشەی **brwa123** بۆ بنکەدراوەیێکی تایبەت بە ناوی **KARMEND** زیاد بکەن.

```sql
root@host# mysql -u root -p password;
Enter password:*******
mysql> use mysql;
Database changed
  
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
   -> ON KARMEND.*
   -> TO 'brwa'@'localhost'
   -> IDENTIFIED BY 'brwa123';
```

فەرمانەکەی سەرەوەدێڕێک لە بنکەدراوەیMySQL بە ناوی user دروستدەکات.



لە کۆتایی هەر فەرمان هێمای **سمیکۆلۆن (;)** بنووسن

### سازدانی پەڕگەی /etc/my.cnf File

لەزۆربەی کات نابێت ئەم پەڕگە دەستکاری بکەن، بە شێوازی پێشگریمان ئەم پەڕگە بریتییە لە:

```sql
[mysqld]
datadir = /var/lib/mysql
socket = /var/lib/mysql/mysql.sock
 
[mysql.server]
user = mysql
basedir = /var/lib
 
[safe_mysqld]
err-log = /var/log/mysqld.log
pid-file = /var/run/mysqld/mysqld.pid
```

لێرە دەتوانن دایرێکتۆری جیاواز بۆ هەڵەکان دابین بکەن.بەڵام لە حاڵەتی ئاسایی نابێ دەستکاری ئەم پەڕگە بکەن.



### فەرمانی بەرێوەبردنیMySQL

لێرە پێرستێک لە فەرمانەکانی بەرێوەبردنی MySQL ـمان هاوردووە کە دەتوانن لە کاتی ئیش لەگەڵ بنکەدراوەکەتان سوودی لێبگرن.

- USE Databasename : بۆ هەڵبژاردنی بنکەدراوەیێک لە ژنیگەی MySQL سوود دەگرین.
- SHOW DATABASES : بنکە دراوەکان کە توانایی دەست پێگەیشتنمان هەیە لە نێویان پێرستی دەکەن.
- SHOW TABLES :خشتەکان کە توانایی دەستپێگەیشتنیانمان هەیە پێرستی دەکەن.
- SHOW COLUMNS FROM tablename :تایبەتمەندییەکان و جۆری تایبەتمەندییکەان،زانیاری کلیل، پێش گریمانەکان و زانیارییەکانی خستێک نیشان دەدات.
- SHOW INDEX FROM tablename : تەواو وردە زانیاری index ـەکانی خشت، کە بریتییە لە کلیلی ئەسڵی نیشان دەدا.
- SHOW TABLE STATUS LIKE tablename\G : گوزارشی کرداری DBMSی بنکەدراوەی MYSQL پیشان دەدا.