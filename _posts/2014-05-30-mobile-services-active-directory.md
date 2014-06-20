---
layout: post
title: Azure Mobile Services & Active Directory authentication
---

> TLDR; When authenticating against Azure Active Directory through the Azure Mobile Services, use the magic string `aad` as he provider, not `windowsazureactivedirectory` as stated in the documentation. 

I just tried out [Azure Mobile Services](http://azure.microsoft.com/en-us/develop/mobile/). If you need a simple backend for your app, mobile services may be the way to go. It supports all mayor mobile plattforms (iOS, Android and WinPhone) as well as html5/js and windows store applications.

One of the things it gives you, is easy authentication through 3rd party providers like facebook and twitter. It also supports azure active directory, which is nice for internal applications.

There is one problem, though. The documentation seems to be a little off...

There are documentation for all platforms. Lets have a look at [the iOS](http://azure.microsoft.com/en-us/documentation/articles/mobile-services-ios-get-started-users/) one: 

---
1. Open the project file QSTodoListViewController.m and in the **viewDidLoad** method, remove the following code that reloads the data into the table:

```objective-c
        [self refresh];
```

2.  Just after the **viewDidLoad** method, add the following code:  

```objective-c
        - (void)viewDidAppear:(BOOL)animated
        {
            MSClient *client = self.todoService.client;            
            if (client.currentUser != nil) {
                return;
            }            
            [client loginWithProvider:@"facebook" controller:self animated:YES completion:^(MSUser *user, NSError *error) {
                [self refresh];
            }];
        }
```

> **Note**
> If you are using an identity provider other than Facebook, change the value passed to `loginWithProvider` above to one of the following: _microsoftaccount_, _facebook_, _twitter_, _google_, or _windowsazureactivedirectory_.

3. Press the **Run** button to build the project, start the app in the iPhone emulator, then log-on with your chosen identity provider.

    When you are successfully logged-in, the app should run without errors, and you should be able to query Mobile Services and make updates to data.

---

The code shows how to authenticate through facebook, and the note states that you can use active directory instead by changing `@"facebook"` to `@"windowsazureactivedirectory"`.

When I tried this, I got a `401` error with the message

>"Error: Authentication with 'windowsazureactivedirectory' is not supported."

For some reason I decided to try with `@"aad"` instead, and lo and behold - it worked! 

Since the documentation is hosted on GitHub, I dropped a [pull request](https://github.com/Azure/azure-content/pull/1827) to notify them of the issue. 

*Update*: the pull request was excepted, and the documentation will soon be (more) accurate on this topic. 