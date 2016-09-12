# <img src="https://raw.githubusercontent.com/alexdrone/Render/master/Doc/logo.png" width="444" alt="Render" />

[![Swift](https://img.shields.io/badge/swift-3-orange.svg?style=flat)](#)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![CocoaPods Compatible](https://img.shields.io/cocoapods/v/Render.svg)](https://img.shields.io/cocoapods/v/REnder)
[![Platform](https://img.shields.io/badge/platform-iOS-lightgrey.svg?style=flat)](#)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://opensource.org/licenses/MIT)
[![Gitter](https://badges.gitter.im/alexdrone/Render.svg)](https://gitter.im/alexdrone/Render?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

*React-inspired Swift library for writing UIKit UIs.*

#Why

From [Why React matters](http://joshaber.github.io/2015/01/30/why-react-native-matters/):

>  [The framework] lets us write our UIs as pure function of their states.
> 
>  Right now we write UIs by poking at them, manually mutating their properties when something changes, adding and removing views, etc. This is fragile and error-prone. [...]
> 
> [The framework] lets us describe our entire UI for a given state, and then it does the hard work of figuring out what needs to change. It abstracts all the fragile, error-prone code out away from us. 

## Installation

### Carthage



To install Carthage, run (using Homebrew):

```bash
$ brew update
$ brew install carthage
```

Then add the following line to your `Cartfile`:

```
github "alexdrone/Render" "master"    
```

#### CocoaPods
You can use [CocoaPods](http://cocoapods.org/) to install `Render` by adding it to your `Podfile`:

```ruby
platform :ios, '8.0'
use_frameworks!
pod 'Render'
```

#### Manually
1. Download and drop ```/Render``` folder in your project.  
2. Congratulations!

To get the full benefits import `Render` wherever you import UIKit

``` swift
import UIKit
import Render
```

#TL;DR

**Render**'s building blocks are *Components* (described in the protocol `ComponentViewType`).


Despite virtually any `UIView` object can be a component (as long as it conforms to the above-cited protocol),
**Render**'s core functionalities are exposed by the two main Component base classes: `ComponentView` and `StaticComponentView` (optimised for components that have a static view hierarchy).

**Render** layout engine is based on [FlexboxLayout](https://github.com/alexdrone/FlexboxLayout).

This is what a component (and its state) would look like:


```swift

struct MyComponentState: ComponentStateType {
	let title: String
	let subtitle: String
	let image: UIImage  
	let expanded: Bool
}

class MyComponentView: ComponentView {
    
    // The component state.
    var componentState: MyComponentState? {
        return self.state as? MyComponentState
    }
    
    /// Constructs the component tree.
  	override func construct() -> ComponentNodeType {

	    let size = self.referenceSize
	
	    return ComponentNode<UIView>(props: [
	      #keyPath(flexDirection): self.featured
	                               ? Directive.FlexDirection.column.rawValue
	                               : Directive.FlexDirection.row.rawValue,
	      #keyPath(backgroundColor): UIColor.black,
	      #keyPath(flexDimensions): self.featured
	                                ? CGSize(width: size.width/2, height: CGFloat(Undefined))
	                                : CGSize(width: size.width, height: 64)]).children([
	
	        // Image view.
	        ComponentNode<UIImageView>(props: [
	          #keyPath(image): self.album?.cover,
	          #keyPath(flexAlignSelf): Directive.Align.center.rawValue,
	          #keyPath(flexDimensions): self.featured
	                                    ? CGSize(width: size.width/2, height: size.width/2)
	                                    : CGSize(width: 48, height: 48)])	        ]),
	
	        // Text wrapper.
	        ComponentNode<UIView>(props: [
	          #keyPath(flexDirection): Directive.FlexDirection.column.rawValue,
	          #keyPath(flexMargin): UIEdgeInsets(top: 4, left: 4, bottom: 4, right: 4),
	          #keyPath(flexAlignSelf): Directive.Align.center.rawValue,]).children([
	
		          // Title.
		          ComponentNode<UILabel>(props: [
		            #keyPath(flexMargin): UIEdgeInsets(top: 4, left: 4, bottom: 4, right: 4),
		            #keyPath(text): self.album?.title ?? "None",
		            #keyPath(textColor): UIColor.white]),
		
		          // Subitle.
		          ComponentNode<UILabel>(props: [
		            #keyPath(flexMargin): UIEdgeInsets(top: 4, left: 4, bottom: 4, right: 4),
		            #keyPath(text): self.album?.artist ?? "Unknown artist",
		            #keyPath(textColor): UIColor.white]),
	        ])
	    ])

  }
    
}

```

[Check playground](Playgrounds/01%20Flexbox%20Components.playground)


The view description is defined by the `construct()` method.

`ComponentNode<T>` is an abstraction around views of any sort that knows how to build, configure and layout the view when necessary.

Every time `renderComponent()` is called, a new tree is constructed, compared to the existing tree and only the required changes to the actual view hierarchy are performed - *if you have a static view hierarchy, you might want to inherit from `StaticComponentView` to skip this part of the rendering* . Also the `configure` closure passed as argument is re-applied to every view defined in the `construct()` method and the layout is re-computed based on the nodes' flexbox attributes. 

The component above would render to:

<img src="Doc/render.jpg" width="454">

**Check the playgrounds for more examples**


###Lightweight Integration with UIKit

*Components* are plain UIViews, so they can be used inside a vanilla view hierarchy with *autolayout* or *layoutSubviews*.
Similarly plain vanilla UIViews (UIKit components or custom ones) can be wrapped in a `ComponentNode` (so they can be part of a `ComponentView` or a `StaticComponentView`).

The framework doesn't force you to use the Component abstraction. You can use normal UIViews with autolayout inside a component or vice versa. This is probably one of the biggest difference from Facebook's `ComponentKit`.

###Performance & Thread Model

**Render**'s `renderComponent()` function is performed on the main thread. Diff+Layout+Configuration runs usually under 16ms on a iPhone 4S, which makes it suitable for cells implementation (with a smooth scrolling).

###Live Refresh

You can use **Render** with [Injection](https://github.com/johnno1962/injectionforxcode) in order to have live refresh of your components.
Install the injection plugin, patch your project for injection and add this code inside your component class (or in your ViewController):

```swift

class MyComponentView: ComponentView {
	...
	func injected() {
		self.renderComponent()
	}
}

```

###Backend-driven UIs

Given the descriptive nature of **Render**'s components, components can be defined in JSON or XML files and downloaded on-demand.
*The ComponentDeserializer is being worked on as we speak*.


###Components embedded in cells

You can wrap your components in `ComponentTableViewCell` or `ComponentCollectionViewCell` and use the classic dataSource/delegate pattern for you view controller.

[Check playground](Playgrounds/04%20Components%20embedded%20in%20Cells.playground)

#TODO/Help?
There are a number of things that are still left to do.

- Support Swift package manager. 
- Improve performance (I think there's a lot of rooms for improvements!)
- More examples and demos



#Credits

- [React](https://github.com/facebook/react): The React github page
- [Few.swift](https://github.com/joshaber/Few.swift): Another React port for Swift. Check it out!
- [css-layout](https://github.com/facebook/css-layout): This project used the C src code for the flexbox layout engine.
- [Backend-driven native UIs](https://www.facebook.com/atscaleevents/videos/1708052886134475/) from [JohnSundell](https://github.com/JohnSundell): A inspiring video about component-driven UIs (the demo project is also inspired from Spotify's UI).


