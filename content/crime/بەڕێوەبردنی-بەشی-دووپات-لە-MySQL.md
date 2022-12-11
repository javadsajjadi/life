---
layout: post
title: بەڕێوەبردنی بەشی دووپات لە MySQL
tags: [mysql]
date : "2018-06-15 05:35:45"
---

### پێشەکی

بە گشتی کۆی دەرئەنجامەکان یان خشتەکان بریتین لە دراوەی دووجارەکان.ئەمە لە زۆربەی کاتەکان ڕێپێدراوە. بەڵام زۆر جاریش دەبێ تۆمارکراوەی دووجار بوێستێنین. یان تۆمارکراوەی دووجار بناسین و بیانسڕینەوە.

### بەڕێوەبردنی دووپاتکراوەکان لە MYSQL

دەتوانرێت لە کلیلێلی ئەسڵی و INDEXــێکی یەکتا لە خشتەیەک بە خانەی گونجاو بۆ وێستاننی تۆمارکراوەکان بەکاربهێنین.
خشتەی خوارەوە بریتیی لە INDEX یان کلیلی ئەسڵێ نییە. هەر بۆیە رێگەدراوە کە دراوەی دووپاتکراوە لە **first_name** و **last_name** تۆماربکرێت.

```sql
CREATE TABLE person_tbl (
   first_name CHAR(20),
   last_name CHAR(20),
   sex CHAR(10)
);
```



بۆ بەرگیری لە تۆمارکراوەکانی دووجار لە کلیلێلی ئەسڵی بۆ ناساندنی سوود دەگرین.کاتێک ئەم کارە ئەنجام دەدەین. پێویستە کە خانەکان بە بڕی **NOT NULL** ڕابگەیێنین.بۆ ئەوە کەPRIMARY KEY بە بڕی NULL ڕێگە نادا:



```sql
CREATE TABLE person_tbl (
   first_name CHAR(20) NOT NULL,
   last_name CHAR(20) NOT NULL,
   sex CHAR(10),
   PRIMARY KEY (last_name, first_name)
);
```



کاتێک لە خشتەیەک بریتییەل لە index ـێک، زیاد کردنی بڕە دووجارەکان بەو indexــە، دەبێتە نیشاندانی هەڵە.بۆ برگیری لە نیشان دانی هەڵە لەسەر index ـە دووپاتکراوەکان لە فەرمانی **INSERT IGNORE** بەجیاتی فەرمانی **INSERT** سود دەگرین.\\گەر تۆمارکراوەیەک تۆمارکراوەیێکی دیکە ڕوونووس نەگرێت.دەزانین کە MYSQL بە شێوازێکی ئاسایی جیگیری کردووە.
گەر تۆمارکراوەیێک دووجار بێت کلیل وشەی **IGNORE** ، MYSQL ئاگادار دەکا کە ئەو بڕە بێ هیچ هەڵەیەک، چاو پۆشی بکا.
لە نمونەی خوارەوە هیچ هەڵەیەک بۆ نووسینی دراوەی دووجار لە خانە یەکتاکان پیشان نادرێن و لە نووسین و تۆمارکردنی دووجارەکان برگیری دەگرێت:

```sql
mysql> INSERT IGNORE INTO person_tbl (last_name, first_name)
   -> VALUES( 'Jay', 'Thomas');
Query OK, 1 row affected (0.00 sec)
 
mysql> INSERT IGNORE INTO person_tbl (last_name, first_name)
   -> VALUES( 'Jay', 'Thomas');
Query OK, 0 rows affected (0.00 sec)
```



هەروەها لە کلیلە وشەی **REPLACE** بە جیاتی INSERT سوود دەگرین.
لەم حاڵەتەگەر بڕی نوێ دووجار بێت، جێگیری بڕی پێشوو تر دەبێت:



```sql
mysql> REPLACE INTO person_tbl (last_name, first_name)
   -> VALUES( 'Ajay', 'Kumar');
Query OK, 1 row affected (0.00 sec)
 
mysql> REPLACE INTO person_tbl (last_name, first_name)
   -> VALUES( 'Ajay', 'Kumar');
Query OK, 2 rows affected (0.00 sec)
```

