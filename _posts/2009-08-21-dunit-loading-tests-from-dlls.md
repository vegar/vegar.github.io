---
title: 'DUnit: Loading tests from dll’s'
author: Vegar
layout: post
permalink: /2009/dunit-loading-tests-from-dlls/
categories:
  - Delphi
  - Development
tags:
  - Delphi
  - DUnit
  - TDD
---
<p>I wanted to split all my unittests for a project into separate packages to keep tings nice and clean. <a href="http://dunit.sourceforge.net/" title="DUnit: An Xtreme testframework for Delphi programs">DUnit</a> comes with a unit called <a href="http://dunit.svn.sourceforge.net/viewvc/dunit/trunk/src/TestModules.pas?revision=15&amp;view=markup" title="View testmodules.pas from sourceforge.net">TestModules.pas</a> which helps you do that.</p>

<p>The unit contains three public methods:</p>

<pre><code>function LoadModuleTests(LibName: string) :ITest;
procedure RegisterModuleTests(LibName: string);
procedure UnloadTestModules;
</code></pre>

<p>The LoadModuleTests( )-function, dynamically loads the given library (dll or dtl files), and returns an interface to the tests defined within. RegisterModuleTests( ) does the same, but also calls RegisterTest( ) with the interface from the library.</p>

<p>There is one requirement that the library has to satisfy. It must export a function named &#8216;Test&#8217; that returns a ITest-interface. Following is a basic project file showing how this can be done:</p>

<pre><code>library OurUnitTests;

uses
  TestFramework;

function Test: ITest;
begin
  result := RegisteredTests;
end;

exports
  Test name 'Test';

end;
</code></pre>

<p>The Test( )-function simply returns the result from the RegisteredTests( ) method from the TestFramework-unit. RegisteredTests( ) returns a ITestSuite containing all the test classes registered.</p>

<p>Here&#8217;s a samle project file that loads all dll-files from the executables folder, and runs all test through the GUITestRunner:</p>

<pre><code>program RunTests;
uses
  Forms,
  TestFramework,
  GUITestRunner,
  TestModules,
  SysUtils,
  Windows,
  JclFileUtils,
  classes,
  Dialogs
  ;

  {$R *.RES}

procedure LoadTests;
var
  filelist: TStringList;
  str: string;
begin
  filelist := TStringList.Create;
  try
    BuildFileList('*.dll', faAnyFile, filelist);

    for str in filelist do
    begin
      try
        RegisterModuleTests(str)
      except
        on E: Exception do
          ShowMessage(e.Message);
      end;
    end;
  finally
    filelist.free;
  end;
end;

begin
  Application.Initialize;

  LoadTests;
  GUITestRunner.RunRegisteredTests;
  UnloadTestModules;
end.
</code></pre>

<p>The BuildFileList( ) method comes from JCL, and simply fills a TStringList with filenames from the given mask. As I said, this is a very basic project, but it shows how to use the TestModules-unit to dynamically load tests. This can easily be expanded to a more flexible solution. You could use command parameters to control what folder should be searched for libraries, or what testrunner should be used, GUI or XML or custom runner.</p>