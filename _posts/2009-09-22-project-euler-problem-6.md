---
title: Project Euler, Problem 6
author: Vegar
layout: post
permalink: /2009/project-euler-problem-6/
categories:
  - Delphi
  - Development
tags:
  - Euler
---
<p>Inspired by my brother over at <a href="http://www.geekality.net/">geekality.net</a>, I thought I should do an attempt on the various problems presented at <a href="http://projecteuler.net/" title="http://projecteuler.net/">projecteuler.net/</a>.  Since he already solved the first five, I jumped right in at <a href="http://projecteuler.net/index.php?section=problems&amp;id=6">problem 6</a>:</p>

<blockquote>
<p>The sum of the squares of the first ten natural numbers is,</p>

<p>12 22 … 102 = 385</p>

<p>The square of the sum of the first ten natural numbers is,</p>

<p>(1 2 … 10)2 = 552 = 3025</p>

<p>Hence the difference between the sum of the squares of the first ten natural numbers and the square of the sum is</p>

<p>3025  385 = 2640.</p>

<p>Find the difference between the sum of the squares of the first one hundred natural numbers and the square of the sum.</p>
</blockquote>

<p>This problem has two distinct parts. First, there is the sum of a sequence of the squared natural numbers, then there is the sum of a sequence of natural numbers, squared.</p>

<h2 id="sumofaseriesofsquarednaturalnumbers">Sum of a series of squared natural numbers</h2>

<p>The problem tells to summarize the square of the first hundred natural numbers. That gives us the following sequence:</p>

<p>12 22 … 992 1002</p>

<p>The simplest solution would be a brute force loop like this one:</p>

<pre><code>function sumSquaredNumbers( ): longword;
var
  i: integer;
begin
  result := 0;
  for i := 1 to 100 do
    result := result   (power(i, 2));
end;
</code></pre>

<p>That wouldn&#8217;t be much fun, though. There has to be a more general, optimized algorithm for this kind of work. First of all, we should make the function workable for any series length. That should be easy – just switch the hardcoded 100 with an given parameter:</p>

<pre><code>function sumSquaredNumbers(serieslength: integer): longword;
var   i: integer;
begin
  for i := 1 to seriesLength do
    result := …
end;
</code></pre>

<p>But the code is still the same, though. Nothing was optimized in any way. We still have the loop, making our function <a href="http://en.wikipedia.org/wiki/Linear_time">linear</a>.</p>

<p>Google to the rescue!</p>

<p><a href="http://www.math.com">math.com</a> have a nice section on Series Expansions, which includes a table of <a href="http://www.math.com/tables/expansion/power.htm">power summations</a>. This table states that</p>

<blockquote>
<p>n</p>

<p>∑
k2
= 1 4 9 … n2
= (1/3)n3 (1/2)n2 (1/6)n</p>

<p>k=1</p>
</blockquote>

<p>Why this should be true, I honestly can&#8217;t tell, but a fast comparison to our brute force solution shows that it gives us the right numbers. That&#8217;s good enough for now:</p>

<pre><code>function sumSquaredNumbers(seriesLength: integer): longword;
begin
  result := trunc(
                ((1/3) * power(seriesLength, 3))
                ((1/2) * power(seriesLength, 2))
                ((1/6) * seriesLength)
              );
end;
</code></pre>

<p>I think this is a nice solution, and a <a href="http://en.wikipedia.org/wiki/Constant_time">constant one</a> as well. It doesn&#8217;t matter if the series is of length one, hundred or thousand – the computation should take the same amount of time.</p>

<h2 id="sumofaseriesofnaturalnumberssquared">Sum of a series of natural numbers, squared</h2>

<p>This part should be a lot easier. We could start with a brute force solution to this part too, but I think we could come up with something smarter right away.</p>

<p>I already know of a simple optimization. If you add the first and last number in the series, you will get the same as if you add the second and next to last number:</p>

<blockquote>
<p>1
2
3
4
5
6
7
8
9
10</p>

<p>10
9
8
7
6
5
4
3
2
1</p>

<p>=
11
11
11
11
11
11
11
11
11
11</p>
</blockquote>

<p>As you can see, every pair of numbers makes the same sum when added together. So for any given series, you can take the first and the last number and add them together, then you can multiply the sum by the length of the series, and finally divide by two.</p>

<blockquote>
<p>n</p>

<p>∑
k
= 1 2 3 … n
= (1 n) * n / 2</p>

<p>k=1</p>
</blockquote>

<p>I know there are other solutions to this, but this one will do.</p>

<pre><code>function sumSquared(seriesLength: longword): longword;
begin
  result := trunc(power(((1   seriesLength) * seriesLength / 2), 2));
end;
</code></pre>

<h2 id="thesolution">The solution</h2>

<p>Now that we have solved each of the two sub parts of the problem, I guess we&#8217;re ready to solve the main thing.</p>

<blockquote>
<p>Find the difference between the sum of the squares of the first one hundred natural numbers and the square of the sum.</p>
</blockquote>

<p>Since we know the sum of the squares, and we now the square of the sum, it&#8217;s just a matter of substract the one from the other:</p>

<pre><code>function diff(serieslength: longword): longword;
begin
  result := sumSquared(serieslength) - sumSquaredNumbers(serieslength);
end;
</code></pre>

<p>I have uploaded my complete solution to <a href="http://svn.vi-kan.net/euler">http://svn.vi-kan.net/euler</a>. The code also contains a simple stopwatch to time the code. This code is so simple, though, that it&#8217;s hard to get any measures.</p>

<p>And that&#8217;s it. Let&#8217;s see if we can solve the next one too.</p>