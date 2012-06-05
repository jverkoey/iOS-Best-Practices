Best Practices for iOS Software Design
======================================

This article's goal is to help you write stable code for your iOS applications. I highly encourage you to
contribute your own best practices via Github's
[pull requests](https://github.com/jverkoey/iOS-Best-Practices/pull/new/master).

<a rel="license" href="http://creativecommons.org/licenses/by/3.0/"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by/3.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a>.

Originally written by: Jeff Verkoeyen (@featherless)

Table of Contents
=================

- [Be Mindful of the Lifetime of Views](#be-mindful-of-the-lifetime-of-views)
  * [Do not access self.view in init- methods](#do-not-access-selfview-in-init--methods)
  * [Use data source protocols to strongly separate data from views](#use-data-source-protocols-to-strongly-separate-data-from-views)
- [UIViewController](#uiviewcontroller)
  * [Use the existing navigation item object](#use-the-existing-navigation-item-object)
- [Debugging](#debugging)
  * [Use lldb for debugging](#use-lldb-for-debugging)
  * [Use NSZombieEnabled to find object leaks](#use-nszombieenabled-to-find-object-leaks)

Be Mindful of the Lifetime of Views
-----------------------------------

> Remind yourself constantly that, at any time, your views may be destroyed.

### Do not access self.view in init- methods

You should **never** access `self.view` in your controller's initialization methods. Doing so almost always leads to
hard to debug bugs because that initialization logic will not be executed again after a memory warning.

Consider a simple example:

```obj-c
- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
  if ((self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil])) {
    self.view.backgroundColor = [UIColor underPageBackgroundColor];
  }
  return self;
}
```

Imagine this controller is the root of a navigation stack and a memory warning occurs. When we pop back to this
controller, the view will no longer have `underPageBackgroundColor` as its background color. This leads to
debugging frustration, even for experienced iOS engineers.

### Use data source protocols to strongly separate data from views

When designing views that interact with data sets, always fetch the data via a data source protocol rather than
exposing setters. Views are not data containers and should not be designed to enable any such abuse. Rather,
views should be treated as expendable components that may leave existence at any point in time.

As a general rule of thumb, anything beyond static presentation information in a view should be requested via a
data source or delegate.

UILabel is a good example of a view that does not need a data source. All of its properties are set once and are
generally not expected to change throughout the lifetime of the view.

UITableView on the other hand requires a data source to fetch its data. Let's imagine what using UITableView would
be like if it didn't have a data source and instead only provided setters.

This design will lead to inevitable abuse when developers attempt to use the table view object as a place to store
their data. When the table view is inevitably released due to a memory warning the data will also be lost! This
means that we need to store the data elsewhere anyway in order to guarantee its lifetime across multiple instances
of the table view.

UIViewController
----------------

> View controllers are the binding between your models and your views.

### Use the existing navigation item object

Every instance of a UIViewController has a `navigationItem` property which should be used to specify the left and
right navigation buttons and the title view. You should *not* create a `UINavigationItem` object because the
base UIViewController implementation will automatically create one when you access `self.navigationItem`. Simply
access `self.navigationItem` and set its properties accordingly.

```obj-c
// UIViewController will automatically create the navigationItem object.
self.navigationItem.rightBarButtonItem = doneButton;
```

Debugging
---------

### Use lldb for debugging

lldb allows you to inspect properties on classes that don't have explicit ivars declared in the object's interface.

To use lldb, select "Edit Scheme..." from the "Product" menu (or press Cmd+Shift+<). Select the "Run" tab
on the left-hand side of the scheme editor. Change the debugger drop down to "LLDB".

### Use NSZombieEnabled to find object leaks

When [NSZombieEnabled](http://cocoadev.com/wiki/NSZombieEnabled) is enabled, objects that are released from memory
will be kept around as "zombies". If you attempt to access the released object again in the future, rather than
crashing with very little context, you will be shown the name of the object that was being accessed. This can be
incredibly helpful in determining where memory leaks are occurring.

To turn on NSZombieEnabled, select "Edit Scheme..." from the "Product" menu (or press Cmd+Shift+<). Select the "Run" tab
on the left-hand side of the scheme editor. Select the "Arguments" tab in that page. Add a new Environment Variable
and call it `NSZombieEnabled`. Set its value to `YES`.
