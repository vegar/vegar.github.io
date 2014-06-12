---
title: UnitTesting on the iPhone
author: Vegar
layout: post
permalink: /2010/unittesting-on-the-iphone/
categories:
  - Development
---
<p><img src="http://blog.vi-kan.net/wp-content/uploads/2010/11/Screen-shot-2010-11-15-at-22.26.41-e1289856633401-150x86.png" alt="" title="GH-Unit" />I have to admit, that I find xcode and objective-c the most challenging developing environment I have ever tried. Specifically, I have struggled to find an easy way of including unittests for an application. Even though it comes with some unittest templates, I haven&#8217;t successfully put it to work. Can&#8217;t say I have tried that much either; it all seemed to complicated to be worth it…</p>

<p>This weekend, the following tweet got my attention:</p>

<pre><code>![TDD for iPhone Development : http://www.vimeo.com/16722907][2]
</code></pre>

<p>The video shows a step-by-step guide on how to get started with unittesting for the iPhone. It looked great, and not that complicated, so I decided to give it a try. 10 minutes later, only delayed by a mysterious &#8220;Failed to launch simulated application: Unknown error.&#8221;-error*, I had my first unittest up and running in the simulator.</p>

<p>If you develop for the iPhone, and either have struggled with unittesting, or just never tried, I will recommend you watch <a href="http://www.vimeo.com/16722907" title="TDD for iPhone @ vimeo">the video</a>. The steps followed in the video is thoroughly documented on the <a href="http://schuchert.wikispaces.com/iPhone.SettingUpTheEnvionment" title="schuchert - iPhone.SettingUpTheEnvionment">authors home page</a>.</p>

<p>*I never found a reason for this error. As for so many other error messages I have encountered in xcode, starting a new project resolved the issue…</p>