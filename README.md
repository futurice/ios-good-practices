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

Find more information about localization in [these presentation slides][l10n-slides] from the February 2012 HelsinkiOS meetup. Most of the talk is still relevant in October 2014.

[l10n-slides]: https://speakerdeck.com/hasseg/localization-practicum

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

#pragma mark - Public Interface

- (void)startFooing;
- (void)stopFooing;

#pragma mark - User Interaction

- (void)foobarButtonTapped;

#pragma mark - XYZFoobarDelegate

- (void)foobar:(Foobar *)foobar didSomethingWithFoo:(Foo *)foo;

#pragma mark - Internal Helpers

- (NSString *)displayNameForFoo:(Foo *)foo;
```

The most important point is to keep these consistent across your project's classes.

## Diagnostics

### Compiler warnings

It is recommended that you enable as many compiler warnings as possible, and treat warnings as errors. This recommendation is justified in [these presentation slides][warnings-slides]. The slides also contain information on how to suppress certain warnings in specific files, or in specific sections of code.

In short, add at least these values to the _“Other Warning Flags”_ build setting:

- `-Wall` _(Enables lots of additional warnings)_
- `-Wextra` _(Enables more additional warnings)_

Also enable the _“Treat warnings as errors”_ build setting.

[warnings-slides]: https://speakerdeck.com/hasseg/the-compiler-is-your-friend

### [Faux Pas](http://fauxpasapp.com/)

Created by our very own [Ali Rantakari](https://twitter.com/AliRantakari), Faux Pas is a fabulous static error detection tool. It analyzes your codebase and finds issues you had no idea even existed. Be sure to run this before shipping any iOS (or Mac) app!

### [Reveal] or [Spark Inspector]

These powerful visual inspectors will save you hours of time when debugging your views, especially if you're using Auto Layout (and you should). Xcode 6 will include [something very similar](http://www.cmenschel.de/xcode-6-view-debugging) for free, though, so maybe just hold off on that purchase until launch day.

[Reveal]: http://revealapp.com/
[Spark Inspector]: http://sparkinspector.com

## Analytics

Including some analytics framework in your app is strongly recommended, as it allows you to gain insights on how people actually use it. Does feature X add value? Is button Y too hard to find? To answer these, you can send events, timings and other measurable information to a service that aggregates and visualizes them – for instance, [Google Tag Manager](http://www.google.com/tagmanager/). The latter is more versatile than Google Analytics in that it inserts a data layer between app and Analytics, so that the data logic can be modified through a web service without having to update the app.

A good practice is to create a slim helper class, e.g. `XYZAnalyticsHelper`, that handles the translation from app-internal models and data formats (XYZModel, NSTimeInterval, …) to the mostly string-based data layer:

```objective-c

- (void)pushAddItemEventWithItem:(XYZItem *)item editMode:(XYZEditMode)editMode
{
    NSString *editModeString = [self nameForEditMode:editMode];

    [self pushToDataLayer:@{
        @"event": "addItem",
        @"itemIdentifier": item.identifier,
        @"editMode": editModeString
    }];
}

```

This has the additional advantage of allowing you to swap out the entire Analytics framework behind the scenes if needed, without the rest of the app noticing.

## Building

### Build Configurations

Even simple apps can be built in different ways. The most basic separation that Xcode gives you is that between _debug_ and _release_ builds. For the latter, there is a lot more optimization going on at compile time, at the expense of debugging possibilities. Apple suggests that you use the _debug_ build configuration for development, and create your App Store packages using the _release_ build configuration. This is codified in the default scheme (the dropdown next to the Play and Stop buttons in Xcode), which commands that _debug_ be used for Run and _release_ for Archive.

However, this is a bit too simple for real-world applications. You might – no, [_should!_](https://blog.futurice.com/five-environments-you-cannot-develop-without) – have different environments for testing, staging and other activities related to your service. Each might have their own base URL, log levels, bundle identifier (so you can install them side-by-side), provisinging profile and so on. Therefore a simple debug/release distinction won't cut it. You can add more build configurations on the "Info" tab of your project settings in Xcode.

#### `xcconfig` files for build settings

Typically build settings are specified in the Xcode GUI, but you can also use _configuration settings files_ (“`.xcconfig` files”) for them. The benefits of using these are:

- You can add comments to explain things
- You can `#include` other build settings files, which helps you avoid repeating yourself:
    - If you have some settings that apply to all build configurations, add a `Common.xcconfig` and `#include` it in all the other files
    - If you e.g. want to have a “Debug” build configuration that enables compiler optimizations, you can just `#include "MyApp_Debug.xcconfig"` and override one of the settings

