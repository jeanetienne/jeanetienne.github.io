---
layout: post
title: Simple Xcode 9 File Template (Part 1)
---

When you're working on a new area of a project, adding new features, writing new code, or even refactoring and reorganising existing code, you often need to create a lot of new files. The most common way to do so is through the File > New > File... menu in Xcode. Depending on the type of file, you may end up with a more or less elaborate template, with some boilerplate code to help you get started. This can be useful when getting to know the Apple frameworks, or later to save some time writing boring code. Wouldn't it be nice to further customise these templates? To have templates specifically tailored to your own style, projects or tastes? Let's see how to create your own file template with Xcode 9.

Before we start talking about how to customise entire file templates, let's have a quick look at the file headers. Ole Begemann has [a thorough article](https://oleb.net/blog/2017/07/xcode-9-text-macros) on this topic. Apple introduced a new text macro in Xcode 9 that all default templates use: `FILEHEADER`. Ole's article goes in detail about the little quirks and limitiations of this macro. It allows you to customise the file header of any templated file, yours as well as the default Apple ones.

I don't think there's any official documentation to support the Xcode templating system, so what's presented here is from my tinkering with Xcode 9 (beta 6), so take it with a grain of salt.

### Table of Contents
- [Anatomy of a File Template](#anatomy-file-template)
- [Finishing Touch](#finishing-touch)
- [Limitations](#limitations)
- [Sources](#sources)
- [Comments](#comments)

# <a name="anatomy-file-template" href="#anatomy-file-template">Anatomy of a File Template</a>
At the file system level, an Xcode File Template is just a folder with a few conventions. User-defined templates should be located in:
```
~/Library/Developer/Xcode/Templates/File Templates/<Custom Group Name>
```
You can have any number of groups, for this article let's go with `My Custom Templates`. Within this folder you can create any number of templates. So let's imagine we start with a new folder called `Basic Template.xctemplate`, that's our first Xcode File Template.

At the very minimum you'll need the following:
```
📂 Basic Template.xctemplate
  📄 ___FILEBASENAME___.swift
  📄 TemplateInfo.plist
```

Or if you're working with `Obj-C`:
```
📂 Basic Template.xctemplate
  📄 ___FILEBASENAME___.h
  📄 ___FILEBASENAME___.m
  📄 TemplateInfo.plist
```

For this example, let's use an extremely basic source file (we'll see more options to customise later):

{% highlight swift %}
//  ___FILEHEADER___

import Foundation

class ___FILEBASENAME___ {
  
}
{% endhighlight %}

The PList file needs at least the `Kind` key (of string type), which can take 2 values. The most used one is  `Xcode.IDEKit.TextSubstitutionFileTemplateKind`. Another possible value seems dedicated to templates that generate `NSManagedObject` classes for Core Data `Xcode.IDECoreDataModeler.ManagedObjectTemplateKind`. I haven't tried it yet, so I wouldn't be able to advise how to use it. 

The minimal PList file looks like this:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Kind</key>
  <string>Xcode.IDEKit.TextSubstitutionFileTemplateKind</string>
</dict>
</plist>
{% endhighlight %}

Now you should be able to see something like the following screenshot in Xcode:

![](/img/screenshot-1.png)

A `Description` and a `Summary` key were used in prior versions of Xcode to display a tooltip and a description. It looks like because of UI changes in more recent versions, Xcode doesn't use those keys anymore.

Other simple useful keys are:
- `DefaultCompletionName` (string type): used to prepopulate the file name field in the "Save As" dialog.
- `AllowedTypes` (array of string type): used to restrict which type of files can be saved, it is only losely enforced. Values are strings of [Uniform Type Identifiers](https://developer.apple.com/library/content/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html). The most common ones for source files would be:
  - `public.swift-source`
  - `public.c-header`
  - `public.c-source`
  - `public.c-plus-plus-source`
  - `public.objective-c-source`
  - `public.objective-c-plus-plus-source`
- `Platforms` (array of string type): used to restrict the template to certain platforms, which effectively only seems to change under which tab the template will be shown in the wizard. Values can be:
  - `com.apple.platform.macosx`
  - `com.apple.platform.iphoneos`
  - `com.apple.platform.watchos`
  - `com.apple.platform.appletvos`

Let's amend our basic template so our PList looks like the following:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Kind</key>
  <string>Xcode.IDEKit.TextSubstitutionFileTemplateKind</string>
  <key>DefaultCompletionName</key>
  <string>Basic</string>
  <key>AllowedTypes</key>
  <array>
    <string>public.swift-source</string>
  </array>
  <key>Platforms</key>
  <array>
    <string>com.apple.platform.iphoneos</string>
  </array>
</dict>
</plist>
{% endhighlight %}

Now you will only see your basic template under the iOS tab of the file template wizard, and when saving the file, the dialog will prepopulate the file name to `Basic.swift`. If you hit "Create", Xcode will create a copy of the `___FILEBASENAME___.swift` and obviously replace all instances of the `FILEBASENAME` macro with whatever you have entered in the dialog.

What happens if you want your file name to be slightly different from your class name? You may want to have spaces or other characteres replaced by hyphens or underscores. For that pupose, you can use a different macro, or a modifier of a macro, for more information on this topic, I recommend you read the [Xcode Text macro reference](https://help.apple.com/xcode/mac/9.0/index.html?localePath=en.lproj#/dev7fe737ce0), and the [Xcode Text macro format reference](https://help.apple.com/xcode/mac/9.0/index.html?localePath=en.lproj#/devc8a500cb9).

In our specific example, the most reasonable choice is to use the `FILEBASENAMEASIDENTIFIER` macro like so:

{% highlight swift %}
//  ___FILEHEADER___

import Foundation

class ___FILEBASENAMEASIDENTIFIER___ {
  
}
{% endhighlight %}
This will replace any non standard C character with an underscore.

This template could come in handy if you had to generate a lot of similar looking classes. But we're still at the Copy/Paste level of usefulness. Let's see what we can do to make it a bit more dynamic.

# <a name="finishing-touch" href="#finishing-touch">Finishing Touch</a>
The last finishing touch to have a professional looking template is of course to add an icon! Xcode will recognise the following files for your template icon:

- TemplateIcon.png (48 ⨯ 48)
- TemplateIcon@2x.png (96 ⨯ 96)

They need to be placed directly in the template folder, along with the PList.

# <a name="limitations" href="#limitations">Limitations</a>
We saw how to create a basic File Template to generate a single custom file, but nothing prevents you from generating several files in one go. Your template could contain the following structure (it may not make sense for a real-life project, this is just for the sake of example):
```
📂 Basic Template.xctemplate
  📄 ___FILEBASENAME___-Header.h
  📄 ___FILEBASENAME___.swift
  📄 ___FILEBASENAME___View.swift
  📄 ___FILEBASENAME___Model.swift
  📄 ___FILEBASENAME___ViewModel.swift
  📄 ___FILEBASENAME___.xib
  📄 TemplateInfo.plist
```

And when saving with a file name of "ListModule" for instance, it would generate the following files:
```
ListModule-Header.h
ListModule.swift
ListModuleView.swift
ListModuleModel.swift
ListModuleViewModel.swift
ListModule.xib
```

Of course you can customise each file with whatever source makes sense in your context, but you'd still go through a Save dialog like this one:

![](/img/screenshot-2.png)

Which makes you feel like you're creating a single file... 

In the [second part of this article (Advanced Xcode 9 Template)]({% post_url 2017-09-10-advanced-xcode-template %}), we'll see how we can display a "configuration panel" with text fields, drop down menus and checkboxes, much like the default "Cocoa Touch Class" template that lets you choose a few options before generating one or several files. 

You can find the [source for this template on GitHub 🐙](https://github.com/jeanetienne/xcode-9-file-template-examples).

# <a name="sources" href="#sources">Sources</a>
When I first got interested in Xcode Templates I read a few blog articles, most of them were quite dated (from Xcode 4 or around). The best source for up to date information is to have a look at Xcode's default templates located in:

```
Xcode.app/Contents/Developer/Library/Xcode/Templates
or
Xcode.app/Contents/Developer/Platforms/<platform>/Developer/Library/Xcode/Templates/
```

An invaluable reference is [Steffen Itterheim's "Xcode 4 Template Documentation"](http://www.learn-cocos2d.com/store/xcode4-template-documentation). Steffen made this book free a few years ago, so you have no reason not to grab it. It's obviously a bit outdated, but is still very helpful. 

# <a name="comments" href="#comments">Comments</a>
Feel free to comment on [Hacker News](https://news.ycombinator.com) or reach me on [Twitter 🐦](https://twitter.com/jeanetienne)
