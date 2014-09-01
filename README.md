iOS Good Practices
==================

_Just like software, this document will rot unless taken care of. To keep the threshold of updating as low as possible, you can edit it the way every developer loves: through version control._

## Why?

Getting on board with iOS can be intimidating. A weird language, own names for almost everything – not to mention the bumpy road for your code to actually make it onto the device. This living document is here to help you, whether you're taking your first steps in Cocoaland or you're curious about doing things "the right way". Everything below is just suggestions, so if you have a good reason to do something differently, by all means go for it!

## Getting Started

### Xcode

[Xcode](https://developer.apple.com/xcode/) is the IDE of choice for most iOS developers, and the only one officially supported by Apple. There are some alternatives, but unless you're already a seasoned iOS person, go with Xcode. It's actually quite usable nowadays.

To install, simply download [Xcode on the Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835). It comes with the newest SDK and simulators, and you can install more stuff under _Preferences > Downloads_.

### Project Setup

A common question when beginning an iOS project is whether to write all views in code or use Interface Builder with Storyboards or XIB files. Both are known to occasionally result in working software. However, there are a few considerations:

#### Why code?
* In Xcode 5, universal apps require separate Storyboards for iPhone and iPad. This can result in a lot of needless duplication. Xcode 6 fixes this by introducing [Size Classes](http://blog.futurice.com/adaptive-view-ios8).
* In Xcode 5, custom fonts and UI elements cannot be represented visually in Storyboards, but will have a generic appearance instead. Again, this changes in Xcode 6.
* Storyboards are more prone to version conflicts due to their complex XML structure.

#### Why Storyboards?
* For the less technically inclined, Storyboards can be a great way to contribute to the project directly, e.g. by tweaking colors or layout constraints. However, this requires a working project setup and some time to learn the basics.

### Ignores

A good first step when putting a project under version control is to have a decent `.gitignore` file. That way, unwanted files (user settings, temporary files, etc.) will never even make it into your repository. Luckily, GitHub has us covered for both [Objective-C](https://github.com/github/gitignore/blob/master/Objective-C.gitignore) and [Swift](https://github.com/github/gitignore/blob/master/Swift.gitignore).

### CocoaPods

If you're planning on including external dependencies (e.g. third-party libraries) in your project, [CocoaPods](http://www.cocoapods.org) is the way to go. Install it like so:

    sudo gem install cocoapods

To get started, move inside your iOS project folder and run

    pod init

This creates a Podfile, which will hold all your dependencies in one place. You can then

    pod install

to install these dependencies and include them as part of a workspace which also holds your own project. Note that from now on, you'll need to open the `.xcworkspace` file instead of `.xcproject`, or your code will not compile.

### Project Structure

To keep all those hundreds of source files ending up in the same directory, it's a good idea to set up some folder structure depending on your architecture. For instance, you can use the following:

    ├─ Models
    ├─ Views
    ├─ Controllers
    ├─ Stores
    ├─ Helpers

First, create them as groups (little yellow "folders") within the group with your project's name in Xcode's Project Navigator. Then, for each of the groups, link them to an actual directory in your project path by opening their File Inspector on the right, hitting the little gray folder icon, and creating a new subfolder with the name of the group in your project directory.

Keep all user strings in localization files right from the beginning. This is good not only for translations, but also for finding user-facing text quickly. You can add a launch argument to your build scheme to launch the app in a certain language, e.g.

    -AppleLanguages (Finnish)

Keep app-wide constants in a `Constants.h` file that is included in the prefix header. For constants, use the syntax

    static CGFloat const XYZBrandingFontSizeSmall = 12.0f;

instead of `#define` for better type safety.


### Branching Model

Especially when distributing an app to the public (e.g. through the App Store), it's a good idea to isolate releases to their own branch with proper tags. Also, feature work that involves a lot of commits should be done on its own branch. [`git-flow`](https://github.com/nvie/gitflow) is a tool that helps you follow these conventions. It is simply a convenience wrapper around Git's branching and tagging commands, but can help maintain a proper branching structure especially for teams. Do all development on feature branches (or on `develop` for smaller work), tag releases with the app version, and commit to master only via

    git flow release finish <version>

## Common Libraries

### AFNetworking

A perceived 99.95 percent of iOS developers use this network library. Sure, iOS 7's `NSURLSession` brought some nice improvements to the rather dated native networking APIs, but `AFNetworking` is still unbeaten when it comes to actually manage a queue of requests, which is pretty much a requirement in any modern app.

### DateTools
As a general rule, don't write your date calculations yourself. [(Here's why.)](https://www.youtube.com/watch?v=-5wpm-gesOY) Luckily, in DateTools you get an MIT-licensed, thoroughly tested library that covers pretty much all your calendary needs.

### [Masonry](https://www.github.com/Masonry/Masonry) and Friends
If you prefer to write your views in code, chances are you've met either of Apple's awkward syntaxes – the regular 'NSLayoutConstraint' factory or the so-called [Visual Format Language](https://developer.apple.com/library/ios/documentation/userexperience/conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH3-SW1). The former is extremely verbose and the latter based on strings, which effectively prevents compile-time checking.

[Masonry](https://www.github.com/Masonry/Masonry) remedies this by introducing its own DSL to make, update and replace constraints. A similar approach for Swift is taken by [Cartography](https://github.com/robb/Cartography), which builds on the language's powerful operator overloading features. For the more conservative, [FLKAutoLayout](https://github.com/floriankugler/FLKAutoLayout) offers a clean, but rather non-magical wrapper around the native APIs.

## Architecture

* Model-View-Controller-Store (MVCS)
    * Stores handle all networking, can cache etc.
    * Expose either `RACSignal`s or void-returning methods with custom completion blocks
* MVVM, if you're brave
    * No own project experience yet

### Models

Keep your models immutable, and use them to translate the remote API's semantics and types to your app. Github's [Mantle](https://github.com/Mantle/Mantle) is a good choice.


### Controllers

Use dependency injection instead of keeping all state around in singletons. The latter is okay only if that state _really_ is global.

    + [[ModelDetailsViewController alloc] initWithModel:(Model *)model];

## Networking

### Traditional way: Use custom callback blocks

```objective-c
// GigStore.h

typedef void (^FetchGigsBlock)(NSArray *gigs, NSError *error);

- (void)fetchGigsForArtist:(Artist *)artist completion:(FetchGigsBlock)completion


// GigsViewController.m

[[GigStore sharedStore] fetchGigsForArtist:artist completion:^(NSArray *gigs, NSError *error) {
    if (!error) {
        // Do something with gigs
    }
    else {
        // :(
    }
];
```

This works, but can quickly lead to callback hell if you need to chain multiple requests. In that case, have a look at [ReactiveCocoa (RAC)](https://www.github.com/ReactiveCocoa/ReactiveCocoa). It's a versatile and multi-purpose library that can change the way people write [entire apps](https://github.com/jspahrsummers/GroceryList), but you can also use it sparingly where it fits the task.

There are good introductions to the concept of RAC (and FRP in general) on [Teehan+Lax](http://www.teehanlax.com/blog/getting-started-with-reactivecocoa/) and [NSHipster](http://nshipster.com/reactivecocoa/).

### Reactive way: Use RAC signals

```objective-c
// GigStore.h

- (RACSignal *)gigsForArtist:(Artist *)artist;


// GigsViewController.m

[[GigStore sharedStore] gigsForArtist:artist]
    subscribeNext:^(NSArray *gigs) {
        // Do something with gigs
    } error:^(NSError *error) {
        // :(
    }
];
```

This allows us to transform or filter gigs before showing them, by combining the gig signal with other signals.

## More Ideas

- Architectural ideas, patterns (e.g. callbacks, delegation, signals)
- FauxPas, Reveal
- Asset naming for designers
- Certificates and provisioning profiles (keep it hands-on)
- Analytics, Tag manager
- Structure of code (pragmas, order of stuff)
