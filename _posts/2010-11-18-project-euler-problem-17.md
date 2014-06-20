---
title: Project Euler, Problem 17
author: Vegar
layout: post
permalink: /2010/project-euler-problem-17/
categories:
  - Development
tags:
  - Euler
---
<p>After my success on <a href="http://blog.vi-kan.net/2010/unittesting-on-the-iphone/" title="UnitTesting on the iPhone">getting unittest to work</a> on the iPhone, I thought I should try it out some more. It&#8217;s been a while since last time I solved an Euler Problem, so maybe this would be a good excuse to solve another one? The last problems involved a lot of math, so this time I found something completely different:</p>

<blockquote>
<p>If the numbers 1 to 5 are written out in words: one, two, three, four, five, then there are 3 3 5 4 4 = 19 letters used in total.
If all the numbers from 1 to 1000 (one thousand) inclusive were written out in words, how many letters would be used?</p>
</blockquote>

<p>Well, lets see, then…</p>

<p>We would need a class that can convert an integer into written words. I will call it WrittenNumber, and give it a class method (NSString *)fromInt:(int) to do the work.</p>

```objective-c
@interface WrittenNumber : NSObject {
}
 (NSString *) fromInt:(int)number;
@end
```

<p>Now, that I got my basic interface in place, I tried it out with the most basic test:</p>

```objective-c
- (void)testSingleDigitOne {
    GHAssertEqualStrings([WrittenNumber fromInt:1], @&quot;one&quot;, nil);
}
```

<p>As expected, it failed :</p>

```
    euler17/testSingleDigitOne ✘ 0.00s

    Name: GHTestFailureException
    File: /../euler17.m
    Line: 40
    Reason: '' should be equal to 'one'.
```

<p>I will not take you through every single step I took, including returning &#8216;one&#8217;, writing a new test and realize that returning &#8216;one&#8217; didn&#8217;t work for other cases and so on… What we need, is to identify every word that we need to build up every number from one to 1000. Up to nineteen, there are all unique words. From there, most written numbers are combined by two or more words. So I put these words into a couple of static arrays:</p>

```objective-c
static NSString * simpleNumbers[] = {
    @&quot;&quot;, @&quot;one&quot;, @&quot;two&quot;, @&quot;three&quot;, @&quot;four&quot;, @&quot;five&quot;, @&quot;six&quot;, @&quot;seven&quot;, @&quot;eight&quot;, @&quot;nine&quot;,
    @&quot;ten&quot;, @&quot;eleven&quot;, @&quot;twelve&quot;, @&quot;thirteen&quot;, @&quot;fourteen&quot;, @&quot;fifteen&quot;, @&quot;sixteen&quot;, @&quot;seventeen&quot;, @&quot;eighteen&quot;, @&quot;nineteen&quot;
};

static NSString *tens[] = {
    @&quot;&quot;, @&quot;&quot;, @&quot;twenty&quot;, @&quot;thirty&quot;, @&quot;forty&quot;, @&quot;fifty&quot;, @&quot;sixty&quot;, @&quot;seventy&quot;, @&quot;eighty&quot;, @&quot;ninety&quot;
};
```

<p>I have padded the &#8216;tens&#8217;-array to make indexing a little easier. tens<a href="http://svn.vi-kan.net/euler">2</a> = &#8220;twenty&#8221;, tens[8] = &#8220;eighty&#8221; and so on. Now, the algorithm that I ended  up with, was to start with the larger part of the number, converting it into text, and then handle the rest in the same way. Peele away the larger part, convert it to text, and repeat.</p>

```objective-c
(NSString *) processSimpleNumbers:(int)number
{
    return simpleNumbers[number];
}

(NSString *) processTens:(int)number
{
    if (number &amp;lt; 20) {
        return [self processSimpleNumbers:number];
    }

    int div = number / 10;
    int rest = number % 10;
    NSString *tmp = tens[div];
    if (rest &amp;gt; 0){
        return [NSString stringWithFormat:@&quot;%@-%@&quot;, tmp, [self processSimpleNumbers:rest]];
    } else{
        return tmp;
    }
}

 (NSString *) processHundreds:(int)number;
{
    if (number &amp;lt; 100) {
        return [self processTens:number];
    }

    int div = number / 100;
    int rest = number % 100;
    NSString *tmp = [NSString stringWithFormat:@&quot;%@ hundred&quot;, simpleNumbers[div]];
    if (rest &amp;gt; 0){
        return [NSString stringWithFormat:@&quot;%@ and %@&quot;, tmp, [self processTens:rest] ];
    } else {
        return tmp;
    }
}

 (NSString *) processThousands:(int)number;
{
    if (number &amp;lt; 1000){
        return [self processHundreds:number];
    }

    int div = number / 1000;
    int rest = number % 1000;

    NSString *tmp = [NSString stringWithFormat:@&quot;%@ thousand&quot;, [self processHundreds:div]];

    if (rest &amp;gt; 0) {
        if (rest &amp;lt; 100) {
            return [NSString stringWithFormat:@&quot;%@ and %@&quot;, tmp, [self processTens:rest]];
        } else {
            return [NSString stringWithFormat:@&quot;%@ %@&quot;, tmp, [self processHundreds:rest]];
        }
    } else{
        return tmp;
    }
}

 (NSString *) fromInt:(int)number
{
    return [self processThousands:number];
}
```

<p>I&#8217;m sure there are prettier ways of solving this, but it works. The remaining task, is trivial. Convert each number and accumulate the length of the strings after removing hyphens and spaces. Of cause, we could get rid of these characters right away by not returning them in the first place, but then again, I did not…</p>

<p>I have uploaded the source to my <a href="http://svn.vi-kan.net/euler">svn repository</a> if you want to take a look. You&#8217;ll find the WrittenNumber class in the iPhone/src folder, and the unittests in the iPhone/tests folder.</p>