CWAC-MasterDetail: Same UI Pattern, Fewer Moving Parts
======================================================

The master-detail pattern as an approach for presenting a collection of
content to the user has been discussed since the introduction of tablets
to the Android ecosystem in 2011. In this pattern, the list of content
(master) and a piece of content (detail) are presented side-by-side on
a tablet, particularly in landscape orientation. In other environments, notably
phones, the master and detail are presented one at a time. The objective
is to allow the same UI definition to leverage a tablet's greater screen
space without much extra development work.

However, once you start tugging on the thread of the master-detail pattern,
you quickly realize that there are many *other* design patterns that Google
recommends that would affect your implementation, such as using contextual
action bars (a.k.a., action modes) for operations on content. Getting *all*
of this design guidance to work, and dealing with classic Android challenges
like configuration changes, results in a lot of infrastructure code, before
you even begin to start writing actual business logic for the app.

This library is designed to supply that infrastructure code, leaving you
free to focus more on what makes your app unique or otherwise important.

This Android library project is also
[available as a JAR](https://github.com/commonsguy/cwac-masterdetail/releases).
[JavaDocs are also available](http://javadocs.commonsware.com/cwac/masterdetail/index.html)
to accompany this README.

Basic Usage: Arbitrary Model
----------------------------
Use these instructions if your data model is something more complex than a `List`
of something.

Step #1: Download the JAR and put it in the `libs/` directory of your
project (or, if you prefer, clone this GitHub repo and add
it as a library project to your main project).

Step #2: Extend `MasterDetailActivity` and `MasterDetailHelper`.

Step #3: Override `buildMasterDetailHelper()` in your `MasterDetailActivity`
to return an instance of your `MasterDetailHelper`.

Step #4: Override `buildPagerAdapter()` in your `MasterDetailHelper` to return
a `PagerAdapter`, just like you would create if you were using a `ViewPager`.
Make sure your `PagerAdapter` has a useful `getPageTitle()` implementation,
just like you would if you were using `PagerTabStrip` or similar indicators
with your `ViewPager`.

Step #5: Override `buildModelCollection()` in your `MasterDetailHelper`
to return your model collection, so that it can be retained across configuration
changes. You can retrieve this later on by calling `getModelCollection()`
on your `MasterDetailHelper`. Returning `null` should be safe. Note that this step
will not be required in future editions of this library, as `buildModelCollection()`
should not be `abstract`, but should just return `null` by default.

And that's it.

What you get includes:

- Automatic display of the one-at-a-time (a.k.a., single-pane) and
side-by-side (a.k.a., dual-pane) master-detail presentations,
based upon the device's width.

- In single-pane mode, when the user switches to the detail, the detail is shown
in a `ViewPager`, to allow for horizontal swiping to browse the content, without
having to bounce back and forth between the master and the detail.

- In dual-pane mode, the pane sizes are resizeable by the user, by long-pressing on
the divider between them, then dragging (using a slightly-modified version
of [Justin Shapcott's SplitPaneLayout](https://github.com/MobiDevelop/android-split-pane-layout))

- The list contents for the master are automatically generated from the titles
of the pages in the `ViewPager`

Other features can be enabled by opting into them, using various configuration
options described later in this document.

Basic Usage: List Model
-----------------------
Use these instructions if your data model is a `List` of model objects, and you want
to gain additional built-in assistance for that from the library.

Step #1: Download the JAR and put it in the `libs/` directory of your
project (or, if you prefer, clone this GitHub repo and add
it as a library project to your main project).

Step #2: Extend `MasterDetailActivity` and `MasterDetailController` (a subclass
of `MasterDetailHelper`).

Step #3: Override `buildMasterDetailHelper()` in your `MasterDetailActivity`
to return an instance of your `MasterDetailController`.

Step #4: Override a series of methods in your `MasterDetailController` subclass:

- `getModelTag()`, given an instance of a model object from your list, should
return some uniquely identifying tag for that object

- `buildFragmentForTag()`, given one of the aforementioned tags, should return
a `Fragment`, designed for display in the detail area, for the model associated
with that tag

- `createNewModel()` should return a new, empty-but-initialized, instance of your
model class (note: you do not need to add it to your list; that will be done for you)

- `removeModel()`, given an instance of a model object from your list, should remove
it (e.g., delete associated database row) (note: you do not need to remove it from your
list; that will be done for you)

- `getOptionsMenuResource()` and `getActionModeResource()`, to return menu resource
identifiers (`R.menu.whatever`), to be used for the action bar and action mode,
respectively

- `getAddMenuId()` should return the menu item ID (`R.id.whatever`) from the
options menu resource, representing the add menu item

- `getRemoveMenuId()` should return the menu item ID (`R.id.whatever`) from the
action mode resource, representing the remove menu item

And that's it.

What you get includes all of what you get from the previous instructions for
using `MasterDetailHelper`, plus:

- Automatic handling of the add action bar item, to create a new instance of your
model and add it to the collection (and the master list), plus display it to the user

- Automatic action mode for operating in multiple-choice mode, triggered by the user
long-pressing on a row in the master list

- Automatic handling of the delete action mode option, removing the selected models
from the collection (and the master list)

Again, other features can be enabled by opting into them, using various configuration
options described later in this document.

Simple Configuration and Usage
------------------------------
This section outlines some fairly easy ways that you can augment what you get
"out of the box" with this library.

### Action Bar

Your `MasterDetailHelper` can override `onCreateOptionsMenu()` and `onOptionsItemSelected()`
methods that behave equivalent to their activity counterparts. Be sure to chain
to the superclass for each of these, to leverage built-in support offered by the
library (particularly with `MasterDetailController`).

### Automatic Action Mode

If your `MasterDetailHelper` wants an action mode to appear when the user long-presses
on a list row, override `offerActionMode()` to return `true`, and override
`getActionModeResource()` to return the menu resource ID (`R.menu.whatever`) to be
used for the action mode itself. The library will handle starting and closing the
action mode for you. You can also override the standard `ActionMode.Callback` methods
on your `MasterDetailHelper` (e.g., `onActionItemClicked()`), though please chain
to the superclass to allow the library's implementations to do their work as well.

An `MasterDetailController` can additionally override `getActionModeTitle()` and
`getActionModeSubtitle()`, which will be called to populate the title and subtitle
of the action mode. `MasterDetailHelper` implementations would perform the same
work in methods like `onItemCheckedStateChanged()`, working directly with the
`ActionMode` instance.

### Empty Views and Multiple-Choice Views

Your `MasterDetailHelper` can override `buildListEmptyView()`, to return the `View`
that should be shown in the master area when there are no models in your collection
(e.g., your `PagerAdapter` returns `0` for `getCount()`). The default implementation
is a simple `ProgressBar`.

Similarly, you can override `buildDetailEmptyView()` to return the `View` to be
shown in the detail area when no model is chosen in the master. This only has an effect
in dual-pane scenarios; in single-pane scenarios (e.g., phones), the detail is only
shown if a model was chosen in the master. The default is simply a blank `View`.

If you elected to enable multiple-choice action mode support, you can also override
`buildDetailMultipleChoiceView()`, which should return a `View` that will be shown
in the detail area when two or more models are chosen in the master. Once again, this
only has an effect in dual-pane scenarios. You are passed a `SparseBooleanArray`
(culled from the `ListView` and its `getCheckedItemPositions()`) to let you know
which models were chosen, in case this affects your decision on what to show. The
default is simply a blank `View`.

### Custom Master Contents

If the simple title-of-the-page `ListView` rows in the master do not meet your needs,
you can override `getView()` on `MasterDetailHelper` to format your rows as you see
fit. Or, you can override `buildListAdapter()` and provide your own `ListAdapter`
for the master. In this latter case, make sure that the `ListAdapter` and `PagerAdapter`
contents line up (e.g., the first row in the `ListAdapter` maps to the first page
of the `PagerAdapter`).

### Custom "Dividing Line" for Strategy

The default behavior is to use the dual-pane display strategy when the device width
is `720dp` or higher. You can change this "dividing line" between single-pane and
dual-pane strategies by overriding `getMinimumDipWidthForDualPane()` on your
`MasterDetailHelper`, returning the new width in `dp`.

Advanced Configuration
----------------------
In addition to the configuration hooks specified above, you can do more
to tailor how your master and detail are presented.

### Custom Activity Base Class

If this all sounds neat, but you cannot extend `MasterDetailActivity`, simply copy
the logic from that implementation into your own activity (or activity base class).
Mostly, it is matter of forwarding select lifecycle methods and other event callbacks
to the `MasterDetailHelper` for processing.

Dependencies
------------
This project depends on the Android Support package v13 edition.

This project requires an `android:minSdkVersion` of 14 or higher.

Version
-------
This is version v0.0.1, meaning that it is so new, it's scary.

Demo
----
In the `demo/` sub-project you will find a sample project demonstrating the use
of `MasterDetailController` and `MasterDetailActivity`.

License
-------
The code in this project is licensed under the Apache
Software License 2.0, per the terms of the included LICENSE
file. Note that a slightly-modified version of
Justin Shapcott's `SplitPaneLayout`, included in this library, is also licensed under
the Apache Software License 2.0.

Questions
---------
If you have questions regarding the use of this code, please post a question
on [StackOverflow](http://stackoverflow.com/questions/ask) tagged with `commonsware` and `android`. Be sure to indicate
what CWAC module you are having issues with, and be sure to include source code 
and stack traces if you are encountering crashes.

If you have encountered what is clearly a bug, or if you have a feature request,
please post an [issue](https://github.com/commonsguy/cwac-camera/issues).
Be certain to include complete steps for reproducing the issue.

Do not ask for help via Twitter.

Also, if you plan on hacking
on the code with an eye for contributing something back,
please open an issue that we can use for discussing
implementation details. Just lobbing a pull request over
the fence may work, but it may not.

Release Notes
-------------
- v0.0.1: initial release

Who Made This?
--------------
<a href="http://commonsware.com">![CommonsWare](http://commonsware.com/images/logo.png)</a>

