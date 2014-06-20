---
layout: post
title: LightInject constructor injection corner case
---

> TLDR; LightInject will always use the constructor with the most resolvable parameters. If there's only one named service of a given type, LightInject will be able to resolve this as the default service.

I'm using [LightInject](http://www.lightinject.net/) for dependency injection, and so far it has worked great. It's a small, [fast](http://www.palmmedia.de/blog/2011/8/30/ioc-container-benchmark-performance-comparison) and easy to use library with good documentation too.

I hit one bump in the road earlier this week, though. I have a dependency to a service with multiple constructors, but so far I have only needed the default constructor. Now I wanted to use one of the other two, but LightInject did not choose the one that I expected.

Let's say this is my service:

```csharp
public interface IMyService
{
    string CalledConstructor { get; }         
}

public class MyService : IMyService
{
    public MyService()
    {
        CalledConstructor = "MyService()";                
    }

    public MyService(string parameterOne)
    {
        CalledConstructor = string.Format("MyService(parameterOne: {0}", parameterOne);                
    }

    public MyService(string parameterOne, string parameterTwo)
    {
        CalledConstructor = string.Format("MyService(parameterOne: {0}, parameterTwo: {1}", parameterOne, parameterTwo);
    }

    public string CalledConstructor { get; private set; }            
}
```
    
In the simplest case, we would register the service, and it would resolve using the default constructor:    

```csharp
[TestMethod]
public void Resolve_should_use_default_constructor()
{
    var container = new ServiceContainer();
    container.Register<IMyService, MyService>();

    var service = container.GetInstance<IMyService>();
   
    Assert.AreEqual("MyService()", service.CalledConstructor);
}
```

If we need to provide a value for `parameterOne`, we would register that as well:

```csharp
[TestMethod]
public void FAILS_Resolve_should_use_constructor_with_one_parameter_but_chooses_the_one_with_two()
{
    var container = new ServiceContainer();
    container.RegisterInstance("FirstValue", "parameterOne");
    container.Register<IMyService, MyService>();

    var service = container.GetInstance<IMyService>();

    Assert.AreEqual(
        string.Format("MyService(parameterOne: {0}", "FirstValue"), 
        service.CalledConstructor);
}
```

And here's where things went wrong for me.

> Test Name:  FAILS\_Resolve\_should\_use\_constructor\_with\_one\_parameter\_but\_chooses\_the\_one\_with\_two
>
> **Result Message**: Assert.AreEqual failed.  
> **Expected**:\<MyService(parameterOne: FirstValue\>.  
> **Actual**:\<MyService(parameterOne: FirstValue, parameterTwo: FirstValue\>.  
 
LightInject chose the third constructor, the one taking two parameters, and used the value given to `parameterOne` for both parameters. Why is that?

The [documentation](http://www.lightinject.net/#toc14) states that
> Note: In the case where the implementing type(Foo) has more than one constructor, LightInject will choose the constructor with the most parameters.

It [also states](http://www.lightinject.net/#toc4) that
> If only one named registration exists, LightInject is capable of resolving this as the default service.

So LightInject will first try to resolve the constructor with the most parameters, which in this case is the one taking two strings. It will then try to resolve each parameter; first by serviceName, then by using the default for the given type. In this case it will find a service called `parameterOne` that it uses for the first parameter, but for the second parameter it want find any service registered with the name `parameterTwo`, so it will try finding one without any name. Since there is only one string instance registered, it will use this as default, even though it was registered with a name.

So how can this be solved?

##1. Hide unwanted constructors  

If a constructor shouldn't be used, there's no need for it. If it's our service, we could delete the constructor, or change the access modifier. If it's part of an external library, we could inherit it and only expose the one that we need.

```csharp
public class MyInheritedService : MyService 
{
    public MyInheritedService(string parameterOne) : base(parameterOne) { }
}

[TestMethod]
public void Solution_1_hide_constructor()
{
    var container = new ServiceContainer();
    container.RegisterInstance("FirstValue", "parameterOne");        
    container.Register<IMyService, MyInheritedService>();

    var service = container.GetInstance<IMyService>();

    Assert.AreEqual(
        string.Format("MyService(parameterOne: {0}", "FirstValue"), 
        service.CalledConstructor);
}
```

##2. Register a dummy service

Since this is a problem only when we have just one string instance registered, we could register a second 'dummy' service to prevent the default resolving.

```csharp
[TestMethod]
public void Solution_2_prevent_resolveing_of_default_service()
{
    var container = new ServiceContainer();
    container.RegisterInstance("FirstValue", "parameterOne");
    container.RegisterInstance("dummy", "dummy");
    container.Register<IMyService, MyService>();

    var service = container.GetInstance<IMyService>();

    Assert.AreEqual(
        string.Format("MyService(parameterOne: {0}", "FirstValue"), 
        service.CalledConstructor);
}
```

All test cases are available as a [Gist](https://gist.github.com/vegar/fa4e35b557ed37150d43) 