فەرمانی INSERT IGNORE یەکەمین کۆی تۆمارکراوە دووجارەکان لەیاددەکا، ئینجا ماوەکەیان دەسڕێتەوە. فەرمانی REPLACE دوایین کۆی لە دووجارەکان لەیاددەکا و هەر جۆرە زانیاری پێش دەسڕێتەوە.
ریگایێکی دیکە لە دروستکردنی خانەی یەکتا، سوود وەرگرتن لە **UNIQUE** بە جیاتی PRIMARY KEY لە خشتەیەکە:

```sql
CREATE TABLE person_tbl (
   first_name CHAR(20) NOT NULL,
   last_name CHAR(20) NOT NULL,
   sex CHAR(10)
   UNIQUE (last_name, first_name)
);
```



### ژماردن و ناساندنی دووجارەکان

نمونەی خوارەوە داوایەکە[^1] کە تۆمارکراوە دووجارەکان لە خانەکانی first_name و last_name دەژمێرێت:

```sql
SELECT COUNT(*) as repetitions, last_name, first_name
   -> FROM person_tbl
   -> GROUP BY last_name, first_name
   -> HAVING repetitions > 1;
```



داواکەی سەرەوە پێرستێک لە هەمووی ئەو تۆمارکراوە دووجارانە لە خشتی person_tbl دەگەڕێنێتەوە.

بەگشتی بۆ ناسینی کۆی بڕەکان کە دووجار بوونەتەوە ئەم قۆناغانە جێنەجی دەکەین:

  * سەرەتا دەبێ دابینی بکەن کام لە خانەکان دراوەی دووجاریان هەیە کە ئیمکانی هەیە دووپاتبکرێنەوە.
  * پێرستی ئەم خانانە لە پێرستی بژاردەی خانەی هاورێ بە  (*) COUNT.
  * خانەکان لە مەرجی GROUP BY یش نەمایەش بدەن.
  * مەرجێکی HAVING زیادبکەن کە تایبەتمەندییەکانی هاوتا دەسڕێتەوە.

### سڕینەوەی دووجار لە دەرئەنجامی داوایەک 

دەتوانن لە فەرمانی **DISTINCT** لە چوارچێوەی SELECT  بۆ پەیداکردنی داواکانی یەکتا لە خشتەیەک بەکاربهێنن:

```
mysql> SELECT DISTINCT last_name, first_name
   -> FROM person_tbl
   -> ORDER BY last_name;
```

جێگرەوەیەبۆ فەرمانی DISTINCT ،ئەمەیە کە مەرجێک GROUP BY زیاد بکەن کە ئەو خانانە کە بژاردەی دەکەن، ناو دەگرێت. ئەمە بۆ سڕینەوەی دووجارەکان و بژاردەی تێکڵآوکراوەکانی یەکتا لە بڕە تایبەتییەکانی خانەکان بەکار دەبردرێت:

```sql
mysql> SELECT last_name, first_name
   -> FROM person_tbl
   -> GROUP BY (last_name, first_name);
```



### سڕینەوەی دووجارەکان بە جێگرەوەکان

گەر تۆمارکراوەی دووجار لە یەک خشتان هەیە وە گەرەکتانە تەواو تۆمارکراوە دووجارەکان لە خشتەیەک بسڕنەوە، ئەم شێوازەی خوارەوە ئەنجام بدەن:

```sql
mysql> CREATE TABLE tmp SELECT last_name, first_name, sex
   -> FROM person_tbl;
   -> GROUP BY (last_name, first_name);
 
mysql> DROP TABLE person_tbl;
mysql> ALTER TABLE tmp RENAME TO person_tbl;
```

شێوازێکی دیکە بۆ سڕینەوەی دووجارەکان لە خشتەیەک، سوود وەرگرتن لە INDEX یان کلیلێکی ئەسڵی بۆ ئەو خشتەیە:

```sql
mysql> ALTER IGNORE TABLE person_tbl
   -> ADD PRIMARY KEY (last_name, first_name);
```