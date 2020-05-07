InAppSettingsKit
================

[![Build Status](https://travis-ci.org/futuretap/InAppSettingsKit.svg?branch=master)](https://travis-ci.org/futuretap/InAppSettingsKit)

InAppSettingsKit (IASK) is an open source framework to easily add in-app settings to your iPhone apps. Normally iOS apps use the `Settings.bundle` resource to add app-specific settings in the Settings app. InAppSettingsKit takes advantage of the same bundle and allows you to present the same settings screen within your app. So the user has the choice where to change the settings.

IASK not only replicates the feature set of system settings but supports a large number of additional elements and configuration options.

<a href="https://flattr.com/thing/799297/futuretapInAppSettingsKit-on-GitHub" target="_blank">
<img src="http://api.flattr.com/button/flattr-badge-large.png" alt="Flattr this" title="Flattr this" border="0" /></a>


How does it work?
=================

To support traditional Settings.app panes, the app must include a `Settings.bundle` with at least a `Root.plist` to specify the connection of settings UI elements with `NSUserDefaults` keys. InAppSettingsKit basically just uses the same Settings.bundle to do its work. This means there's no additional work when you want to include a new settings parameter. It just has to be added to the Settings.bundle and it will appear both in-app and in Settings.app. All settings types like text fields, sliders, toggle elements, child views etc. are supported.


How to include it?
==================

The source code is available on [github](http://github.com/futuretap/InAppSettingsKit). There are several ways of installing it:

**Using Carthage**

Add to your `Cartfile`:

    github "futuretap/InAppSettingsKit" "master"


**Using CocoaPods**

Add to your `Podfile`:

    pod 'InAppSettingsKit'

**Using a static library**

Use the static library project to include InAppSettingsKit. To see an example on how to do it, open `InAppSettingsKit.xcworkspace`. It includes the sample application that uses the static library as well as the static library project itself. To include the static library project there are only a few steps necessary:

* add the `InAppSettingsKit.xcodeproject` into your application's workspace
* add `libInAppSettingsKit.a` to your application's libraries by opening the Build-Phases pane of the main application and adding it in `Link Binary with Libraries`
* use IASK by importing it via #import <InAppSettingsKit/...>
* for Archive builds there's a minor annoyance: To make those work, you'll need to add `$(OBJROOT)/UninstalledProducts/include` to the `HEADER_SEARCH_PATHS`

Then you can display the InAppSettingsKit view controller using a navigation push, as a modal view controller or in a separate tab of a TabBar based application. The sample app demonstrates all three ways to integrate InAppSettingsKit. 

You may need to make two changes to your project to get it to compile:

1. Add `MessageUI.framework` and
2. Enable ARC for the IASK files.

Both changes can be made by finding your target and navigating to the Build Phases tab. 

`MessageUI.framework` is needed for `MFMailComposeViewController` and can be added in the "Link Binary With Libraries" Section. Use the + icon.

To enable ARC select all IASK* source files in the "Compile Sources" section, press Enter, insert `-fobjc-arc` and then "Done".


App Integration
===============

In order to start using IASK you must:

- Add `Settings.bundle` to your project (`File` -> `Add File` -> `Settings bundle`) and edit `Root.plist` with your settings (see Apple's documentation on the [Schema File Root Content](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/UserDefaults/Preferences/Preferences.html)). Read on to get insight into more advanced uses.

Further integration depends on how your app is structured.

**Apps with UI built in code**

- Create a class inheriting from the `IASKAppSettingsViewController`:

```objective-c
#import "InAppSettingsKit/IASKAppSettingsViewController.h"

@interface SettingsTableViewController : IASKAppSettingsViewController

@end
```

and continue with instantiating `SettingsTableViewController`.  This way,
you can customize the appearance of InAppSettingsKit by overriding some
`UITableViewDataSource` or `UITableViewDelegate` methods.


There's a sample application called `InAppSettingsAppStaticLibrary` which is
an example of integration of IASK (static library build) with the app.

**Apps with UI built with storyboards**

- Create the `IASKAppSettingsViewController` subclass named `SettingsTableViewController` like above
- Drag and drop a Table View Controller embedded into a Navigation Controller into your app and wire the storyboard
  to your app UI
- Set the Table View Controller class to `SettingsTableViewController`
- In the Table View Controller set "Show Done Button" under "App Settings View Controller" to "On"
- Set the Table View to "Grouped" style

There's a sample application `InAppSettingsSampleAppStoryboard` which shows how to wire everything up.

**Additional changes**

Depending on your project it might be needed to make some changes in the startup code of your app. Your app has to be able to reconfigure itself at runtime if the settings are changed by the user. This could be done in a `-reconfigure` method that is being called from `-applicationDidFinishLaunching` as well as in the delegate method `-settingsViewControllerDidEnd:` of `IASKAppSettingsViewController`.


iCloud sync
===========
To sync your `NSUserDefaults` with iCloud, there's another project called [FTiCloudSync](https://github.com/futuretap/FTiCloudSync) which is implemented as a category on `NSUserDefaults`: All write and remove requests are automatically forwarded to iCloud and all updates from iCloud are automatically stored in `NSUserDefaults`. InAppSettingsKit automatically updates the UI if the standard `NSUserDefaults` based store is used.



Goodies
=======
The intention of InAppSettingsKit was to create a 100% imitation of the Settings.app behavior. However, we added some bonus features for extra flexibility.


Custom inApp plists
---------------------------
Since iOS 4 Settings plists can be device-dependent: `Root~ipad.plist` will be used on iPad and `Root~iphone.plist` on iPhone. If not existent, `Root.plist` will be used. InAppSettingsKit adds the possibility to override those standard files by using `.inApp.plist` instead of `.plist`. Alternatively, you can create a totally separate bundle named `InAppSettings.bundle` instead of the usual `Settings.bundle`. The latter approach is useful if you want to suppress the settings in Settings.app.

In summary, the plists are searched in this order:

- InAppSettings.bundle/FILE~DEVICE.inApp.plist
- InAppSettings.bundle/FILE.inApp.plist
- InAppSettings.bundle/FILE~DEVICE.plist
- InAppSettings.bundle/FILE.plist
- Settings.bundle/FILE~DEVICE.inApp.plist
- Settings.bundle/FILE.inApp.plist
- Settings.bundle/FILE~DEVICE.plist
- Settings.bundle/FILE.plist

Different in-app settings are useful in a variety of situations. For example, [Where To?](http://www.futuretap.com/whereto) uses this mechanism to change the wording of "At next start" (for resetting confirmation dialogs) to be appropriate if the app is already running.


Privacy link
------------
If the app includes a usage key for various privacy features such as camera or location access in its `Info.plist`, IASK displays a "Privacy" cell at the top of the root settings page. This cell opens the system Settings app and displays the settings pane for the app where the user can specify the privacy settings for the app.

If you don't want to show Privacy cells, set the property `neverShowPrivacySettings` to `YES`.

The sample app defines `NSMicrophoneUsageDescription` to let the cell appear. Note that the settings page doesn't show any privacy settings yet because the app doesn't actually access the microphone. Privacy settings only show up in the Settings app after first use of the privacy-protected API.


Open URL
--------
InAppSettingsKit adds a new element `IASKOpenURLSpecifier` that allows to open a specified URL using an external application (i.e. Safari or Mail). See the sample `Root.inApp.plist` for details.


Mail Composer
-------------
The custom `IASKMailComposeSpecifier` element allows to send mail from within the app by opening a mail compose view. You can set the following (optional) parameters using the settings plist: `IASKMailComposeToRecipents`, `IASKMailComposeCcRecipents`, `IASKMailComposeBccRecipents`, `IASKMailComposeSubject`, `IASKMailComposeBody`, `IASKMailComposeBodyIsHTML`. Optionally, you can implement

    - (BOOL)settingsViewController:(id<IASKViewController>)settingsViewController shouldPresentMailComposeViewController:(MFMailComposeViewController*)mailComposeViewController forSpecifier:(IASKSpecifier*)specifier;

in your delegate to customize the mail (e.g. pre-fill the body with dynamic content, add attachments) modify the appearance of the compose view controller or even block the standard presentation. An alert is displayed if Email is not configured on the device. `IASKSpecifier` is the internal model object defining a single settings cell. Important IASKSpecifier properties:

- `key`: corresponds to the `Key` in the Settings plist
- `title`: the localized title of settings key
- `type`: corresponds to the `Type` in the Settings plist
- `defaultValue`: corresponds to the `DefaultValue` in the Settings plist


Buttons
-------
InAppSettingsKit adds a `IASKButtonSpecifier` element that allows to call a custom action. Just add the following delegate method:

    - (void)settingsViewController:(IASKAppSettingsViewController*)sender buttonTappedForSpecifier:(IASKSpecifier*)specifier;

The sender is always an instance of `IASKAppSettingsViewController`, a `UIViewController` subclass. So you can access its view property (might be handy to display an action sheet) or push another view controller. Another nifty feature is that the title of IASK buttons can be overriden by the (localizable) value from `NSUserDefaults` (or any other settings store - see below). This comes in handy for toggle buttons (e.g. Login/Logout). See the sample app for details.

By default, Buttons are aligned centered except if an image is specified (default: left-aligned). The default alignment may be overridden.


Multiline Text Views
--------------------
Similar to standard text fields, `IASKTextViewSpecifier` displays a full-width, multi line text view that resizes according to the entered text. It also supports `KeyboardType`, `AutocapitalizationType` and `AutocorrectionType`.


Date Picker
-----------
`IASKDatePickerSpecifier` displays a `UIDatePicker` to set a date and/or time. It supports the following options:

 - `DatePickerMode`: one of `Date`, `Time`, or `DateAndTime` (see [UIDatePickerMode](https://developer.apple.com/documentation/uikit/uidatepickermode)). Default is `DateAndTime`.
 - `MinuteInterval`: The interval at which the date picker displays minutes. Default: 1.

There are 3 optional delegate methods to customize how to store and display dates and times:

    - (NSDate*)settingsViewController:(IASKAppSettingsViewController*)sender dateForSpecifier:(IASKSpecifier*)specifier;

Implement this if you store the date/time in a custom format other than as `NSDate` object. Called when the user starts editing a date/time by selecting the title cell above the date/time picker.

    - (NSString*)settingsViewController:(IASKAppSettingsViewController*)sender datePickerTitleForSpecifier:(IASKSpecifier*)specifier;
    
Implement this to customize the displayed value in the title cell above the date/time picker.

    - (void)settingsViewController:(IASKAppSettingsViewController*)sender setDate:(NSDate*)date forSpecifier:(IASKSpecifier*)specifier;

Implement this if you store the date/time in a custom format other than an `NSDate` object. Called when the user changes the date/time value using the picker.


Custom Views
------------
You can specify your own `UITableViewCell` within InAppSettingsKit by using the type `IASKCustomViewSpecifier`. A mandatory field in this case is the `Key` attribute. Also, you have to support the `IASKSettingsDelegate` protocol and implement these methods:

    - (CGFloat)tableView:(UITableView*)tableView heightForSpecifier:(IASKSpecifier*)specifier;
    - (UITableViewCell*)tableView:(UITableView*)tableView cellForSpecifier:(IASKSpecifier*)specifier;

Both methods are called for all your `IASKCustomViewSpecifier` entries. To differentiate them, you can access the `Key` attribute using `specifier.key`. In the first method you return the height of the cell, in the second method the cell itself. You should use reusable `UITableViewCell` objects as usual in table view programming. There's an example in the Demo app.
Optionally you can implement

    - (void)settingsViewController:(IASKAppSettingsViewController*)sender tableView:(UITableView *)tableView didSelectCustomViewSpecifier:(IASKSpecifier*)specifier;

to catch tap events for your custom view.

If you specify `File`,  `IASKViewControllerClass`, `IASKViewControllerStoryBoardId`, or `IASKSegueIdentifier` (see below), the selection behavior of a custom view is identical to a child pane and the delegate is not called on selection.



Group Headers and Footers
-------------------------
The FooterText key for Group elements is available in system settings. It is supported in InAppSettingsKit as well. On top of that, we support this key for Multi Value elements as well. The footer text is displayed below the table of multi value options.

You can define a custom header view for `PSGroupSpecifier` segments by adding a `Key` attribute and implementing the following method in your `IASKSettingsDelegate`:
    
	- (UIView *)settingsViewController:(id<IASKViewController>)settingsViewController tableView:(UITableView *)tableView viewForHeaderForSection:(NSInteger)section;

You can adjust the height of the header by implementing the following method:

	- (CGFloat)settingsViewController:(id<IASKViewController>)settingsViewController tableView:(UITableView*)tableView heightForHeaderForSection:(NSInteger)section;

For simpler header title customization without the need for a custom view, and provided the `-settingsViewController:tableView:viewForHeaderForSection:` method has not been implemented or returns `nil` for the section, implement the following method:

	- (NSString *)settingsViewController:(id<IASKViewController>)settingsViewController tableView:(UITableView*)tableView titleForHeaderForSection:(NSInteger)section;

If the method returns `nil` or a 0-length string, the title defined in the `.plist` will be used.

This behaviour is similar to custom table view cells. When implementing a method and if you need it, the section key can be retrieved from its index conveniently with:

	NSString *key = [settingsViewController.settingsReader keyForSection:section];

Check the demo app for a concrete example.

For footer customization, three methods from the `IASKSettingsDelegate` protocol can be similarly implemented.


Custom ViewControllers
----------------------
For child pane elements (`PSChildPaneSpecifier`), Apple requires a `file` key that specifies the child plist. InAppSettingsKit allow to alternatively specify `IASKViewControllerClass` and `IASKViewControllerSelector`. In this case, the child pane is displayed by instantiating a UIViewController subclass of the specified class and initializing it using the init method specified in the `IASKViewControllerSelector`. The selector must have two arguments: an `NSString` argument for the file name in the Settings bundle and the `IASKSpecifier`. The custom view controller is then pushed onto the navigation stack. See the sample app for more details.
##### Using Custom ViewControllers from StoryBoard
Alternatively specify `IASKViewControllerStoryBoardId` to initiate a viewcontroller from [main storyboard](https://developer.apple.com/library/ios/documentation/general/conceptual/Devpedia-CocoaApp/Storyboard.html/).
Specifiy `IASKViewControllerStoryBoardFile` to use a story board other than MainStoryboard file.


Perform Segues
--------------
As an alternative to `IASKViewControllerClass` and `IASKViewControllerSelector` for child pane elements (`PSChildPaneSpecifier`), InAppSettingsKit is able to navigate to another view controller, by performing any segue defined in your storyboard. To do so specify the segue identifier in `IASKSegueIdentifier`.


Subtitles
---------
The `IASKSubtitle` key allows to define subtitles for these elements: Toggle, ChildPane, OpenURL, MailCompose, Button. Using a subtitle implies left alignment.
A child pane displays its value as a subtitle, if available and no `IASKSubtitle` is specified.


Placeholder
-----------
The `IASKPlaceholder` key allows to define placeholder for TextField and TextView (`IASKTextViewSpecifier`).


Regular expression validation
-----------------------------
The `IASKRegex` key can be used to specify a regular expression for validating text entered into TextField. For example checking the text entered is a valid number (only one decimal point, if a minus sign is present it must be the first charecter, etc). When this key is used the settings store is only update when editting of the field finishes. This enables easier editing as the field doesn't have to be valid after every charecter change.

By default, the text field shakes on a validation failure when leaving the field. The following optional delegate methods can be used to customise the behaviour on a validation failure and a subsequent validation sucess. For example, the text field could be set to red to indicate the value isn't valid. The delegate can prevent the default validation failure behaviour by returning `NO` from the `validationFailureForSpecifier` method.
- `(BOOL)settingsViewController:(IASKAppSettingsViewController*)sender validationFailureForSpecifier:(IASKSpecifier*)specifier textField:(IASKTextField *)field previousValue:(NSString*)prevValue;`
- `(void)settingsViewController:(IASKAppSettingsViewController*)sender validationSuccessForSpecifier:(IASKSpecifier*)specifier textField:(IASKTextField *)field;`


Text alignment
--------------
For some element types, a `IASKTextAlignment` attribute may be added with the following values to override the default alignment:

- `IASKUITextAlignmentLeft` (ChildPane, TextField, Buttons, OpenURL, MailCompose)
- `IASKUITextAlignmentCenter` (ChildPane, Buttons, OpenURL)
- `IASKUITextAlignmentRight` (ChildPane, TextField, Buttons, OpenURL, MailCompose)


Toggle style
------------
`PSToggleSwitchSpecifier` switches use a `UISwitch` by default. By specifying the option `IASKToggleStyle` = `Checkmark`, checkmarks are displayed for selected keys and nothing for unselected keys.


Variable font size
------------------
By default, the labels in the settings table are displayed in a variable font size, especially handy to squeeze-in long localizations (beware: this might break the look in Settings.app if labels are too long!).
To disable this behavior, add a `IASKAdjustsFontSizeToFitWidth` Boolean attribute with value `NO`.


Icons
-----
All element types (except sliders which already have a `MinimumValueImage`) support an icon image on the left side of the cell. You can specify the image name in an optional `IASKCellImage` attribute. The ".png" or "@2x.png" suffix is automatically appended and will be searched in the project. Optionally, you can add an image with suffix "Highlighted.png" or "Highlighted@2x.png" to the project and it will be automatically used as a highlight image when the cell is selected (for Buttons and ChildPanes).


MultiValue Lists
----------------
MultiValue lists (`PSMultiValueSpecifier`) can fetch their values and titles dynamically from the delegate instead of the static Plist. Implement these two methods in your `IASKSettingsDelegate`:

    - (NSArray*)settingsViewController:(IASKAppSettingsViewController*)sender valuesForSpecifier:(IASKSpecifier*)specifier;
    - (NSArray*)settingsViewController:(IASKAppSettingsViewController*)sender titlesForSpecifier:(IASKSpecifier*)specifier;

The sample app returns a list of all country codes as values and the localized country names as titles.

MultiValue lists can be sorted alphabetically by adding a `true` Boolean `DisplaySortedByTitle` key in the Plist.
MultiValue list entries can be given an image. Specify images via the `IconNames` attribute (next to Values/Titles/ShortTitles etc.).


Settings Storage
----------------
The default behaviour of IASK is to store the settings in `[NSUserDefaults standardUserDefaults]`. However, it is possible to change this behavior by setting the `settingsStore` property on an `IASKAppSettingsViewController`. IASK comes with two store implementations: `IASKSettingsStoreUserDefaults` (the default one) and `IASKSettingsStoreFile`, which read and write the settings in a file of the path you choose. If you need something more specific, you can also choose to create your own store. The easiest way to create your own store is to create a subclass of `IASKAbstractSettingsStore`. Only 3 methods are required to override. See `IASKSettingsStore.{h,m}` for more details.


Notifications
-------------
There's a `kIASKAppSettingChanged` notification that is sent for every changed settings key. The `object` of the notification is the sending view controller  and the `userInfo` dictionary contains the key and new value of the affected key.


Dynamic cell hiding
-------------------
Sometimes, options depend on each other. For instance, you might want to have an "Auto Connect" switch, and let the user set username and password if enabled. To react on changes of a specific setting, use the `kIASKAppSettingChanged` notification explained above.

To hide a set of cells use:

    - (void)[IASKAppSettingsViewController setHiddenKeys:(NSSet*)hiddenKeys animated:(BOOL)animated];

or the non-animated version:

	@property (nonatomic, retain) NSSet *hiddenKeys;

See the sample app for more details. Note that InAppSettingsKit uses Settings schema, not TableView semantics: If you want to hide a group of cells, you have to include the Group entry as well as the member entries.

More information
----------------
In the [Dr. Touch podcast](http://www.drobnik.com/touch/2010/01/dr-touch-010-a-new-decade/) and the [MDN Show Episode 027](http://itunes.apple.com/us/podcast/the-mdn-show/id318584787) [Ortwin Gentz](http://twitter.com/ortwingentz) talks about InAppSettingsKit.


Support
=======
Please don't use Github issues for support requests, we'll close them. Instead, post your question on [StackOverflow](http://stackoverflow.com) with tag `inappsettingskit`.


The License
===========

We released the code under the liberal BSD license in order to make it possible to include it in every project, be it a free or paid app. The only thing we ask for is giving the original developers some credit. The easiest way to include credits is by leaving the "Powered by InAppSettingsKit" notice in the code. If you decide to remove this notice, a noticeable mention on the App Store description page or homepage is fine, too. 
