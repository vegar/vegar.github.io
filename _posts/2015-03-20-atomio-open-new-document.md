---
title: "atom.io: opening a new document"
author: Vegar
layout: post
categories:
  - Development
published: true
date: "2015-03-20 11:46"
---

Opening documents in atom.io, is managed through the [`open` method of the `workspace`](https://atom.io/docs/api/v0.187.0/Workspace#instance-open) object.

To open a new document in atom.io, omit the URI parameter of the open call:

```coffee
{CompositeDisposable} = require 'atom'

module.exports = MyPackage =

    subscriptions: null

    activate: (state) ->
        # Events subscribed to in atom's system can be easily cleaned up with a CompositeDisposable
        @subscriptions = new CompositeDisposable

        # Register command that toggles this view
        @subscriptions.add atom.commands.add 'atom-workspace', 'my-package:execute': => @execute()

    deactivate: ->
        @subscriptions.dispose()

    execute: ->
        # no URI given
        atom.workspace.open()
```
When saving the document, you will be prompted for a filename.

If you provide a URI to a non-existing file, a new document will be created with that filename. The file is not created before you hit the `save` command. You will not be prompted for filename.

```coffee
execute: ->
    # URI for non-existing URI given
    atom.workspace.open('c:\\temp\\temp.txt')
```

![](/images/2015/03/temp.txt_-_atom.png)


If you do not provide a full URI, just a name, it will use the current projects root folder. If no project is open, you will get a new un-named document.

```coffee
execute: ->
    # relative URI for non-existing file
    atom.workspace.open('newdoc')
```

At this time, this is [undocumented behavior](https://github.com/atom/atom/pull/6005), but hopefully, not for long.
