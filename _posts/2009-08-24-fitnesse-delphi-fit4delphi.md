---
title: 'FitNesse + Delphi –> Fit4Delphi'
author: Vegar
layout: post
permalink: /2009/fitnesse-delphi-fit4delphi/
categories:
  - Delphi
  - Development
tags:
  - Fit4Delphi
  - FitNesse
---
<p><img src="http://blog.vi-kan.net/wp-content/uploads/2009/08/FitNesseLogo.gif" alt="FitNesseLogo" title="FitNesseLogo" />I have been thinking about <a href="http://fitnesse.org/">FitNesse</a> for a couple of weeks now. It seems to be near to perfect for testing the type of code that I&#8217;m currently writing. A lot of calculations with a lot of rules, odd cases and exceptions. So I finally started to check it out, to see if it is possible to use FitNesse to test code written in delphi. After some googling, I found a lot of references to the fact that it is possible, but not much on how. There ain&#8217;t much references to people actually using it either. I found two projects, though, <a href="http://code.google.com/p/fit4delphi/">Fit4Delphi</a> and <a href="http://fitnesse.org/FrontPage.FitServers.DelphiFit">DelphiFIT</a>. It looks like DelphiFIT has been merged with the Fit4Delphi project, so there is only one solution left for FitNesse with Delphi. If it is any good, one is all you need, though. At first, it&#8217;s a bit unclear what components are playing together in FitNesse. There are two main parts of the system, with two different roles. The first part is the one doing the actual testing bit. The second is a development environment in the form of a wiki, made for writing, invoking and maintaining tests.</p>

<h2 id="getitrunning">Get it running</h2>

<p>So let see if we can make this work. Fit4Delphi comes with a older version of FitNesse. You can choose to use this version, or you can get a newer version from <a href="http://fitnesse.org">fitnesse.org</a>. I like new shiny things, so I downloaded a brand new fitnesse.jar. Assuming that java already exists on your computer, fitnesse.jar is all you need to get going. Put it in a folder somewhere, e.g. c: itnesse\, and run</p>

<pre><code>c:itnesse\java -jar fitnesse.jar
</code></pre>

<p>Fitnesse will now unpack it self before it starts. If you later want to upgrade to a newer version of FitNesse, you can download a new jar-file and new resources will be extracted next time you start FitNesse. When FitNesse starts, it will tell you what version it is and what port it is running on. It will also tell you a couple of other things, but the port number is what we need right now. The port number defaults to 80, the standard http port. So fire up you webbrowser, and navigate to <a href="http://localhost:80">http://localhost:80</a>, and you will be presented with the default front page for FitNesse. To the left, you have a tree-structure showing what pages exists in the wiki, and to the right you have the content for the current page. In the FitNesse branch of the tree, you can find the user guide for FitNesse amoung all the acceptance tests for FitNesse itself. On the FrontPage, click the edit-button to the left. At the bottom of the page, write the name of your new testsuite, &#8216;DelphiTests&#8217;, or what ever. It need to be a <a href="http://fitnesse.org/FitNesse.UserGuide.WikiWord">&#8216;WikiWord&#8217;</a>, though. When you save the document, you will see your inserted text together with a &#8216;[?]&#8217;. This means that the wiki sees your wikiword as a title for a page, but can&#8217;t find the page itself. When clicking on the questionmark, the wiki will create the page, and let you edit it. So let&#8217;s start with a simple table. Fill out the new page with something like the following:</p>

<pre><code>|test table|
|in value|out value?|
|1|2|
</code></pre>

<p>After saving, you will see your table nicely formatted. Is there a Test button to the left? If not, click the Properties-button, and choose Test as page type. Now, click the Test-button… Nothing happends, right? FitNesse finds the table and want it to be tested, but can&#8217;t find anyone willing to work. It needs to know who to call for the job. It is done by defining the <a href="http://fitnesse.org/FitNesse.UserGuide.CustomizingTestExecution">&#8216;COMMAND_PATERN&#8217;</a> variable. When testing delphi code, we will need  DelphiFitServer.exe to do the work for us, so we give FitNesse the path to this executable. I have copied both DelphiFitServer.exe and fit.bpl to a &#8216;bin&#8217;-folder in my FitNesse-folder. You will find the source in it4delphi itserver\ and it4delphi it.</p>

<pre><code>!define COMMAND_PATTERN {bin\DelphiFitServer.exe -v %p}
</code></pre>

<p>The –v parameters tell DelphiFitServer to print out messages, and the %p parameter represents the <a href="http://fitnesse.org/FitNesse.UserGuide.MarkupPath">path</a> where DelphiFitServer should look for fixtures. FitNesse builds this path from classpath-settings in the wiki. Let&#8217;s include one in our page:</p>

<pre><code>!path bin\*.bpl
</code></pre>

<p>Let&#8217;s hit that Test button again and see if there is any progress. This time we get a lot of yellow on our page. While red tells us that a test did not pass, yellow tells us that something went wrong when trying execute the test. As you can see, DelphiFitServer couldn&#8217;t find fixture for our table. If you click the &#8216;Output Captured&#8217; sign in the right corner, you will get some details on the execution. You can see what exe-file was used, and what bpl-files was loaded. We now need to add some testcode on our own.</p>

<h2 id="writingourfirstfixture">Writing our first fixture</h2>

<p>From the yellow text, we found that we need a class named &#8216;TestTable&#8217;. Acctually, FitNesse is quite flexible when it comes to naming. You can read more about it <a href="http://fitnesse.org/FitNesse.UserGuide.GracefulName" title="Graceful Naming">the documentation</a>, but I will disclose that a class named &#8216;TTestTable&#8217; should be accepted. So lets create a new delphi package. Add a reference to fit.bpl and the source path for fit4delphi it\source, fit4delphi it\source\exception and fit4delphi\RegExpr. Create a new unit, and define a new class deriving from TColumnFixture. You will need the file ColumnFixture in your uses section for this to compile. You will also need to enable detailed rtti for your class, and register it with a call to RegisterClass:</p>

<pre><code>unit fitTestTable;

interface

uses
  ColumnFixture;

type
  {$M }
  {$METHODINFO ON}
  TTestTable = class(TColumnFixture)
  published
  end;

implementation

uses
  Classes;

{ TTestTable }

initialization
  RegisterClass(TTestTable)

end.
</code></pre>

<p>Build the new package, and move it to the bin folder of your FitNesse installation. Run the test again, and you will find that there is now three exceptions instead of one. Progress! This time, it chokes on the column titles. Could not find field: in value. Well, lets add it, then. A published property named InValue of type double should do. Compile… Copy… Test…  One down, two to go. Of course it choked on the next column also. This time we need to add a function named OutValue returning a double.  Let&#8217;s fake the implementation for now, and just return 0.</p>

<pre><code>   TTestTable = class(TColumnFixture)
  private
    FInnValue: double;
  published
    property InnValue: double read FInnValue write FInnValue;
    function OutValue: double;
  end;
</code></pre>

<p>Now, that got rid of both the remaining exceptions, but leaves us with some ugly red spots. This is actually good though: Our test runs with out problems, but fails. I will leave it up to you to make the test pass. It shouldn&#8217;t be to hard, and the reward of green colour should be enough to make you wanna write some real tests.</p>