---
title: Project Euler, Problem 7
author: Vegar
layout: post
permalink: /2009/project-euler-problem-7/
categories:
  - Development
tags:
  - Euler
---
<p>It want be easy to keep up with <a href="http://www.geekality.net/2009/09/24/project-euler-problem-7/">my brother</a> on this. Well, here&#8217;s my take on Project Eulers problem no 7:</p>

<blockquote>
<p>By listing the first six prime numbers: 2, 3, 5, 7, 11, and 13, we can see that the 6th prime is 13.</p>

<p>What is the 10001st prime number?</p>
</blockquote>

<p>Algorithms conserning primes is a large and complex field. I guess the are many different solutions to this problem, so the question will be how efficient the one that I find will be.</p>

<p>One thing is for sure, though. We need a way of finding primes. As far as I know, there is no formula to find the nth prime. We have to find the first 10 000 primes first before we can find our wanted one.</p>

<h2 id="aprimegenerator">A Prime Generator</h2>

<p>I will use a <a href="http://17slon.com/blogs/gabr/2007/03/fun-with-enumerators-part-6-generators.html">enumerator-pattern</a> for our prime generator. For that, we will need two classes. First we will need a factory-class that implements the GetEnumerator function. Then we will need the class that handles the actual enumeration.</p>

<pre><code>type
  TPrimeGenerator = class
  public
    function GetCurrent: extended;
    function MoveNext: boolean;
    property Current: extended read GetCurrent;
  end;

  TPrimeGeneratorFactory = class
  public
    function GetEnumerator: TPrimeGenerator;
  end;
</code></pre>

<p>With this implementation, we will be allowed to write code like this:</p>

<pre><code>for prime in TPrimeGeneratorFactory.Create do
begin
  ...
end;
</code></pre>

<p>This code would generate an awful lot of primes, though. It would be nice to give it some kind limit. We can add that to the constructor, and check in the MoveNext method if the limit is reached. We would also need some live-time management on both the factory and the generator. The generator it self can be created and destroyed by the factory, but we can also make it easier by letting the factory implement an interface. If we do, delphi will manage it&#8217;s lifetime for us.</p>

<pre><code>type
  TPrimeGenerator = class
  public
    constructor Create(numberOfPrimes: integer);
    ...
  end;

  IPrimeGeneratorFactory = interface
    function GetEnumerator: TPrimeGenerator;
  end;

  TPrimeGeneratorFactory = class(TInterfacedObject, IPrimeGeneratorFactory)
  public
    constructor Create(numberOfPrimes: integer);
    destructor Destroy; override;
  end;

  function PrimeGenerator(numberOfPrimes: integer): IPrimeGeneratorFactory;
</code></pre>

<p>Now, we can rewrite our for … in – loop, and just get the number of primes that we want. We will know longer have any leaks either.</p>

<pre><code>for prime in PrimeGenerator(10001) do
begin
  ...
end;
</code></pre>

<h2 id="findingprimes">Finding Primes</h2>

<p>Our generator doesn&#8217;t actually generate primes yet, though. Let&#8217;s see if we can make it do so. So how do we find primes? What is a prime? Wikipedia gives us the following definition of <a href="http://en.wikipedia.org/wiki/Prime_number">prime numbers</a>:</p>

<blockquote>
<p>In mathematics, a prime number (or a prime) is a natural number which has exactly two distinct natural number divisors: 1 and itself.</p>
</blockquote>

<p>So if we loop through all numbers, and test if it&#8217;s possible to devide it by any other number, we should have our primes. The problem is, though, that this takes time. A lot of time. We have to find a way of shortening down the ammount of numbers we need to check. The first thing we should remove, is all even numbers. Every single even number is divisible by the number 2, so they can&#8217;t be primes. Except for the number 2 it self – it&#8217;s a prime, cause has only two divisors: 1 and itself.</p>

<p>Since 2 is the first prime number, it would be an easy exception to handle, and we have effectively cut shortened down possible primes to the half of what we started with.</p>

<p>Let&#8217;s write our FindNextPrime method:</p>

<pre><code>function TPrimeGenerator.FindNextPrime(PreviousPrimeFound: extended): extended;
var
  found: boolean;
  currentNumber: extended;
begin
  currentNumber := PreviousPrimeFound;
  repeat
    if (CurrentNumber = 1) or (currentNumber = 2) then
    begin
      currentNumber := currentNumber   1;
      result := true;
    end
    else
    begin
      currentNumber := currentNumber   2;
      found := CheckIfPrime(currentNumber);
    end;
  until found;

  result := currentNumber;
end;
</code></pre>

<p>We here assume that PreviousPrimeFound equals 1 when no previous prime is found. So if the previous prime is 1 or 2, we just increment by one, knowing that both 2 and 3 is prime numbers, and signal a successfull found. If previous prime was not 1 or 2, it has to be 3 or any other odd number, so we increment by two to skip even numbers.</p>

<p>Let&#8217;s take a look at the CheckIfPrime( ) method. How do we check if a number is divisible by any other number? I guess we have to check:</p>

<pre><code>function TPrimeGenerator.CheckIfPrime(number: integer): boolean;
var
  i: integer;
begin
  result := true;
  for i := 2 to number - 1 do
  begin
    if number mod i = 0 then
    begin
      result := false;
      break;
    end;
  end;
end;
</code></pre>

<p>Can you see a bottleneck here? For every number we test, there is one more number to devide. How can we optimize this?</p>

<p>First of all, every natural number can be expressed as a product of primes. 12 = 2 x 2 x 3, 20 = 2 x 2 x 5 and so on. So if a number is divisible by a non prime number, it should also be divisible by a prime number. So if we can find a way of just dividing by primes we would speed up things a lot.</p>

<p>So let&#8217;s add a list of previous found primes to the generator class, and loop through that list instead of all numbers.</p>

<pre><code>function TPrimeGenerator.CheckIfPrime(number: extended): boolean;
var
  itr: DIterator;
  prevPrime: extended;
  found: boolean;
begin
  found := false;
  itr := FPrimes.start;
  while IterateOver(itr) do
  begin
    prevPrime := getExtended(itr);

    if Frac(number / prevPrime) = 0 then
    begin
      found := true;
      break;
    end;
  end;

  result := not found;
end;
</code></pre>

<p>This makes things a lot faster, but I think we still can gain something. In the same wikipedia article that I refered to earlier, it is sufficient to test primes smaller than the sqaure root of the number we are currently testing. So let&#8217;s add that to our loop:</p>

<pre><code>function TPrimeGenerator.CheckIfPrime(number: extended): boolean;
var
  itr: DIterator;
  prevPrime: extended;
  found: boolean;
  root: extended;
begin
  root := sqrt(number);
  ...
  while IterateOver(itr) do
  begin
    prevPrime := getExtended(itr);
    if prevPRime &amp;gt; root then
      break;

    if ...
  end;
  ...
end;
</code></pre>

<h2 id="thesolution">The Solution</h2>

<p>Just like the solution for problem 6, you can find the complete code at <a href="http://svn.vi-kan.net/euler">http://svn.vi-kan.net/euler</a>. It gives the answer in about 40&#8211;50 ms.</p>