---
title: Project Euler, Problem 9
author: Vegar
layout: post
permalink: /2009/project-euler-problem-9/
categories:
  - Development
tags:
  - Euler
---
<p>On our way to Lofoten last weekend, my wife and I had a small brainstorming on problem no. 9:</p>

<blockquote>
<p>A Pythagorean triplet is a set of three natural numbers, a &lt; b &lt; c, for which,</p>

<p>a2 b2 = c2</p>

<p>For example, 32 42 = 9 16 = 25 = 52.</p>

<p>There exists exactly one Pythagorean triplet for which a b c = 1000.
Find the product abc.</p>
</blockquote>

<p>Both of us agreed there had to be some kind of mathematical formula for this kind of problem. The problem, though, was that non of us new this formula, and there wasn&#8217;t that many math books in the car that day… So instead of trying to guess the magic formula, we tried to optimize the search for the right numbers.</p>

<p>We could put three for loops inside each other, scanning the numbers from 1 to 1000 for each. Trying 1 1 1 as a possible solution isn&#8217;t that clever, though. Neither is the combination 1000 1000 1000.</p>

<p>Let&#8217;s take a look at the maximum possible values for a, b and c. The fact that a &lt; b &lt; c makes this kind of easy.  Since all three numbers must be positive, c can&#8217;t be larger than 997 to make room for a = 1 and b = 2. The number b can&#8217;t be larger than 499, which makes room for a = 1 and c = 500. And finally, a can&#8217;t be larger than 332, which makes room for b = 333 and c = 335.</p>

``` pascal
    for a := 1 to 332 do
      for b := a 1 to 498 do
        for c := b 1 to 998 do
        begin
          ...
        end;
```

<p>Now, the test it self should be easy:</p>

``` pascal
if (a + b + c = 1000) and (a*a + b*b = c*c) then
  ...
```

<p>We can do one more easy optimization. I our current c makes for a larger sum than 1000, we can break out of the innermost loop. This actually saves us a lot of cycles.</p>

``` pascal
        for c := b + 1 to 998 do
        begin
          sum := a + b + c;
          if (sum) > 1000 then
            break;

          if (sum = 1000) and (a*a + b*b = c*c) then
          begin
            ...
          end;
        end;
```

<p>And that&#8217;s it. The complete solution is available as usual. Check it out at <a href="http://svn.vi-kan.net/euler">http://svn.vi-kan.net/euler</a>.</p>