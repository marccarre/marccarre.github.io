---
layout: post
status: publish
published: true
title: Unit testing of an asynchronous method in C#
author:
  display_name: zhong
  login: Zh0nG
  email: zh0nggu0.1337@gmail.com
  url: http://www.zh0ng.net
author_login: Zh0nG
author_email: zh0nggu0.1337@gmail.com
author_url: http://www.zh0ng.net
excerpt: How to write NUnit tests for an asynchronous method in C#.
wordpress_id: 117
wordpress_url: http://www.zh0ng.net/?p=117
date: '2010-07-13 21:50:13 +0100'
date_gmt: '2010-07-13 20:50:13 +0100'
categories:
- C# &#47; .NET
tags:
- Unit test
- C#
- CSharp
- Asynchronous
- Delegate
- Event
- NUnit
- Threading
- Synchronization
comments:
- id: 1984
  author: David Keaveny
  author_email: davidkeaveny@gmail.com
  author_url: http://davidkeaveny.blogspot.com
  date: '2012-06-21 01:27:05 +0100'
  date_gmt: '2012-06-21 00:27:05 +0100'
  content: This approach works well; but please fix the formatting of the code sample,
    as it's very broken and difficult to read.
- id: 1985
  author: zhong
  author_email: zh0nggu0.1337@gmail.com
  author_url: http://www.zh0ng.net
  date: '2012-06-21 05:47:34 +0100'
  date_gmt: '2012-06-21 04:47:34 +0100'
  content: |-
    Thanks David!
    FYI, I have just fixed the formatting.
    Cheers.
- id: 2259
  author: Stefan Staub
  author_email: ste.staub@gmail.com
  author_url: ''
  date: '2013-05-06 16:59:43 +0100'
  date_gmt: '2013-05-06 15:59:43 +0100'
  content: Thanks for this. I had one problem when I added the Asserts inside the
    callback, it always resulted in Timeout. I just made the result available outside
    the callback and made the asserts after the "Sync" block. Then it worked as a
    charm :)
- id: 2291
  author: EeKay's Blog &ndash; NUnit and Multiple Asynchronous webservice calls in
    C#
  author_email: ''
  author_url: http://me.eekay.nl/index.php/2013/nunit-and-multiple-asynchronous-webservice-calls-in-c/
  date: '2013-05-24 06:55:51 +0100'
  date_gmt: '2013-05-24 05:55:51 +0100'
  content: "[...] this nice post at zhong&#8217;s website indicates, you can use the
    ManualResetEvent class to let the main thread wait until it gets [...]"
---
<p>Whether you are applying <a title="Introduction to Test Driven Development" href="http:&#47;&#47;www.agiledata.org&#47;essays&#47;tdd.html" target="_blank">Test Driven Development<&#47;a> or just want to unit test some function, you probably already know <a title="NUnit: Unit Testing for C#" href="http:&#47;&#47;www.nunit.org&#47;" target="_blank">NUnit<&#47;a>, which like all the *Unit, give some useful tools to write tests for your C# code.<&#47;p></p>
<p>However you may have some asynchronous methods you would like to test, especially as C# offers you very convenient ways to write them:<&#47;p></p>
<ol>
<li>using some <a title="Asynchronous methods using delegates, among other" href="http:&#47;&#47;support.microsoft.com&#47;kb&#47;315582&#47;en" target="_blank">delegates<&#47;a> to make a random thread from the "Thread Pool" execute it, while you're performing another operation in your main thread, and...<&#47;li>
<li>using some <a title="C# events" href="http:&#47;&#47;msdn.microsoft.com&#47;en-us&#47;library&#47;ms366768(v=VS.80).aspx" target="_blank">events<&#47;a> to eventually notify the end of the operation, and why not, provide its results, using <a title="C# customized events" href="http:&#47;&#47;msdn.microsoft.com&#47;en-us&#47;library&#47;w369ty8x(v=VS.80).aspx" target="_blank">your own *EventArgs class<&#47;a>.<&#47;li><br />
<&#47;ol></p>
<p>In this unit test, you would like to be sure your asynchronous operation is correctly performed, and <strong><span style="text-decoration: underline;">always<&#47;span><&#47;strong> give the correct result:<&#47;p><br />
[sourcecode language="csharp"]<br />
[Test]<br />
public void TestAsyncMethodWithEvent()<br />
{<br />
    &#47;&#47; Set up some handler to check the result of the asynchronous operation:<br />
    AsyncOperationComplete += new EventHandler<AsyncOperationEventArgs>((source, args) =><br />
    {<br />
        Assert.IsNotNull(args.AsyncOperationResult); &#47;&#47; The test fails if we don't have the expected result.<br />
    });<br />
    &#47;&#47; Perform the asynchronous operation:<br />
    AsyncOperation();<br />
}<br />
[&#47;sourcecode]<br &#47;><br &#47;></p>
<p>But as says the name, it's asynchronous and therefore could be tricky to test as you don't directly manage the flow of execution of your background thread, the one performing the asynchronous operation in "parallel".<&#47;p></p>
<p>In the above example, the test would just continue its execution by exiting TestAsyncMethodWithEvent() and returning a nice "Success" even if AsyncOperation() failed.<&#47;p></p>
<p>That's why, in order to make it deterministic, you will need to make it "unparallel", using some synchronization tools, like the <a title="C#: ManualResetEvent" href="http:&#47;&#47;msdn.microsoft.com&#47;en-us&#47;library&#47;system.threading.manualresetevent(VS.80).aspx" target="_blank">ManualResetEvent<&#47;a> class.<br &#47;> This way, you will be able to quietly wait for the end of the background thread, get the result and use NUnit to assert its correctness:<&#47;p><br />
[sourcecode language="csharp"]<br />
[Test]<br />
public void TestAsyncMethodWithEvent()<br />
{<br />
    ManualResetEvent syncEvent = new ManualResetEvent(false);<br />
    AsyncOperationComplete += new EventHandler<AsyncOperationEventArgs>((source, args) =><br />
    {<br />
        Assert.IsNotNull(args.AsyncOperationResult); &#47;&#47; The test fails if we don't have the expected result.<br />
        syncEvent.Set(); &#47;&#47; Notifies the end of the asynchronous operation.<br />
    });<br />
    AsyncOperation();<br />
    if (!syncEvent.WaitOne(10000)) &#47;&#47; Wait for the end of the asynchronous operation during 10 seconds maximum.<br />
    {<br />
        Assert.Fail("Timeout."); &#47;&#47; The test fails if the operation didn't end fast enough.<br />
    }<br />
}<br />
[&#47;sourcecode]<br &#47;><br &#47;></p>
<p>As you can see in this second example, we just:<&#47;p></p>
<ol>
<li>wait for the synchronization event in our main thread.<&#47;li>
<li>send the synchronization event in the background thread, as soon as the result has been tested.<&#47;li>
<li>we don't wait more than, say 10 seconds, as the operation is supposed to be quick.<&#47;li><br />
<&#47;ol></p>
<p>This last point, about setting a time limit, can save your server if you are repeatedly running your tests there, for example, doing some&nbsp;<a title="Continuous Integration" href="http:&#47;&#47;martinfowler.com&#47;articles&#47;continuousIntegration.html" target="_blank">continuous integration<&#47;a> with&nbsp;<a title="Hudson: Extensible continuous integration server" href="http:&#47;&#47;hudson-ci.org&#47;" target="_blank">Hudson<&#47;a>.<&#47;p></p>
