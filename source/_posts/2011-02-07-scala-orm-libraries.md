---
layout: post
title: "Scala ORM 工具筆記-Circumflex與ActiveJDBC"
date: 2012-02-07 13:44
comments: true
categories: Scala 
---

#Circumflex#

---

##Prepare##
[Circumflex][]我是使用SBT連到Maven repository去載入他所有的Library。
載入的項目有

    circumflex-orm-2.1.jar
    circumflex-core-2.1.jar
    commons-io-2.0.1
    circumflex-cache-2.1.jar
    slf4j-api-1.6.1
    ehcache-core-2.4.4
    jta-1.1.jar
    c3p0-0.9.1.1.jar

Circumflex全部載入約2.3MB左右，比其他LightWeight的Library要大的很多。

##Create Table##
Circumflex的設計，主要是在是產生一個對應的資料物件，例如

{%codeblock lang:scala%}
class Country extends Record[String, Country] {
  val code = "code".VARCHAR(2).NOT_NULL.DEFAULT("'ch'")
  val name = "name".TEXT.NOT_NULL

  def cities = inverseMany(City.country)
  def relation = Country
  def PRIMARY_KEY = code
}

object Country extends Country with Table[String, Country]
{%endcodeblock%}

其中class與object兩者皆為必要的，但是在object Country的繼承型態宣告，連續繼承了Country與Table。
我覺得這部分會讓人有點困惑為何需要如此處理，但是這還是其次的。

在建立好資料物件之後，就可以利用此資料物件Create Table。

{%codeblock lang:scala%}
    val ddl = new DDLUnit(Country)
    ddl.CREATE
    ddl.DROP
    ddl.DROP_CREATE
{%endcodeblock%}

建立Table的部分非常方便。

##讓我不會用的Association##
關聯式資料庫的架構非常常見，不管是One-to-One、One-to-Many等結構，我的小程式也很需要這方面功能的支持。
Circumflex的設定方式是將變數宣告成inverseMany或inverseOne的型態，他目前沒有支援Many-to-Many。

{%codeblock lang:scala%}
class City extends Record[Long, City] {
  val country = "country_code".TEXT.REFERENCES(Country).ON_DELETE(CASCADE).ON_UPDATE(NO_ACTION)
}

class Country extends Record[String, Country] {
  def cities = inverseMany(City.country)
}
{%endcodeblock%}

inverseMany會回傳一個inverseMany的物件，呼叫inverseMany.get之後可以得到一個seq的物件。這個物件是immutable的
，因此你沒有辦法插入資料。所以我想像中的使用方式是Country與City兩個都是各自去儲存的，
然後可以透過Country.cities()取得所有有關的City，相對的City.country()則可以查詢取得對應的Country。
但是這比較不符合我想要的方式，我還是希望那個Collection是可以讓我CRUD的，並且在儲存Country的時候可以一併更新City。


#ActiveJDBC#

---

##Prepare##
[activejdbc][]所需要的Library比較多一點，因此建議使用Maven去匯入，或者使用SBT將會減少很多麻煩。
匯入的方式可以參考[Getting Start][]。
我在這邊也簡單列出一下使用activejdbc有匯入的Library

    activejdbc-1.2-SNAPSHOT.jar
    slf4j-api-1.5.10
    javalite-common-1.2-SNAPSHOT.jar
    ehcache-core-2.4.5

上面這幾個Library中，以ehcache最大約9xx KB

##Create Table##
activejdbc並沒有支援Create Table等動作，因此這些動作需要自己撰寫SQL來處理。

##Write Model##
寫Model的時候最主要要注意物件的名稱，如果你的Table名稱是"employees"這種複數名詞，那麼你的Model名稱就應該為"employee"。
activejdbc會自動對應複數名詞的Table與單數名詞的Model。(不知道對於es之類的這種變化他是不是也可以處理)。
當然如果有需要的話，也可以使用**@Table**來指定Table名稱。

{%codeblock lang:scala%}
    @Table("TABLE_NAME")
    class employe extends Model{}
{%endcodeblock%}

P.S 幹悲劇了，由於activejdbc裡面的Model的set有三種傳值方式。

    set(Object... namesAndValues)
    set(String[] attributeNames,Object[] values)
    set(String attribute,Object value)

這種寫法Scala在使用的時候會認為是**ambiguous reference to overloaded definition**
因為當我們使用

    employe.set("name","John")
    
的時候，Scala會覺得符合set(Object... namesAndValues)與set(String attribute,Object Value)。
不過我覺得這方面Scala蠻合理的，去查了一下似乎也傾向不會去修改這個。(抱頭 Orz)
如果修改成使用

    set(String[] attributeNames,Object[] values)

就不會有問題了 WTF
另外一個解決方式就是使用setString等指定型態的設定方式，我想這應該是個不錯的解法

    employe.setString("name","John")


#放棄activejdbc#
他還有一個instrumentation的步驟，這個步驟我覺得影響太多了，不適合拿來跟Scala一起合用。



[Circumflex]:http://circumflex.ru/
[activejdbc]:http://code.google.com/p/activejdbc/
[Getting Start]:http://code.google.com/p/activejdbc/wiki/GettingStarted
