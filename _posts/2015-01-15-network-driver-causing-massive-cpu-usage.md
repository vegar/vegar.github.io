---
layout: post
title: Network driver the cause of massive CPU usage
---

> TLDR; For several months, `svchost.exe` has shown a constant CPU usage of about 20%. Today I found it caused by a bad network driver. 

My computer is about one year old. At first I found it both swift and enduring, with a battery lasting even for the longest meetings. But something changed, and I have for the last month learned to bring a power cord with me.

To day I decided to find the source of the problem. 

 * [Process Exlorer](http://technet.microsoft.com/en-us/sysinternals/bb896653.aspx) shows `svchost.exe` hugs the CPU.
 * [Resource Monitor](http://blogs.technet.com/b/askperf/archive/2012/02/01/using-resource-monitor-to-troubleshoot-windows-performance-issues-part-1.aspx) shows `PerfSvc - User Profile Service` being the hugger.
 * [Process Monitor](http://technet.microsoft.com/en-us/sysinternals/bb896645) shows a constant search of the registry. Same few [registry keys](# "HKLM\System\CurrentControlSet\Control\Nls\CustomLocale\virtualization") over and over again. This turns out to be something searchable.
 * [Google](https://www.google.com/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=hklm%5Csystem%5Ccurrentcontrolset%5Ccontrol%5Cnls%5Ccustomlocale%5Cvirtualization) provides a [thread](http://answers.microsoft.com/en-us/windows/forum/windows8_1-performance/profsvc-consumes-100-of-a-thread-in-win-81/42f1daad-79c6-42a1-97d9-609bad32bfa9) describing exact same issue, blaming a bad network driver for the problem.
 * DeviceManager shows my network driver, `Intel Ethernet Connection I217-LM`, is from September 2014, and there's one from late October available for [download](https://downloadcenter.intel.com/SearchResult.aspx?lang=eng&ProdId=3680)
 
And now PerfSvc is gone from the radar. Nice. Looking forward to the next meeting. Hoping it's a long one and that my battery lasts. :-)
