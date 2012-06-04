Best Practices for iOS Software Design
======================================

This article's goal is to help you write more stable code for your iOS applications. I highly encourage you to
contribute your own best practices via Github's
[pull requests](https://github.com/jverkoey/iOS-Best-Practices/pull/new/master).

<a rel="license" href="http://creativecommons.org/licenses/by/3.0/"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by/3.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a>.

Originally written by: Jeff Verkoeyen (@featherless)

Table of Contents
=================

- [Be Mindful of the Lifetime of Views](#be-mindful-of-the-lifetime-of-views)
  * [Do not access self.view in init- methods](#rule-do-not-access-selfview-in-init--methods)
  * [Use data source protocols to strongly separate data from views](##rule-use-data-source-protocols-to-strongly-separate-data-from-views)

Be Mindful of the Lifetime of Views
-----------------------------------

> Remind yourself constantly that, at any time, your views may be destroyed.

### Rule: Do not access self.view in init- methods

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

### Rule: Use data source protocols to strongly separate data from views

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