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

#### Localization

Keep all user strings in localization files right from the beginning. This is good not only for translations, but also for finding user-facing text quickly. You can add a launch argument to your build scheme to launch the app in a certain language, e.g.

    -AppleLanguages (Finnish)

For more complex translations such as plural forms that depending on a number of items (e.g. "1 person" vs. "3 people"), you should use the [`.stringsdict` format](https://developer.apple.com/library/prerelease/ios/documentation/MacOSX/Conceptual/BPInternational/StringsdictFileFormat/StringsdictFileFormat.html) instead of a regular `localizable.strings` file. As soon as you've wrapped your head around the crazy syntax, you have a powerful tool that knows how to make plurals for "one", some", "few" and "many" items, as needed [e.g. in Russian or Arabic](http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html).

#### Constants

Keep app-wide constants in a `Constants.h` file that is included in the prefix header. For constants, use the syntax

    static CGFloat const XYZBrandingFontSizeSmall = 12.0f;

instead of `#define` for better type safety.


### Branching Model

Especially when distributing an app to the public (e.g. through the App Store), it's a good idea to isolate releases to their own branch with proper tags. Also, feature work that involves a lot of commits should be done on its own branch. [`git-flow`](https://github.com/nvie/gitflow) is a tool that helps you follow these conventions. It is simply a convenience wrapper around Git's branching and tagging commands, but can help maintain a proper branching structure especially for teams. Do all development on feature branches (or on `develop` for smaller work), tag releases with the app version, and commit to master only via

    git flow release finish <version>

## Common Libraries

### AFNetworking

A perceived 99.95 percent of iOS developers use this network library. Sure, iOS 7's `NSURLSession` brought some nice improvements to the rather dated native networking APIs, but `AFNetworking` is still unbeaten when it comes to actually managing a queue of requests, which is pretty much a requirement in any modern app.

### DateTools
As a general rule, don't write your date calculations yourself. [(Here's why.)](https://www.youtube.com/watch?v=-5wpm-gesOY) Luckily, in DateTools you get an MIT-licensed, thoroughly tested library that covers pretty much all your calendary needs.

### [Masonry](https://www.github.com/Masonry/Masonry) and Friends
If you prefer to write your views in code, chances are you've met either of Apple's awkward syntaxes – the regular 'NSLayoutConstraint' factory or the so-called [Visual Format Language](https://developer.apple.com/library/ios/documentation/userexperience/conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH3-SW1). The former is extremely verbose and the latter based on strings, which effectively prevents compile-time checking.

[Masonry](https://www.github.com/Masonry/Masonry) remedies this by introducing its own DSL to make, update and replace constraints. A similar approach for Swift is taken by [Cartography](https://github.com/robb/Cartography), which builds on the language's powerful operator overloading features. For the more conservative, [FLKAutoLayout](https://github.com/floriankugler/FLKAutoLayout) offers a clean, but rather non-magical wrapper around the native APIs.

## Architecture

* [Model-View-Controller-Store (MVCS)](http://programmers.stackexchange.com/questions/184396/mvcs-model-view-controller-store)
    * Stores handle all networking, cache data etc.
    * Expose either `RACSignal`s or void-returning methods with custom completion blocks
* [Model-View-ViewModel (MVVM)](http://www.objc.io/issue-13/mvvm.html)
    * Quite new concept for Cocoa developers, but gaining traction

### Common Patterns

* __Delegation:__ Apple uses this a lot (some would say, too much). Use when you want to communicate stuff back e.g. from a modal view.
* __Callback blocks:__ Allow for a more loose coupling, while keeping related code sections close to each other. Also scales better than delegation when there are many senders.
* __Signals:__ The centerpiece of [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa), they allow chaining and combining to your heart's content, thereby offering a way out of [callback hell](http://elm-lang.org/learn/Escape-from-Callback-Hell.elm).

### Models

Keep your models immutable, and use them to translate the remote API's semantics and types to your app. Github's [Mantle](https://github.com/Mantle/Mantle) is a good choice.


### Controllers

Use dependency injection instead of keeping all state around in singletons. The latter is okay only if that state _really_ is global.

```objective-c
+ [[FooDetailsViewController alloc] initWithFoo:(Foo *)foo];
```

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

## Assets

[Asset catalogs](https://developer.apple.com/library/ios/recipes/xcode_help-image_catalog-1.0/Recipe.html) are the best way to manage all your project's visual assets. They can hold both universal and device-specific (iPhone 4-inch, iPhone Retina, iPad, etc.) assets and will automatically serve the correct ones for a given name. Teaching your designer(s) how to add and commit things there (Xcode has its own built-in Git client) can save a lot of time that would otherwise be spent copying stuff from emails or other channels to the codebase. It also allows them to instantly try out their changes and iterate if needed.

Asset catalogs expose only the names of image sets, abstracting away the actual file names within the set. This nicely prevents asset name conflicts, as files such as `button_large@2x.png` are now namespaced inside their image sets. However, some discipline when naming assets can make life easier:

```objective-c
IconCheckmarkHighlighted.png // Universal, non-Retina
IconCheckmarkHighlighted@2x.png // Universal, Retina
IconCheckmarkHighlighted~iphone.png // iPhone, non-Retina
IconCheckmarkHighlighted@2x~iphone.png // iPhone, Retina
IconCheckmarkHighlighted-568h@2x~iphone.png // iPhone, Retina, 4-inch
IconCheckmarkHighlighted~ipad.png // iPad, non-Retina
IconCheckmarkHighlighted@2x~ipad.png // iPad, Retina
```

The modifiers `-568h`, `@2x`, `~iphone` and `~ipad` are not required per se, but having them in the file name when dragging the file to an image set will automatically place them in the right "slot", thereby preventing assignment mistakes that can be hard to hunt down.

## Coding Style

### Structure

[Pragma marks](http://nshipster.com/pragma/) are a great way to group your methods, especially in view controllers. Here is a common structure that works with almost any view controller:

```objective-c
#pragma mark - Lifecycle

- (instancetype)initWithFoo:(Foo *)foo;
- (void)dealloc;

#pragma mark - View Lifecycle

- (void)viewDidLoad;
- (void)viewWillAppear:(BOOL)animated;

#pragma mark - Auto Layout

- (void)makeViewConstraints;

#pragma mark - User Interaction

- (void)foobarButtonTapped;

#pragma mark - XYZFoobarDelegate

- (void)foobar:(Foobar *)foobar didSomethingWithFoo:(Foo *)foo;

#pragma mark - Internal Helpers

- (NSString *)displayNameForFoo:(Foo *)foo;
```

The most important point is to keep these consistent across your project's classes.

## Diagnostics

### [FauxPas](http://fauxpasapp.com/)

Created by our very own [Ali Rantakari](https://twitter.com/AliRantakari), Faux Pas is a fabulous static error detection tool. It analyzes your codebase and finds issues you had no idea even existed. Be sure to run this before shipping any iOS (or Mac) app!

### [Reveal](http://revealapp.com/)

This expensive, but powerfool visual inspector will save you hours of time when debugging your views, especially if you're using Auto Layout (and you should). Xcode 6 will include [something very similar](http://www.cmenschel.de/xcode-6-view-debugging) for free, though, so maybe just hold off on that purchase until launch day.

## More Ideas

- Certificates and provisioning profiles (keep it hands-on)
- Analytics, Tag manager
