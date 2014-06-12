---
title: 'Particle-challenge part 2: OK – the math-part is a little bit hard…'
author: Vegar
layout: post
permalink: /2009/particle-challenge-part-2-ok-the-math-part-is-a-little-bit-hard/
categories:
  - Development
tags:
  - Challenge
  - OpenGL
  - Particle
---
<p><img src="http://imgs.xkcd.com/comics/matrix_transform.png" alt="Matrix Transformation" title="In fact, draw all your rotational matrices sideways. Your professors will love it! And then they'll go home and shrink." /> It&#8217;s summer. I&#8217;m 500 km from home, and I&#8217;m trying to find some time to continue on the particle-challenge.  In <a href="http://blog.vi-kan.net/2009/1st-particle-challenge-getting-something-unto-the-screen/#more-55" title="1st particle-challenge: Getting something unto the screen">the last post</a>, I didn&#8217;t decide where to go next; The world, the particles or physics. After some thinking, I found the first one most important, and chose to read a little about how the coordinate system in OpenGL works. It sounds simple enough, but after some coding, I really feel that I need more math skills to get control of this. I have now started to read &#8216;<a href="http://my.safaribooksonline.com/0735713901" title="‘Beginning Math and Physics for Game Programmers’">Beginning Math and Physics for Game Programmers</a>&#8217; to see if I can get the hang of it.</p>

<p>I have started on a camera class, but can&#8217;t make it work quite like I want to. There ain&#8217;t much functionality in it yet, either.</p>

<p>The camera consist of three vectors: eye position, forward direction and up direction. In theory, the class should make a rotation-matrix from the forward and up vectors, and then translate the matrix by the eye vector.</p>

<p>For now, it uses gluLookAt( ) to do the transformations instead. The reason why is simply that <a href="http://stackoverflow.com/questions/1176782/opengl-help-with-camera-transformation" title="StackOverflow: &quot;OpenGL: Help with camera transformation&quot;">I can&#8217;t get the math to work</a>… I&#8217;ll have to look at it some more at a later time, but now, I want more particle-fun and less math-hassle.</p>

<p>A  <a href="http://svn.vi-kan.net/particle/0.2/">snapshot of the code</a> is available.</p>