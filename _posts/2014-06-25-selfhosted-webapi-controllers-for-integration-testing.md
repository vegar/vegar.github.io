---
layout: post
title: Self host WebAPI controllers for testing
---

> TLDR; With the help of `Owin`, `TestServer` and `HttpClient`, you can do a full integration test of your WebApi in memory without deployment to test host.

I had heard perople talking about 'self-hosting' controllers for tests, and last night I manged to get a test rig up and running that I kind of liked.

So normaly, you can unittest your controllers fairly well by just instantiating it and call the methods on it directly. Some times, though, you want to test everything between the client request and the actual action invokation, like authorization attributes and routing etc.

So here's a basic xUnit test showing how I can have a controller defined inside my test, host it, and make a request to it:

```csharp    
    public class WebApiPingTest : IDisposable
    {
        public class WebApiUnderTestController : ApiController
        {
            [HttpGet, Route("api/ping")]
            public async Task<string> Ping()
            {
                return await Task.FromResult("pong");
            }
        }

        private readonly TestServer _server;
        private readonly HttpClient _client;

        public WebApiPingTest()
        {
            _server = TestServer.Create(app =>
                {
                    var config = new HttpConfiguration();
                    config.MapHttpAttributeRoutes();
                    app.UseWebApi(config);
                });
            _client = new HttpClient(_server.Handler) { BaseAddress = new Uri("http://testserver/") };
        }

        [Fact]
        public async Task WebApiOnOwinUpAndRuning()
        {
            var response = await _client.GetAsync("api/ping");
            var result = await response.Content.ReadAsStringAsync();

            Assert.Equal("\"pong\"", result);
        }

        public void Dispose()
        {
            _client.Dispose();
            _server.Dispose();
        }
    }
```

By separating the constructor and disposer in to a base class, you can get away with very few lines of code.

There are two nuget packages that you need for this to work. First you will need the [microsoft.aspnet.webapi.owinselfhost v5.1.2](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.OwinSelfHost/) package for the selfhosted WebApi, and then you'll need  [microsoft.owin.testing v2.1](https://www.nuget.org/packages/Microsoft.Owin.Testing/) for the `TestServer`.