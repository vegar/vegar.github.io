---
title: Project Euler, Problem 8
author: Vegar
layout: post
permalink: /2009/project-euler-problem-8/
categories:
  - Development
tags:
  - Euler
---
<p>A couple of days ago, I solved eulers problem no. 8. I finally found some time to blog about my solution.</p>

<blockquote>
<p>Find the greatest product of five consecutive digits in the 1000-digit number.</p>

<p>73167176531330624919225119674426574742355349194934
96983520312774506326239578318016984801869478851843
85861560789112949495459501737958331952853208805511
12540698747158523863050715693290963295227443043557
66896648950445244523161731856403098711121722383113
62229893423380308135336276614282806444486645238749
30358907296290491560440772390713810515859307960866
70172427121883998797908792274921901699720888093776
65727333001053367881220235421809751254540594752243
52584907711670556013604839586446706324415722155397
53697817977846174064955149290862569321978468622482
83972241375657056057490261407972968652414535100474
82166370484403199890008895243450658541227588666881
16427171479924442928230863465674813919123162824586
17866458359124566529476545682848912883142607690042
24219022671055626321111109370544217506941658960408
07198403850962455444362981230987879927244284909188
84580156166097919133875499200524063689912560717606
05886116467109405077541002256983155200055935729725
71636269561882670428252483600823257530420752963450</p>
</blockquote>

<p>I can&#8217;t see any smart solution for this problem. A simple brute force scan through the series of digits will have to do.</p>

<p>I start out with the number as a string. That makes it easy to loop through digit by digit.  To make it even easier, I have made an array of five bytes that I place &#8216;over&#8217; the current character. This gives me a &#8216;view&#8217; of the five current digits that needs to be multiplied as bytes instead of characters.</p>

<pre><code>const
  SUBJECT = '73167176531330624919225119674426574742355349 .... 57530420752963450';

type
  PFiveDigits = ^TFiveDigits;
  TFiveDigits = array[0..4] of byte;

begin
  for I := 1 to length(Subject) - 6 do
  begin
    FiveDigits := @SUBJECT[i];
  end;
end;
</code></pre>

<p>A character in the range 0 to 9 has byte value in the range of 48 to 57. Before we multiply the values, we have to normalize them. We can do that by subtracting 48 from each byte value.</p>

<pre><code>const
  NULLCHAR = ord('0');
  ...
  present := (FiveDigits[0]-NULLCHAR) * (FiveDigits[1]-NULLCHAR) * (FiveDigits[2]-NULLCHAR) * (FiveDigits[3]-NULLCHAR) * (FiveDigits[4]-NULLCHAR);
</code></pre>

<p>Now we just have to keep track of the highest produced product do find the answer.</p>

<pre><code>    largest := 0;
    for I := 1 to length(Subject) - 6 do
    begin
      FiveDigits := @SUBJECT[i];
      present := (FiveDigits[0]-NULLCHAR) * (FiveDigits[1]-NULLCHAR) * (FiveDigits[2]-NULLCHAR) * (FiveDigits[3]-NULLCHAR) * (FiveDigits[4]-NULLCHAR);
      if present &amp;gt; largest then
      begin
        largest := present;
        position := i;
      end;
    end;
</code></pre>

<p>As usual, you will find the code at <a href="http://svn.vi-kan.net/euler">http://svn.vi-kan.net/euler</a>.</p>