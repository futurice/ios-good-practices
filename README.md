iOS Good Practices
==================

_Just like software, this document will rot unless we take care of it. We encourage everyone to help us on that – just open an issue or send a pull request!_

Interested in other mobile platforms? Our [Best Practices in Android Development][android-best-practices] and [Windows App Development Best Practices][windows-app-development-best-practices] documents have got you covered.

[android-best-practices]: https://github.com/futurice/android-best-practices
[windows-app-development-best-practices]: https://github.com/futurice/windows-app-development-best-practices

## Why?

Getting on board with iOS can be intimidating. Neither Swift nor Objective-C are widely used elsewhere, the platform has its own names for almost everything, and it's a bumpy road for your code to actually make it onto a physical device. This living document is here to help you, whether you're taking your first steps in Cocoaland or you're curious about doing things "the right way". Everything below is just suggestions, so if you have a good reason to do something differently, by all means go for it!

## Contents

If you are looking for something specific, you can jump right into the relevant section from here.

1. [Getting Started](#getting-started)
1. [Common Libraries](#common-libraries)
1. [Architecture](#architecture)
1. [Networking](#networking)
1. [Assets](#assets)
1. [Coding Style](#coding-style)
1. [Diagnostics](#diagnostics)
1. [Analytics](#analytics)
1. [Building](#building)
1. [Deployment](#deployment)
1. [In-App Purchases (IAP)](#in-app-purchases-iap)
1. [License](#license)

## Getting Started

### Xcode

[Xcode][xcode] is the IDE of choice for most iOS developers, and the only one officially supported by Apple. There are some alternatives, of which [AppCode][appcode] is arguably the most famous, but unless you're already a seasoned iOS person, go with Xcode. Despite its shortcomings, it's actually quite usable nowadays!

To install, simply download [Xcode on the Mac App Store][xcode-app-store]. It comes with the newest SDK and simulators, and you can install more stuff under _Preferences > Downloads_.

[xcode]: https://developer.apple.com/xcode/
[appcode]: https://www.jetbrains.com/objc/
[xcode-app-store]: https://itunes.apple.com/us/app/xcode/id497799835

### Project Setup

A common question when beginning an iOS project is whether to write all views in code or use Interface Builder with Storyboards or XIB files. Both are known to occasionally result in working software. However, there are a few considerations:

#### Why code?
* Storyboards are more prone to version conflicts due to their complex XML structure. This makes merging much harder than with code.
* It's easier to structure and reuse views in code, thereby keeping your codebase [DRY][dry].
* All information is in one place. In Interface Builder you have to click through all the inspectors to find what you're looking for.

[dry]: http://en.wikipedia.org/wiki/Don%27t_repeat_yourself

#### Why Storyboards?
* For the less technically inclined, Storyboards can be a great way to contribute to the project directly, e.g. by tweaking colors or layout constraints. However, this requires a working project setup and some time to learn the basics.
* Iteration is often faster since you can preview certain changes without building the project.
* In Xcode 6, custom fonts and UI elements are finally represented visually in Storyboards, giving you a much better idea of the final appearance while designing.
* Starting with iOS 8, [Size Classes][size-classes] allow you to design for different device types and screens without duplication.

[size-classes]: http://blog.futurice.com/adaptive-view-ios8

### Ignores

A good first step when putting a project under version control is to have a decent `.gitignore` file. That way, unwanted files (user settings, temporary files, etc.) will never even make it into your repository. Luckily, GitHub has us covered for both [Swift][swift-gitignore] and [Objective-C][objc-gitignore].

[swift-gitignore]: https://github.com/github/gitignore/blob/master/Swift.gitignore
[objc-gitignore]: https://github.com/github/gitignore/blob/master/Objective-C.gitignore

### CocoaPods

If you're planning on including external dependencies (e.g. third-party libraries) in your project, [CocoaPods][cocoapods] offers easy and fast integration. Install it like so:

    sudo gem install cocoapods

To get started, move inside your iOS project folder and run

    pod init

This creates a Podfile, which will hold all your dependencies in one place. After adding your dependencies to the Podfile, you run

    pod install

to install the libraries and include them as part of a workspace which also holds your own project. It is generally [recommended to commit the installed dependencies to your own repo][committing-pods], instead of relying on having each developer running `pod install` after a fresh checkout.

Note that from now on, you'll need to open the `.xcworkspace` file instead of `.xcproject`, or your code will not compile. The command

    pod update

will update all pods to the newest versions permitted by the Podfile. You can use a wealth of [operators][cocoapods-pod-syntax] to specify your exact version requirements.

[cocoapods]: http://www.cocoapods.org
[cocoapods-pod-syntax]: http://guides.cocoapods.org/syntax/podfile.html#pod
[committing-pods]: https://www.dzombak.com/blog/2014/03/including-pods-in-source-control.html

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

For more complex translations such as plural forms that depending on a number of items (e.g. "1 person" vs. "3 people"), you should use the [`.stringsdict` format][stringsdict-format] instead of a regular `localizable.strings` file. As soon as you've wrapped your head around the crazy syntax, you have a powerful tool that knows how to make plurals for "one", some", "few" and "many" items, as needed [e.g. in Russian or Arabic][language-plural-rules].

Find more information about localization in [these presentation slides][l10n-slides] from the February 2012 HelsinkiOS meetup. Most of the talk is still relevant in October 2014.

[stringsdict-format]: https://developer.apple.com/library/prerelease/ios/documentation/MacOSX/Conceptual/BPInternational/StringsdictFileFormat/StringsdictFileFormat.html
[language-plural-rules]: http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html
[l10n-slides]: https://speakerdeck.com/hasseg/localization-practicum

#### Constants

Keep app-wide constants in a `Constants.h` file that is included in the prefix header.

Instead of preprocessor macro definitions (via `#define`), use actual constants:

    static CGFloat const XYZBrandingFontSizeSmall = 12.0f;
    static NSString * const XYZAwesomenessDeliveredNotificationName = @"foo";

Actual constants are type-safe, have more explicit scope (they’re not available in all imported/included files until undefined), cannot be redefined or undefined in later parts of the code, and are available in the debugger.


### Branching Model

Especially when distributing an app to the public (e.g. through the App Store), it's a good idea to isolate releases to their own branch with proper tags. Also, feature work that involves a lot of commits should be done on its own branch. [`git-flow`][gitflow-github] is a tool that helps you follow these conventions. It is simply a convenience wrapper around Git's branching and tagging commands, but can help maintain a proper branching structure especially for teams. Do all development on feature branches (or on `develop` for smaller work), tag releases with the app version, and commit to master only via

    git flow release finish <version>

[gitflow-github]: https://github.com/nvie/gitflow

## Common Libraries

Generally speaking, make it a conscious decision to add an external dependency to your project. Sure, this one neat library solves your problem now, but maybe later gets stuck in maintenance limbo, with the next OS version that breaks everything being just around the corner. Another scenario is that a feature only achievable with external libraries suddenly becomes part of the official APIs. In a well-designed codebase, switching out the implementation is a small effort that pays off quickly. Always consider solving the problem using Apple's extensive (and mostly excellent) frameworks first!

Therefore this section has been deliberately kept rather short. The libraries featured here tend to reduce boilerplate code (e.g. Auto Layout) or solve complex problems that require extensive testing, such as date calculations. As you become more proficient with iOS, be sure to dive into the source here and there, and acquaint yourself with their underlying Apple frameworks. You'll find that those alone can do a lot of the heavy lifting.

### AFNetworking

A perceived 99.95 percent of iOS developers use this network library. While `NSURLSession` is surprisingly powerful by itself, [AFNetworking][afnetworking-github] remains unbeaten when it comes to actually managing a queue of requests, which is pretty much a requirement in any modern app.

[afnetworking-github]: https://github.com/AFNetworking/AFNetworking

### DateTools
As a general rule, [don't write your date calculations yourself][timezones-youtube]. Luckily, in [DateTools][datetools-github] you get an MIT-licensed, thoroughly tested library that covers pretty much all your calendary needs.

[timezones-youtube]: https://www.youtube.com/watch?v=-5wpm-gesOY
[datetools-github]: https://github.com/MatthewYork/DateTools

### Auto Layout Libraries
If you prefer to write your views in code, chances are you've met either of Apple's awkward syntaxes – the regular 'NSLayoutConstraint' factory or the so-called [Visual Format Language][visual-format-language]. The former is extremely verbose and the latter based on strings, which effectively prevents compile-time checking.

[Masonry][masonry-github] remedies this by introducing its own DSL to make, update and replace constraints. A similar approach for Swift is taken by [Cartography][cartography-github], which builds on the language's powerful operator overloading features. For the more conservative, [FLKAutoLayout][flkautolayout-github] offers a clean, but rather non-magical wrapper around the native APIs.

[visual-format-language]: https://developer.apple.com/library/ios/documentation/userexperience/conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH3-SW1
[masonry-github]: https://www.github.com/Masonry/Masonry
[cartography-github]: https://github.com/robb/Cartography
[flkautolayout-github]: https://github.com/floriankugler/FLKAutoLayout

## Architecture

* [Model-View-Controller-Store (MVCS)][mvcs]
    * This is the default Apple architecture (MVC), extended by a Store layer that vends Model instances and handles the networking, caching etc.
    * Every Store exposes to the view controllers either `RACSignal`s or `void`-returning methods with custom completion blocks
* [Model-View-ViewModel (MVVM)][mvvm]
    * Motivated by "massive view controllers": MVVM considers `UIViewController` subclasses part of the View and keeps them slim by maintaining all state in the ViewModel
    * Quite new concept for Cocoa developers, but [gaining][cocoasamurai-rac] [traction][raywenderlich-mvvm]
* [View-Interactor-Presenter-Entity-Routing (VIPER)][viper]
    * Rather exotic architecture that might be worth looking into in larger projects, where even MVVM feels too cluttered and testability is a major concern

[mvcs]: http://programmers.stackexchange.com/questions/184396/mvcs-model-view-controller-store
[mvvm]: http://www.objc.io/issue-13/mvvm.html
[cocoasamurai-rac]: http://cocoasamurai.blogspot.de/2013/03/basic-mvvm-with-reactivecocoa.html
[raywenderlich-mvvm]: http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1
[viper]: http://www.objc.io/issue-13/viper.html

### “Event” Patterns

These are the idiomatic ways for components to notify others about things:

* __Delegation:__ _(one-to-one)_ Apple uses this a lot (some would say, too much). Use when you want to communicate stuff back e.g. from a modal view.
* __Callback blocks:__ _(one-to-one)_ Allow for a more loose coupling, while keeping related code sections close to each other. Also scales better than delegation when there are many senders.
* __Notification Center:__ _(one-to-many)_ Possibly the most common way for objects to emit “events” to multiple observers. Very loose coupling — notifications can even be observed globally without reference to the dispatching object.
* __Key-Value Observing (KVO):__ _(one-to-many)_ Does not require the observed object to explicitly “emit events” as long as it is _Key-Value Coding (KVC)_ compliant for the observed keys (properties). Usually not recommended due to its implicit nature and the cumbersome standard library API.
* __Signals:__ _(one-to-many)_ The centerpiece of [ReactiveCocoa][reactivecocoa-github], they allow chaining and combining to your heart's content, thereby offering a way out of [callback hell][elm-escape-from-callback-hell].

[elm-escape-from-callback-hell]: http://elm-lang.org/learn/Escape-from-Callback-Hell.elm

### Models

Keep your models immutable, and use them to translate the remote API's semantics and types to your app. Github's [Mantle](https://github.com/Mantle/Mantle) is a good choice.

### Views

When laying out your views using Auto Layout, be sure to add the following to your class:

    + (BOOL)requiresConstraintBasedLayout
    {
        return YES;
    }

Otherwise you may encounter strange bugs when the system doesn't call `-updateConstraints` as you would expect it to.

### Controllers

Use dependency injection, i.e. pass any required objects in as parameters, instead of keeping all state around in singletons. The latter is okay only if the state _really_ is global.

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

This works, but can quickly lead to callback hell if you need to chain multiple requests.

### Reactive way: Use RAC signals

If you find yourself in callback hell, have a look at [ReactiveCocoa (RAC)][reactivecocoa-github]. It's a versatile and multi-purpose library that can change the way people write [entire apps][groceryList-github], but you can also use it sparingly where it fits the task.

There are good introductions to the concept of RAC (and FRP in general) on [Teehan+Lax][teehan-lax-rac] and [NSHipster][nshipster-rac].

[grocerylist-github]: https://github.com/jspahrsummers/GroceryList
[teehan-lax-rac]: http://www.teehanlax.com/blog/getting-started-with-reactivecocoa/
[nshipster-rac]: http://nshipster.com/reactivecocoa/

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

[Asset catalogs][asset-catalogs] are the best way to manage all your project's visual assets. They can hold both universal and device-specific (iPhone 4-inch, iPhone Retina, iPad, etc.) assets and will automatically serve the correct ones for a given name. Teaching your designer(s) how to add and commit things there (Xcode has its own built-in Git client) can save a lot of time that would otherwise be spent copying stuff from emails or other channels to the codebase. It also allows them to instantly try out their changes and iterate if needed.

[asset-catalogs]: https://developer.apple.com/library/ios/recipes/xcode_help-image_catalog-1.0/Recipe.html

### Using Bitmap Images

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

### Using Vector Images

You can include the original [vector graphics (PDFs)][vector-assets] produced by designers into the asset catalogs, and have Xcode automatically generate the bitmaps from that. This reduces the complexity of your project (the number of files to manage.)

[vector-assets]: http://martiancraft.com/blog/2014/09/vector-images-xcode6/

## Coding Style

### Naming

Apple pays great attention to keeping naming consistent, if sometimes a bit verbose, throughout their APIs. When developing for Cocoa, you make it much easier for new people to join the project if you follow [Apple's naming conventions][cocoa-coding-guidelines].

[cocoa-coding-guidelines]: https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html

Here are some basic takeaways you can start using right away:

A method beginning with a _verb_ indicates that it performs some side effects, but won't return anything:
`- (void)loadView;`
`- (void)startAnimating;`

Any method starting with a _noun_, however, returns that object and should do so without side effects:
`- (UINavigationItem *)navigationItem;`
`+ (UILabel *)labelWithText:(NSString *)text;`

It pays off to keep these two as separated as possible, i.e. not perform side effects when you transform data, and vice versa. That will keep your side effects contained to smaller sections of the code, which makes it more understandable and facilitates debugging.

### Structure

[Pragma marks](http://nshipster.com/pragma/) are a great way to group your methods, especially in view controllers. Here is a common structure that works with almost any view controller:

```objective-c

#import "SomeModel.h"
#import "SomeView.h"
#import "SomeController.h"
#import "SomeStore.h"
#import "SomeHelper.h"
#import <SomeExternalLibrary/SomeExternalLibraryHeader.h>

static NSString * const XYZFooStringConstant = @"FoobarConstant";
static CGFloat const XYZFooFloatConstant = 1234.5;

@interface XYZFooViewController () <XYZBarDelegate>

@property (nonatomic, copy, readonly) Foo *foo;

@end

@implementation XYZFooViewController

#pragma mark - Lifecycle

- (instancetype)initWithFoo:(Foo *)foo;
- (void)dealloc;

#pragma mark - View Lifecycle

- (void)viewDidLoad;
- (void)viewWillAppear:(BOOL)animated;

#pragma mark - Layout

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

@end
```

The most important point is to keep these consistent across your project's classes.

### External Style Guides

Futurice does not have company-level guidelines for coding style. It can however be useful to peruse the style guides of other software companies, even if some bits can be quite company-specific or opinionated.

* GitHub: [Swift](https://github.com/github/swift-style-guide) and [Objective-C](https://github.com/github/objective-c-style-guide)
* Ray Wenderlich: [Swift](https://github.com/raywenderlich/swift-style-guide) and [Objective-C](https://github.com/raywenderlich/objective-c-style-guide)
* Google: [Objective-C](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml)
* The New York Times: [Objective-C](https://github.com/NYTimes/objective-c-style-guide)
* Sam Soffes: [Objective-C](https://gist.github.com/soffes/812796)
* Luke Redpath: [Objective-C](http://lukeredpath.co.uk/blog/2011/06/28/my-objective-c-style-guide/)

## Diagnostics

### Compiler warnings

It is recommended that you enable as many compiler warnings as possible, and treat warnings as errors. This recommendation is justified in [these presentation slides][warnings-slides]. The slides also contain information on how to suppress certain warnings in specific files, or in specific sections of code.

In short, add at least these values to the _“Other Warning Flags”_ build setting:

- `-Wall` _(Enables lots of additional warnings)_
- `-Wextra` _(Enables more additional warnings)_

Also enable the _“Treat warnings as errors”_ build setting.

[warnings-slides]: https://speakerdeck.com/hasseg/the-compiler-is-your-friend

### Clang Static Analyzer

The Clang compiler (which Xcode uses) has a _static analyzer_ that performs control and data flow analysis on your code and checks for lots of errors that the compiler cannot.

You can manually run the analyzer from the _Product → Analyze_ menu item in Xcode.

The analyzer can work in either “shallow” or “deep” mode. The latter is much slower but may find more issues due to cross-function control and data flow analysis.

Recommendations:

- Enable _all_ of the checks in the analyzer (by enabling all of the options in the “Static Analyzer” build setting sections)
- Enable the _“Analyze during ‘Build’”_ build setting for your release build configuration to have the analyzer run automatically during release builds. (Seriously, do this — you’re not going to remember to run it manually.)
- Set the _“Mode of Analysis for ‘Analyze’”_ build setting to _Shallow (faster)_
- Set the _“Mode of Analysis for ‘Build’”_ build setting to _Deep_

### [Faux Pas](http://fauxpasapp.com/)

Created by our very own [Ali Rantakari][ali-rantakari-twitter], Faux Pas is a fabulous static error detection tool. It analyzes your codebase and finds issues you had no idea even existed. Be sure to run this before shipping any iOS (or Mac) app!

_(Note: all Futurice employees get a free license to this — just ask Ali.)_

[ali-rantakari-twitter]: https://twitter.com/AliRantakari

### Debugging

When your app crashes, Xcode does not break into the debugger by default. To achieve this, add an exception breakpoint (click the "+" at the bottom of Xcode's Debug Navigator) to halt execution whenever an exception is raised. In many cases, you will then see the line of code responsible for the exception. This catches any exception, even handled ones. If Xcode keeps breaking on benign exceptions in third party libraries e.g., you might be able to mitigate this by choosing _Edit Breakpoint_ and setting the _Exception_ drop-down to _Objective-C_.

For view debugging, [Reveal][reveal] and [Spark Inspector][spark-inspector] are two powerful visual inspectors that can save you hours of time, especially if you're using Auto Layout and want to locate views that are collapsed or off-screen. Granted, Xcode offers [something very similar][xcode-view-debugging] for free, but it's iOS 8+ only and feels somewhat less polished.

[reveal]: http://revealapp.com/
[spark-inspector]: http://sparkinspector.com
[xcode-view-debugging]: https://developer.apple.com/library/ios/recipes/xcode_help-debugger/using_view_debugger/using_view_debugger.html

### Profiling

Xcode comes with a profiling suite called Instruments. It contains a myriad of tools for profiling memory usage, CPU, network communications, graphics and much more. It's a complex beast, but one of its more straight-forward use cases is tracking down memory leaks with the Allocations instrument. Simply choose _Product_ > _Profile_ in Xcode, select the Allocations instrument, hit the Record button and filter the Allocation Summary on some useful string, like the prefix of your own app's class names. The count in the Persistent column then tells you how many instances of each object you have. Any class for which the instance count increases indiscriminately indicates a memory leak.

Also good to know is that Instruments has an Automation tool for recording and playing back UI interactions as JavaScript files. [UI Auto Monkey][ui-auto-monkey] is a script that will use Automation to randomly pummel your app with taps, swipes and rotations which can be useful for stress/soak testing.

Pay extra attention to how and where you create expensive classes. `NSDateFormatter`, for instance, is very expensive to create and doing so in rapid succession, e.g. inside a `tableView:cellForRowAtIndexPath:` method, can really slow down your app. Instead, keep a static instance of it around for each date format that you need.

[ui-auto-monkey]: https://github.com/jonathanpenn/ui-auto-monkey

## Analytics

Including some analytics framework in your app is strongly recommended, as it allows you to gain insights on how people actually use it. Does feature X add value? Is button Y too hard to find? To answer these, you can send events, timings and other measurable information to a service that aggregates and visualizes them – for instance, [Google Tag Manager][google-tag-manager]. The latter is more versatile than Google Analytics in that it inserts a data layer between app and Analytics, so that the data logic can be modified through a web service without having to update the app.

[google-tag-manager]: http://www.google.com/tagmanager/

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

### Crash Logs

First you should make your app send crash logs onto a server somewhere so that you can access them. You can implement this manually (using [PLCrashReporter][plcrashreporter] and your own backend) but it’s recommended that you use an existing service instead — for example one of the following:

* [Crashlytics](http://www.crashlytics.com)
* [HockeyApp](http://hockeyapp.net)
* [Crittercism](https://www.crittercism.com)
* [Splunk MINTexpress](https://mint.splunk.com)

[plcrashreporter]: https://www.plcrashreporter.org

Once you have this set up, ensure that you _save the Xcode archive (`.xcarchive`)_ of every build you release. The archive contains the built app binary and the debug symbols (`dSYM`) which you will need to symbolicate crash reports from that particular version of your app.


## Building

### Build Configurations

Even simple apps can be built in different ways. The most basic separation that Xcode gives you is that between _debug_ and _release_ builds. For the latter, there is a lot more optimization going on at compile time, at the expense of debugging possibilities. Apple suggests that you use the _debug_ build configuration for development, and create your App Store packages using the _release_ build configuration. This is codified in the default scheme (the dropdown next to the Play and Stop buttons in Xcode), which commands that _debug_ be used for Run and _release_ for Archive.

However, this is a bit too simple for real-world applications. You might – no, [_should!_][futurice-environments] – have different environments for testing, staging and other activities related to your service. Each might have its own base URL, log level, bundle identifier (so you can install them side-by-side), provisioning profile and so on. Therefore a simple debug/release distinction won't cut it. You can add more build configurations on the "Info" tab of your project settings in Xcode.

[futurice-environments]: https://blog.futurice.com/five-environments-you-cannot-develop-without

#### `xcconfig` files for build settings

Typically build settings are specified in the Xcode GUI, but you can also use _configuration settings files_ (“`.xcconfig` files”) for them. The benefits of using these are:

- You can add comments to explain things
- You can `#include` other build settings files, which helps you avoid repeating yourself:
    - If you have some settings that apply to all build configurations, add a `Common.xcconfig` and `#include` it in all the other files
    - If you e.g. want to have a “Debug” build configuration that enables compiler optimizations, you can just `#include "MyApp_Debug.xcconfig"` and override one of the settings
- Conflict resolution and merging becomes easier

Find more information about this topic in [these presentation slides][xcconfig-slides].

[xcconfig-slides]: https://speakerdeck.com/hasseg/xcode-configuration-files

### Targets

A target resides conceptually below the project level, i.e. a project can have several targets that may override its project settings. Roughly, each target corresponds to "an app" within the context of your codebase. For instance, you could have country-specific apps (built from the same codebase) for different countries' App Stores. Each of these will need development/staging/release builds, so it's better to handle those through build configurations, not targets. It's not uncommon at all for an app to only have a single target.

### Schemes

Schemes tell Xcode what should happen when you hit the Run, Test, Profile, Analyze or Archive action. Basically, they map each of these actions to a target and a build configuration. You can also pass launch arguments, such as the language the app should run in (handy for testing your localizations!) or set some diagnostic flags for debugging.

A suggested naming convention for schemes is `MyApp (<Language>) [Environment]`:

    MyApp (English) [Development]
    MyApp (German) [Development]
    MyApp [Testing]
    MyApp [Staging]
    MyApp [App Store]

For most environments the language is not needed, as the app will probably be installed through other means than Xcode, e.g. TestFlight, and the launch argument thus be ignored anyway. In that case, the device language should be set manually to test localization.

## Deployment

Deploying software on iOS devices isn't exactly straightforward. That being said, here are some central concepts that, once understood, will help you tremendously with it.

### Signing

Whenever you want to run software on an actual device (as opposed to the simulator), you will need to sign your build with a __certificate__ issued by Apple. Each certificate is linked to a private/public keypair, the private half of which resides in your Mac's Keychain. There are two types of certificates:

* __Development certificate:__ Every developer on a team has their own, and it is generated upon request. Xcode might do this for you, but it's better not to press the magic "Fix issue" button and understand what is actually going on. This certificate is needed to deploy development builds to devices.
* __Distribution certificate:__ There can be several, but it's best to keep it to one per organization, and share its associated key through some internal channel. This certificate is needed to ship to the App Store, or your organization's internal "enterprise app store".

### Provisioning

Besides certificates, there are also __provisioning profiles__, which are basically the missing link between devices and certificates. Again, there are two types to distinguish between development and distribution purposes:

* __Development provisioning profile:__ It contains a list of all devices that are authorized to install and run the software. It is also linked to one or more development certificates, one for each developer that is allowed to use the profile. The profile can be tied to a specific app or use a wildcard App ID (*). The latter is [discouraged][jared-sinclair-signing-tips], because Xcode is notoriously bad at picking the correct files for signing unless guided in the right direction. Also, certain capabilities like Push Notifications or App Groups require an explicit App ID.

* __Distribution provisioning profile:__ There are three different ways of distribution, each for a different use case. Each distribution profile is linked to a distribution certificate, and will be invalid when the certificate expires.
    * __Ad-Hoc:__ Just like development profiles, it contains a whitelist of devices the app can be installed to. This type of profile can be used for beta testing on 100 devices per year. For a smoother experience and up to 1000 distinct users, you can use Apple's newly acquired [TestFlight][testflight] service. Supertop offers a good [summary of its advantages and issues][testflight-discussion].
    * __App Store:__ This profile has no list of allowed devices, as anyone can install it through Apple's official distribution channel. This profile is required for all App Store releases.
    * __Enterprise:__ Just like App Store, there is no device whitelist, and the app can be installed by anyone with access to the enterprise's internal "app store", which can be just a website with links. This profile is available only on Enterprise accounts.

[jared-sinclair-signing-tips]: http://blog.jaredsinclair.com/post/116436789850/
[testflight]: https://developer.apple.com/testflight/
[testflight-discussion]: http://blog.supertop.co/post/108759935377/app-developer-friends-try-testflight

To sync all certificates and profiles to your machine, go to Accounts in Xcode's Preferences, add your Apple ID if needed, and double-click your team name. There is a refresh button at the bottom, but sometimes you just need to restart Xcode to make everything show up.

#### Debugging Provisioning

Sometimes you need to debug a provisioning issue. For instance, Xcode may refuse to install the build to an attached device, because the latter is not on the (development or ad-hoc) profile's device list. In those cases, you can use Craig Hockenberry's excellent [Provisioning][provisioning] plugin by browsing to `~/Library/MobileDevice/Provisioning Profiles`, selecting a `.mobileprovision` file and hitting Space to launch Finder's Quick Look feature. It will show you a wealth of information such as devices, entitlements, certificates, and the App ID.

[provisioning]: https://github.com/chockenberry/Provisioning

### Uploading

[iTunes Connect][itunes-connect] is Apple's portal for managing your apps on the App Store. To upload a build, Xcode 6 requires an Apple ID that is part of the developer account used for signing. This can make things tricky when you are part of several developer accounts and want to upload their apps, as for mysterious reasons _any given Apple ID can only be associated with a single iTunes Connect account_. One workaround is to create a new Apple ID for each iTunes Connect account you need to be part of, and use Application Loader instead of Xcode to upload the builds. That effectively decouples the building and signing process from the upload of the resulting `.app` file.

After uploading the build, be patient as it can take up to an hour for it to show up under the Builds section of your app version. When it appears, you can link it to the app version and submit your app for review.

[itunes-connect]: https://itunesconnect.apple.com

## In-App Purchases (IAP)

When validating in-app purchase receipts, remember to perform the following checks:

- __Authenticity:__ That the receipt comes from Apple
- __Integrity:__ That the receipt has not been tampered with
- __App match:__ That the app bundle ID in the receipt matches your app’s bundle identifier
- __Product match:__ That the product ID in the receipt matches your expected product identifier
- __Freshness:__ That you haven’t seen the same receipt ID before.

Whenever possible, design your IAP system to store the content for sale server-side, and provide it to the client only in exchange for a valid receipt that passes all of the above checks. This kind of a design thwarts common piracy mechanisms, and — since the validation is performed on the server — allows you to use Apple’s HTTP receipt validation service instead of interpreting the receipt `PKCS #7` / `ASN.1` format yourself.

For more information on this topic, check out the [Futurice blog: Validating in-app purchases in your iOS app][futu-blog-iap].

[futu-blog-iap]: http://futurice.com/blog/validating-in-app-purchases-in-your-ios-app

## License

[Futurice][futurice] • Creative Commons Attribution 4.0 International (CC BY 4.0)

[futurice]: http://www.futurice.com


## More Ideas

- 3x assets, iPhone 6 screen sizes explained
- Add list of suggested compiler warnings
- Ask IT about automated Jenkins build machine
- Add section on Testing
- Add "proven don'ts"

[reactivecocoa-github]: https://github.com/ReactiveCocoa/ReactiveCocoa
