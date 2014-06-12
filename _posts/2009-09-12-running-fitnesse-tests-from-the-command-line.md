---
title: Running FitNesse tests from the command line
author: Vegar
layout: post
permalink: /2009/running-fitnesse-tests-from-the-command-line/
categories:
  - Delphi
  - Development
tags:
  - Fit4Delphi
  - FitNesse
---
<p><img src="http://blog.vi-kan.net/wp-content/uploads/2009/09/image.png" alt="image" title="image" /> Using FitNesse to write and run test is nice, but sometimes you want to run the tests as part of an automatic build cycle. FitNesse has this possibility, and fit4delphi comes with a testrunner that makes it possible with delphi code as well.</p>

<p>In the fit4delphi package, inside the testrunner folder, you will find a project named DelphiTestRunner.dproj. You can use this project to automatically run a test, or a suite of tests, and store the result as a file. To do his, you have to tell the testrunner where the FitNesse server is running, and which file and in what format the result should be store in. This is done with command line parameters in the given form:</p>

<blockquote>
<p>DelphiTestRunner.exe [options] host port page</p>
</blockquote>

<p>Host and port must be the same as specified when starting FitNesse. The page parameter is the url for the wiki page you want to test. All subpages of this page will also be tested.</p>

<p>There are multiple options you can specify.</p>

<ul>
<li>-debug
This option will print FitNesse protocol actions to the console.</li>
<li>-v
This option should give a verbose output about test progress to the console, but personally, I can&#8217;t make it work…</li>
<li>-results file
The result of the testrun is saved to a textfile with the given name.</li>
<li>-html file
The result of the testrun is saved to a html file with the given name.</li>
<li>-xml file
The result of the testrun is saved to a xml file with the given name. The xml format is given <a href="http://fitnesse.org/FitNesse.UserGuide.RestfulTests" title="FitNesse.UserGuide.RestfulTests">in the userguide</a>.</li>
<li>-nopath
This option will make FitNesse ignore !path options specified in the wiki pages.</li>
<li>-suiteFilter filter
Specifying a filter will only run pages with tags matching the filter. You can read more about this <a href="http://fitnesse.org/FitNesse.UserGuide.TestSuites.TagsAndFilters" title="FitNesse.UserGuide.TestSuites.TagsAndFilters">in the userguide</a>.</li>
</ul>

<p>An example of a command line could be some thing like this:</p>

<blockquote>
<p>DelphiTestRunner.exe –html results.html localhost 80 MyTestSuite</p>
</blockquote>