Find more information about this topic in [these presentation slides][xcconfig-slides].

[xcconfig-slides]: https://speakerdeck.com/hasseg/xcode-configuration-files

### Targets

A target resides conceptually below the project level, i.e. a project can have several targets that may override its project settings. Roughly, each target corresponds to "an app" within the context of your codebase. For instance, you could have country-specific apps (built from the same codebase) for different countries' App Stores. Each of these will need development/staging/release builds, so it's better to handle those through build configurations, not targets. It's not uncommon at all for an app to only have a single target.

## Schemes

Schemes tell Xcode what should happen when you hit the Run, Test, Profile, Analyze or Archive action. Basically, they map each of these actions to a target and a build configuration. You can also pass launch arguments, such as the language the app should run in (handy for testing your localizations!) or set some diagnostic flags for debugging.

A suggested naming convention for schemes is `MyApp (<Language>) [Environment]`:

    MyApp (English) [Development]
    MyApp (German) [Development]
    MyApp [Testing]
    MyApp [Staging]
    MyApp [App Store]

For most environments the language is not needed, as the app will probably be installed through other means than Xcode, e.g. TestFlight, and the launch argument thus be ignored anyway. In that case, the device language should be set manually to test localization.

## Deployment

Deploying software for iOS is a pain. That being said, there are some things to know that will help you tremendously with it.

Whenever you want to run software on an actual device (as opposed to the simulator), you will need to sign your build with a __certificate__ issued by Apple. Each certificate is linked to a private/public keypair, the private half of which resides in your Mac's Keychain. There are two types of certificates:

* __Development certificate:__ Every developer on a team has their own, and it is generated upon request. Xcode might do this for you, but it's better not to press the magic "Fix issue" button and understand what is actually going on. This certificate is needed to deploy development builds to devices.
* __Distribution certificate:__ There can be several, but it's best to keep it to one per organization, and share its associated key through some internal channel. This certificate is needed to ship to the App Store, or your organization's internal "enterprise app store".

Besides certificates, there are also __provisioning profiles__, which are basically the missing link between devices and certificates. Again, there are two types to distinguish between development and distribution purposes:

* __Development provisioning profile:__ It contains a list of all devices that the device is authorized to be built on. It is also linked to one or more development certificates, one for each developer that is allowed to use the profile. The profile can be tied to a specific app, but for most development purposes it's perfectly fine to use the wildcard profile, whose App ID ends in an asterisk (*).

* __Distribution provisioning profile:__ There are three different ways of distribution, each for a different use case. Each distribution profile is linked to a distribution certificate, and will be invalid when the certificate expires.
    * __Ad-Hoc:__ Just like development profiles, it contains a whitelist of devices the app can be installed to. This type of profile is used for beta testing (e.g. TestFlight). Note that due to Apple's acquisition of TestFlight, this is likely to change in late 2014.
    * __App Store:__ This profile has no list of allowed devices, as anyone can install it through Apple's official distribution channel. This profile is required for all App Store releases.
    * __Enterprise:__ Just like App Store, there is no device whitelist, and the app can be installed by anyone with access to the enterprise's internal "app store", which can be just a website with links. This profile is available only on Enterprise accounts.

To sync all certificates and profiles to your machine, go to Accounts in Xcode's Preferences, add your Apple ID if needed, and double-click your team name. There is a refresh button at the bottom, but sometimes you just need to restart Xcode to make everything show up.

## More Ideas

- [https://github.com/vsouza/awesome-ios](https://github.com/vsouza/awesome-ios)
- Update for Xcode 6
    - No automatic precompiled header
- Pod usage: `pod install` vs `pod update`
