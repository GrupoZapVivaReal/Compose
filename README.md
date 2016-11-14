[![Build Status](https://travis-ci.org/VivaReal/Compose.svg?branch=master)](https://travis-ci.org/VivaReal/Compose)
[![codecov](https://codecov.io/gh/VivaReal/Compose/branch/master/graph/badge.svg)](https://codecov.io/gh/VivaReal/Compose)
[![Version](https://img.shields.io/cocoapods/v/Compose.svg?style=flat)](http://cocoapods.org/pods/Compose)
[![Pod Platform](https://img.shields.io/cocoapods/p/Compose.svg)](http://cocoapods.org/pods/Compose)
[![Pod License](https://img.shields.io/cocoapods/l/Compose.svg)](https://github.com/vivareal/Compose/blob/master/LICENSE)
[![Carthage Compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

# Compose is a data driven library that will help compose your complex and dynamic Views.

It helps you create complex and dynamic Views using easy and simple composition of data structures. It is really easy to use and extend.

**And it also is a breeze to implement on existing projects.**

## First you will create units that encapsulates the data they will display
```swift
let info = "Info to be displayed"
let infoUnit = LabelUnit(text: info, traits: [.height(40)])
```
and then, you can add this unit to a container:

```swift
container.state = [infoUnit]
```
you can also conditionaly add this unit to the container, like:

```swift
var units: [ComposingUnit] = [....]
units.add(if: someCondition) {
	return LabelUnit(text: info, traits: [.height(40)])
}
container.state = units
```
So, instead of dealing with many dataSource/delegate methods you can just create an array of `ComposingUnit`s and assign it to a `state` property of a container.

## So, what are ComposingUnits?
**The protocol [ComposingUnit](https://vivareal.github.io/Compose/Protocols/ComposingUnit.html) is the heart of this framework.**

You can make any class or structure conform to it, taking the advantages of `Value Type` and `Reference Type` when they best suits your needs.
Also, this class/structure **should hold all the data that it will display**. With this approach, we don't need to hold a reference to the models that generated these units.

Let's say we want to display a list of all the feed items we have.

```swift
let feedItems: [FeedItem] = ... //You can grab an array from CoreData, JSON, Realm, anywhere...
var feedUnits: [ComposingUnit] = feedItems.map { FeedUnit(id: $0.uniqueId, title: $0.title, image: $0.image, likeCount:Int) }
container.state = feedUnits
```
So after we create the `feedUnits` array we don't need `feedItems` anymore and we can easily use our feedUnits in any thread.
**And we can add any other `ComposingUnit` to this array, allowing us to display a view totally different to a feed item in the same list**

```swift
feedUnits.add(if: feedUnits.count > 4) {
	return SeeMoreFeedsUnit(feedsCount: feedUnits.count)
}
```
### Handling selection and other delegates callbacks
To handle **cell selection** or other delegate callbacks, all your class/structure has to do is implement an applicable `Protocol`. For cell selection it is the `SelectableUnit` protocol. This protocol defines a method that will be called once the cell has been selected. 

You can check all [extension protocols here](https://vivareal.github.io/Compose/Extending%20Units.html)

### Grouping units to represent a single unit
You can also use a [CollectionStackUnit](https://vivareal.github.io/Compose/Structs/CollectionStackUnit.html) to group some units together as a single unit

```swift
var units: [ComposingUnit] = [...] //create somewhere
units.add(ifLet: data.optionalFeedItem) { feedItem in
	var innerUnits: [ComposingUnit] = [HeaderUnit(text: feedItem.title)]
	innerUnits.add(ifLet: feedItem.image) { image in
		return FeedImageUnit(image: image)
	}
	innerUnits.add(ifLet: feedItem.video) { video in
		return FeedVideoUnit(video: video)
	}
	innerUnits.append(ActionBarUnit(likes: feedItem.likes, comments: feedItem.comments))
	let unit = CollectionStackUnit(identifier: feedItem.uniqueIdentifier, direction: .vertical, traits: [], units: innerUnits)
	return unit
}
```
Maybe you could also create a function that returns this item based on a FeedItem

```swift
	func FeedUnit(from: FeedItem)-> CollectionStackUnit {
		var innerUnits: [ComposingUnit] = [HeaderUnit(text: feedItem.title)]
		innerUnits.add(ifLet: feedItem.image) { image in
			return FeedImageUnit(image: image)
		}
		innerUnits.add(ifLet: feedItem.video) { video in
			return FeedVideoUnit(video: video)
		}
		innerUnits.append(ActionBarUnit(likes: feedItem.likes, comments: feedItem.comments))
		let unit = CollectionStackUnit(identifier: feedItem.uniqueIdentifier, direction: .vertical, traits: [], units: innerUnits)
		return unit
	}
	
```

```swift
var units: [ComposingUnit] = [...] //create somewhere
units.add(ifLet: data.optionalFeedItem) { feedItem in
	return FeedUnit(from: feedItem)
}
```

### Dynamic cell size calculation
We use a struct called [DimensionUnit](https://vivareal.github.io/Compose/Structs/DimensionUnit.html) to represent a cell width/height calculation. A `DimensionUnit` can calculate a dimension using:
* Static values: It will ignore it's container size and always return this static value
* Percent based values: It will return a percentual of it's container dimension
* Total value based: It will return it's container dimension minus a static value
* Custom based: It will execute a closure passing it's container size as parameter.

Using `DimensionUnit` we can easily express our units height and width.

## ComposingContainer
In order to display an array of `ComposingUnit`s you will need an UIView that conforms to `ComposingContainer`.  

We provide two default containers in the framework: `ComposingCollectionView` and `ComposingTableView`. Both have **automatic detection of inserts/updates/deletes** in their state they are displaying.

## Tests
It gets really simple to test your interface, as you can test the presence of some specific unit, and you don't need to render your interface.

```swift
let dummyItem = FeedItem(...)
var feedUnit = FeedUnit(from: dummyItem)
XCTAssert(feedUnit.units.count == 3)
dummyItem.image = nil
dummyItem.video = nil
feedUnit = FeedUnit(from: dummyItem)
XCTAssert(feedUnit.units.count == 2)
```

## Examples

We provide some cool examples in our **Example project**.

To run, clone this repo, and open the `Example/Compose_Example.xcodeproj`. **You don't need to do any pod install or any configuration to run this project**

## Installation

#### CocoaPods
Compose is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your `Podfile`:

```ruby
pod "Compose"
```

#### Carthage
[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager that builds your dependencies and provides you with binary frameworks.

You can install Carthage with [Homebrew](http://brew.sh/) using the following command:

```bash
$ brew update
$ brew install carthage
```

To integrate Compose into your Xcode project using Carthage, specify it in your `Cartfile`:

```ogdl
github "VivaReal/Compose" ~> 1.0
```

Run `carthage update` to build the framework and drag the built `Compose.framework` into your Xcode project.

#### Manual
You can download this repo, drag the `Compose.xcodeproj` inside your project and link the `Compose` framework 
 
## Documentation
You can find all documentation about Compose here: [Documentation](https://vivareal.github.io/Compose/index.html)

## Author

Bruno Bilescky, [brunogb@gmail.com](mailto:brunogb@gmail.com)

## License

Compose is available under the MIT license. See the LICENSE file for more info.
