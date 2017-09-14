---
layout: post
title: Advanced Xcode 9 File Template (Part 2)
---

In the [first part of this article (Simple Xcode 9 File Template)]({% post_url 2017-08-27-xcode-template %}) we saw how to create a simple Xcode File Template to generate a custom file. Let's go beyond the Copy/Paste type of template, and see what we can do to make a more dynamic template with options to let the user tailor the generated files.

### Table of Contents
- [Adding Options](#adding-options)
- [Code Variants](#code-variants)
- [Sources](#sources)
- [Comments](#comments)

# <a name="adding-options" href="#adding-options">Adding Options</a>
In the [previous example]({% post_url 2017-08-27-xcode-template %}) we saw how to create a basic File Template to generate a single custom file. Now let's see how we can display a "configuration panel" with text fields, drop down menus and checkboxes, much like the default "Cocoa Touch Class" template that lets you choose a few options before generating one or several files. 

For this example, let's pretend we want a template to create a View, a Model and a ViewModel for an iOS app. We also want to give the opportunity to let the user choose if the view is coming from the Main storyboard, from a XIB file or from code, so the loading code can be generated adequately.

Let's start with a few simple files for now in this new template:
```
📂 View, Model & ViewModel.xctemplate
  📄 ___FILEBASENAME___.swift
  📄 ___FILEBASENAME___View.swift
  📄 ___FILEBASENAME___ViewModel.swift
  📄 TemplateInfo.plist
```

So far the View, Model and ViewModel files are pretty barren, we'll come back to those later:

`___FILEBASENAME___.swift`:
{% highlight swift %}
//  ___FILEHEADER___

import Foundation

class ___FILEBASENAMEASIDENTIFIER___ {
  
}
{% endhighlight %}

`___FILEBASENAME___View.swift`:
{% highlight swift %}

//  ___FILEHEADER___

import UIKit

class ___FILEBASENAMEASIDENTIFIER___: UIView {
  
}
{% endhighlight %}

`___FILEBASENAME___ViewModel.swift`:
{% highlight swift %}
//  ___FILEHEADER___

import Foundation

class ___FILEBASENAMEASIDENTIFIER___ {
  
}
{% endhighlight %}

The PList will be a bit different from the PList of the basic template:
- We won't need the `DefaultCompletionName` and `AllowedTypes` keys, as we're not offering the option to chose a single file name anymore.
- We're introducing an `Options` key, which is an array of dictionaries. Each dictionary represents an option.

An "option dictionary" has the following keys:
                                                                                                                                        
| Key               | Description                                                                                       | Type            |
| ---               | ---                                                                                               | ---             |
| `Name`            | Label displayed next to the option.                                                               | String          |
| `Description`     | Description used as a tooltip when the mouse hovers over the option.                              | String          |
| `Type`            | The type of the option, possible values are: `checkbox`, `text`, `static`, `combo`, `popup`.      | String          |
| `Required`        | Whether the wizard should disable the Next button if this option doesn't have a value.            | Bool            |
| `Identifier`      | Uniquely identifies the option and is used to refer to the option's content as a variable.        | String          |
| `Default`         | Default value, can use variables from other options (using the strings `true` or `false`, or using the Bool values `<true/>` or `<false/>` doesn't seem to work with `checkbox` types, as the wizard seems to remember the last used value). | String |
| `Values`          | The possible values of a `combo` or `popup`. | Array of String |
| `RequiredOptions` | Can be used to enable the current option, only if certain values of another option are selected. For example, only enabling a checkbox for some values of another popup option. The key of the dictionary must be the identifier of the other option, and the value of the dictionary must be the array of subset of values from the `popup` or `combo` option that allow the current option to be enabled. | Dictionary |
| `SortOrder`       | Can be used to change the order in which the options are displayed, otherwise they are displayed in the order they are entered in the `Options` array. | Number |

As long as there's one option under the `Options` key, Xode will show a panel titled "Choose options for your new file".

![](/img/screenshot-3.png)

Let's add a name for the View/ViewModel. To do so we need to add the following option to the `Options` key, like so:

{% highlight xml %}
<key>Options</key>
<array>
  <dict>
    <key>Identifier</key>
    <string>productName</string>
    <key>Required</key>
    <true/>
    <key>Name</key>
    <string>View:</string>
    <key>Description</key>
    <string>The name of the View and ViewModel to create</string>
    <key>Type</key>
    <string>text</string>
    <key>Default</key>
    <string>MyView</string>
  </dict>
</array>
{% endhighlight %}

**⚠️ IMPORTANT:** Note the `identifier` of the option: For Xcode to know that it is the base name you want to use for the files (and to populate the `FILEBASENAME` and `FILEBASENAMEASIDENTIFIER` macro), you need to stick to **`productName`**.

The full PList may look like this now:
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Kind</key>
  <string>Xcode.IDEKit.TextSubstitutionFileTemplateKind</string>
  <key>Platforms</key>
  <array>
    <string>com.apple.platform.iphoneos</string>
  </array>
  <key>Options</key>
  <array>
    <dict>
      <key>Identifier</key>
      <string>productName</string>
      <key>Required</key>
      <true/>
      <key>Name</key>
      <string>View:</string>
      <key>Description</key>
      <string>The name of the View and ViewModel to create</string>
      <key>Type</key>
      <string>text</string>
      <key>Default</key>
      <string>MyView</string>
    </dict>
  </array>
</dict>
</plist>
{% endhighlight %}

With this you should see the following screen in Xcode when you select your template:

![](/img/screenshot-4.png)

*Note: The description appears if you hover over the label* 😉.

Now if you click next, you will see a save dialog, asking you to chose a location, but you won't be asked for a file name. The name is derived from the `productName` option.

Now let's amend a bit the source files to make them a bit more useful and let's start with the `___FILEBASENAME___ViewModel.swift` (again, this is for the sake of example, you may implement your ViewModels differently):
{% highlight swift %}
//  ___FILEHEADER___

import Foundation

class ___FILEBASENAMEASIDENTIFIER___ {

  private let model: ___VARIABLE_productName:identifier___

  init(with___VARIABLE_productName:identifier___ aModel: ___VARIABLE_productName:identifier___) {
      model = aModel
  
      // Add you own ViewModel logic for a ___VARIABLE_productName:identifier___.
      // ...
  }

}
{% endhighlight %}

When using the template, if the user types "Person" in the name field for instance, the files will be generated with the names:  `Person.swift`, `PersonView.swift` and `PersonViewModel.swift`, as expected.

However, in a rather confusing but also smart move, Xcode seems to be re-evaluating the `FILEBASENAME` macro (and `FILEBASENAMEASIDENTIFIER`) in context. Inside the file, the `FILEBASENAME` macro has the value "PersonViewModel", which is the actual final name of the file, and is pretty useful for nameing the class. If you want to have the raw input of what the user typed, the word "Person", you need to use the value of the `productName` variable.

Variables are based on the identifier of the "option" defined in the PList. For an option with identifier `optionName` you can access its value with `___VARIABLE_optionName___`. You can also add colon separated modifiers to this macro, all of this is explained in the [Xcode Text macro format reference](https://help.apple.com/xcode/mac/9.0/index.html?localePath=en.lproj#/devc8a500cb9). 

In our case we want a C identifier from the variable named `productName`, so we use `___VARIABLE_productName:identifier___`. Yep, it's a bit wild 😱.

*Note: I can't find a macro modifier to change the case of a variable, and I don't want to have a class member start with an uppercase character, that's why in this case I gave it a generic sounding name `model`. you could add another option to your template, to ask the user to give you a name for this variable* 🤷‍♀️.

In the same vein, we can amend the view with a function:
{% highlight swift %}
class ___FILEBASENAMEASIDENTIFIER___: UIView {

  func configure(withViewModel viewModel: ___VARIABLE_productName:identifier___ViewModel) {
    // configure the view with a ___VARIABLE_productName:identifier___ViewModel
  }

}
{% endhighlight %}

# <a name="code-variants" href="#code-variants">Code Variants</a>
Now let's add two options to ask the user of the template what type of View they want to use, and if they want to have a XIB file generated for them.

We'll first add a `popup` with the following choices:
- Storyboard
- XIB file
- Code

This is done with the following option:
{% highlight xml %}
<dict>
  <key>Identifier</key>
  <string>viewType</string>
  <key>Required</key>
  <true/>
  <key>Name</key>
  <string>View type:</string>
  <key>Description</key>
  <string>The type of view to use</string>
  <key>Type</key>
  <string>popup</string>
  <key>Default</key>
  <string>Storyboard</string>
  <key>Values</key>
  <array>
    <string>Storyboard</string>
    <string>XIB file</string>
    <string>Code</string>
  </array>
</dict>
{% endhighlight %}

Now we'll add a `checkbox` to offer to generate a XIB file. But this checkbox should only be active if the XIB file option is selected in the popup menu.

This is done with the following option:
{% highlight xml %}
<dict>
  <key>Identifier</key>
  <string>generateXIBfile</string>
  <key>Name</key>
  <string>Also create a XIB file</string>
  <key>Description</key>
  <string>Whether to create a XIB file with the same name</string>
  <key>Type</key>
  <string>checkbox</string>
  <key>RequiredOptions</key>
  <dict>
    <key>viewType</key>
    <array>
      <string>XIB file</string>
    </array>
  </dict>
</dict>
{% endhighlight %}

So now your PList should look like this:
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Kind</key>
  <string>Xcode.IDEKit.TextSubstitutionFileTemplateKind</string>
  <key>Platforms</key>
  <array>
    <string>com.apple.platform.iphoneos</string>
  </array>
  <key>Options</key>
  <array>
    <dict>
      <key>Identifier</key>
      <string>productName</string>
      <key>Required</key>
      <true/>
      <key>Name</key>
      <string>View:</string>
      <key>Description</key>
      <string>The name of the View and ViewModel to create</string>
      <key>Type</key>
      <string>text</string>
      <key>Default</key>
      <string>MyView</string>
    </dict>
    <dict>
      <key>Identifier</key>
      <string>viewType</string>
      <key>Required</key>
      <true/>
      <key>Name</key>
      <string>View type:</string>
      <key>Description</key>
      <string>The type of view to use</string>
      <key>Type</key>
      <string>popup</string>
      <key>Default</key>
      <string>Storyboard</string>
      <key>Values</key>
      <array>
        <string>Storyboard</string>
        <string>XIB file</string>
        <string>Code</string>
      </array>
    </dict>
    <dict>
      <key>Identifier</key>
      <string>generateXIBfile</string>
      <key>Name</key>
      <string>Also create a XIB file</string>
      <key>Description</key>
      <string>Whether to create a XIB file with the same name</string>
      <key>Type</key>
      <string>checkbox</string>
      <key>RequiredOptions</key>
      <dict>
        <key>viewType</key>
        <array>
          <string>XIB file</string>
        </array>
      </dict>
    </dict>
  </array>
</dict>
</plist>
{% endhighlight %}

And your template options panel should look like this:

![](/img/screenshot-5.png)

For this to actually work we need to go back to the files, and reorganise them. We need a subfolder for each variation in the `.xctemplate` folder. The subfolders need to follow a certain naming convention. For a `popup` (and I assume it works similarly for a `combo`, but I haven't tested), the subfolders need to match the name of the possible values of the popup, for a `checkbox`, it just needs to match the `identifier` of the checkbox. If you want to combine values, you just append them in the order they are presented to the user.

This works out to be the following folder structure for our template:
```
📂 View, Model & ViewModel.xctemplate
  📁 Code
    // Files relevant for a view setup in code
  📁 Storyboard
    // Files relevant for a storyboard view
  📁 XIB file
    // Files relevant for a XIB file based view, without a XIB file
  📁 XIB filegenerateXIBfile
    // Files relevant for a XIB file based view, with a XIB file
  📄 TemplateInfo.plist
```

I know, the folder `XIB filegenerateXIBfile` has a pretty ugly name 😨

With all those folders, you can have as many variations of the code, but not all files need to change. In this example, only our `___FILEBASENAME__View.swift` needs to change, and a XIB file needs to exist or not. The other two swift files don't need to change. So to avoid duplicating to much code, I prefer having a "base folder" with the invariant files and hard-linking them into each template folder. Xcode only looks at the folder with names that match the options names, so a `base` folder would be invisible and wouldn't impact the template.

To create a hard link, simply navigate in your terminal to the base folder, and use the following command: 

```
ln model.swift ___FILEBASENAME__.swift
```

**⚠️ IMPORTANT:** Once you have created a hard link to a base file, you can't copy/paste the same hardlinked file in other folders. You need to keep recreating "fresh" hard links.

Repeat the same steps for the ViewModel file, then simply paste a regular copy of the `___FILEBASENAME__View.swift` in each template subfolder. You can also create a basic XIB file in Xcode and include it in the ugly named folder `XIB filegenerateXIBfile`. In the XIB file you may want to edit the XML to add a `FILEBASENAMEASIDENTIFIER` macro to help link it to its base class. 

Once you've done all of this, you should end up with the following folder structure:
```
📂 View, Model & ViewModel.xctemplate
  📂 base
    📄 model.swift
    📄 viewModel.swift
  📂 Code
    📄 ___FILEBASENAME___.swift → model.swift
    📄 ___FILEBASENAME___View.swift
    📄 ___FILEBASENAME___ViewModel.swift → viewModel.swift
  📂 Storyboard
    📄 ___FILEBASENAME___.swift → model.swift
    📄 ___FILEBASENAME___View.swift
    📄 ___FILEBASENAME___ViewModel.swift → viewModel.swift
  📂 XIB file
    📄 ___FILEBASENAME___.swift → model.swift
    📄 ___FILEBASENAME___View.swift
    📄 ___FILEBASENAME___ViewModel.swift → viewModel.swift
  📂 XIB filegenerateXIBfile
    📄 ___FILEBASENAME___.swift → model.swift
    📄 ___FILEBASENAME___View.swift
    📄 ___FILEBASENAME___View.xib
    📄 ___FILEBASENAME___ViewModel.swift → viewModel.swift
  📄 TemplateInfo.plist
```

The customisation if the View files (different `init` for different ways of loading the view, depending on the type of view) and the XIB file is left as an exercise for the reader 😁. 

You can find the [source for this template on GitHub 🐙](https://github.com/jeanetienne/xcode-9-file-template-examples).

You now have a pretty advanced Xcode template that can generate several files with several variations of your own custom boilerplate. It even lets you choose if you want to generate a XIB file or not. That's pretty neat!

# <a name="sources" href="#sources">Sources</a>
When I first got interested in Xcode Templates I read a few blog articles, most of them were quite dated (from Xcode 4 or around). The best source for up to date information is to have a look at Xcode's default templates located in:

```
Xcode.app/Contents/Developer/Library/Xcode/Templates
or
Xcode.app/Contents/Developer/Platforms/<platform>/Developer/Library/Xcode/Templates/
```

An invaluable reference is [Steffen Itterheim's "Xcode 4 Template Documentation"](http://www.learn-cocos2d.com/store/xcode4-template-documentation). Steffen made this book free a few years ago, so you have no reason not to grab it. It's obviously a bit outdated, but is still very helpful. 

# <a name="comments" href="#comments">Comments</a>
Feel free to comment on [Hacker News](https://news.ycombinator.com/item?id=15244089) or reach me on [Twitter 🐦](https://twitter.com/jeanetienne)
