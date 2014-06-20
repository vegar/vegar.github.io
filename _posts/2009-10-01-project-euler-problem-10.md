---
title: Project Euler, Problem 10
author: Vegar
layout: post
permalink: /2009/project-euler-problem-10/
categories:
  - Development
tags:
  - Euler
---
<p>Yet another problem involving primes. Guess I have to make a better prime generator soon…</p>

<blockquote>
<p>The sum of the primes below 10 is 2 3 5 7 = 17.</p>

<p>Find the sum of all the primes below two million.</p>
</blockquote>

<p>Soon, but not yet…</p>

<p>Using the same generator from <a href="http://blog.vi-kan.net/2009/project-euler-problem-7/" title="Find the 10001st prime">problem 7</a>, I go for an easy, but slow, solution for this one. There is one thing, though. Our generator generates up to the nth prime, but in this problem, we need to find every prime below 2000000. Already decided not to make a new prime generator, we cheat by asking it generate a lot of primes, and then jump out as soon as we hit a prime to big.</p>

``` pascal
sum := 0;
for prime in PrimeGenerator(500000) do
begin
  if prime >= 2000000 then
    break;
  sum := sum + prime;
end;
```

<p>I choose to generate 500000 primes, pretty sure one prime out of four should be enough.</p>

<p>And that&#8217;s it – it&#8217;s all there at <a href="http://svn.vi-kan.net/euler">http://svn.vi-kan.net/euler</a>.</p>