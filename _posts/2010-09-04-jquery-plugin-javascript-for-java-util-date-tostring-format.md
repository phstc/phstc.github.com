--- 
layout: post
title: jQuery Plugin-  Javascript for java.util.Date.toString() format
tags: 
- Date
- English posts
- JavaScript
- jQuery
- SimplateDateFormat
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _syntaxhighlighter_encoded: "1"
  _wp_old_slug: jquery-plugin-javascript-for-formatting-java-util-date
  dsq_thread_id: "175191235"
---

[jquery-dateFormat](http://plugins.jquery.com/project/jquery-dateFormat) it's a jQuery Plugin that I made to formatting [java.util.Date.toString](http://download.oracle.com/javase/1.4.2/docs/api/java/util/Date.html#toString%28%29) output using JavaScript. The source code for this plugin is available on [GitHub](http://github.com/phstc/jquery-dateFormat).

--

**Have a look at [jquery-dateFormat on GitHub](http://github.com/phstc/jquery-dateFormat) for an updated documentation.**

--

The plugin was developed using TDD ;)

![](/images/posts/Screen-shot-2010-09-04-at-12.34.51-PM.png)

To execute the test suite, you need to open the [Test.html](http://github.com/phstc/jquery-dateFormat/blob/master/Test.html) in the Firefox. The [JsUnitTest](http://jsunittest.com/) doesn't work in other browsers, perhaps works in other versions, but in  the version that I used in this projects it doesn't work.

##Formatting using css classes

To "automatic" formatting  using css classes, you need to add shortDateFormat or longDateFormat in class attribute on the html objects (span, div, td, label, p etc).

The css class names patterns by default are:

* shortDateFormat = dd/MM/yyyy
* longDateFormat = dd/MM/yyyy hh:mm:ss

Example:

    <span class="shortDateFormat">2009-12-18 10:54:50.546</span>
    <span class="longDateFormat">2009-12-18 10:54:50.546</span>>

Output:

    18/12/2009
    18/12/2009 10:54:50

The  format patterns for shortDateFormat and longDateFormat are defined in  jquery.dateFormat-1.0.js. They are executed on page load by this  function bellow declared in jquery.dateFormat-1.0.js.

    $(document).ready(function () {
     $(".shortDateFormat").each(function (idx, elem) {
     if ($(elem).is(":input")) {
     $(elem).val($.format.date($(elem).val(), 'dd/MM/yyyy'));
     } else {
     $(elem).text($.format.date($(elem).text(), 'dd/MM/yyyy'));
     }
     });
     $(".longDateFormat").each(function (idx, elem) {
     if ($(elem).is(":input")) {
     $(elem).val($.format.date($(elem).val(), 'dd/MM/yyyy hh:mm:ss'));
     } else {
     $(elem).text($.format.date($(elem).text(), 'dd/MM/yyyy hh:mm:ss'));
     }
     });
    });

##Formatting using JavaScript

To formatting using JavaScript you only need to execute $.format.date(text, pattern).

    //Text
    $.format.date('2009-12-18 10:54:50.546', "dd/MM/yyyy");
    //HTML Object
    $.format.date($('#spanDate').text(), "dd/MM/yyyy");
    //Scriptlet
    $.format.date(<%=java.util.Date().toString()%>, "dd/MM/yyyy");
    //JSON
    var obj = ajaxRequest();
    $.format.date(obj.date, "dd/MM/yyyy");

##Format Patterns

The patterns to formatting are based on [java.text.SimpleDateFormat](http://download.oracle.com/javase/1.4.2/docs/api/java/text/SimpleDateFormat.htm).

List of current format patterns available date and time patterns.

* yyyy = long year
* yyyy = short year
* MM = month
* MMM = month abbreviation (Jan, Feb ... Dec)
* MMMM = long month (January, February ... December)
* ddd = day of the week in words (Monday, Tuesday ... Sunday)
* dd = day
* hh = hour in am/pm (1-12)
* HH = hour in day (0-23) 
* mm = minute
* ss = second
* a = am/pm marker 
* SSS = milliseconds

##Excepted input types

* 2009-12-18 10:54:50.546 - The default java.util.Date.toString format
* Wed Jan 13 10:43:41 CET 2010 - (???) When I made this plugin, I needed to develop to this input, but I don't remember who generated it
* 2010-10-19T11:40:33.527+02:00 - The default JAXB formatting of java.util.Date
* Sat Mar 05 2011 11:47:35 GMT-0300 (BRT) - The default JavaScript new Date().toString() output
