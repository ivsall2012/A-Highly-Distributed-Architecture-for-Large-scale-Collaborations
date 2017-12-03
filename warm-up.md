[切换到中文](https://github.com/ivsall2012/A-Highly-Distributed-Architecture-for-Large-scale-Collaborations/blob/master/warm-up.md)

# Warm Up
The followings are some of the mistakes I made, learned lessons and experiences.  

## Purpose 
The purpose of this article is to give you more context in order to read the next one [the Service Router Architecture(SRA)](the Service Router Architecture(SRA)).  
If you think I'm wrong about something here, please DO let me know.  

## Content
- [1. The Whole Team Working On The Same Copy of the Project](#1-the-whole-team-working-on-the-same-copy-of-the-project)
- [2. Those "common", "Core" modules acting like a black hole sucking every "utility" into it](#2-those-common-core-modules-acting-like-a-black-hole-sucking-every-utility-into-it)
- [3. Horizontal Dependencies](#3-horizontal-dependencies)
- [4. Reversed vertical dependencies](#4-reversed-vertical-dependencies)
- [5. Use of Closure](#5-use-of-closure)
	- [Pros](#pros)
	- [Cons](#cons)
	- [So we should use it or not??](#so-we-should-use-it-or-not)
- [6. Presentation layer](#6-presentation-layer)
- [7. Networking](#7-networking)
- [8. Local Storage](#8-local-storage)
	- [CoreData](#coredata)
	- [Realm](#realm)
	- [SQLite](#sqlite)
	- [How to Shard Data Models](#how-to-shard-data-models)
	- [How to Transform JSON Data into Models](#how-to-transform-json-data-into-models)
- [9. How to Design A Multiple Delegate System](#9-how-to-design-a-multiple-delegate-system)


### 1. The Whole Team Working On The Same Copy of the Project
DON'T!!!  
You should at least use [Cocoapods's pod module](https://guides.cocoapods.org/making/using-pod-lib-create.html) to construct your project in pieces.  
And even you did it in this way, you will still have a dependency hell as your project grows bigger. That's why I'm writing those two articles for solving problems in large scale collaborations. Stay tuned!  

### 2. Those "common", "Core" modules acting like a black hole sucking every "utility" into it
DON'T!!!  
Try your best to make everything clear and explicit, no shady shady stuff!    
If you have a string extension, then create a pod module for that, e.g. [StringExtension](https://github.com/iOSModularization/StringExtension).  
If you have a bunch of strings used frequently, such as notification names or server domain names, put them into a categorized module, something like "ServerDomainNames" or "AudioPlayerNotifications".  
Then you import whichever you need, instead of importing all of them.  


### 3. Horizontal Dependencies
**A cross-module horizontal dependency** is the one which has strong business logic but not related inherently to current module. And you just need some of its classes or properties in order to use its services, such as use its view controller in order to present or push it. Another example, authenticate user's status by importing the Auth module. I call this kind of dependency "name-space invasion" -- they really don't need to know each other in order to work.  You should embrace a more modern approach: using a router architecture.  


### 4. Reversed vertical dependencies
**A vertical dependency** is the one which has relatively lower level than current module. For example a UI media player depends on a AVPlayer which doesn't care about where you get the URL resources from. And AVPlayer doesn't depend on the media player. AVPlayer here acts like a utility tool that **other project can use it too**. AVPlayer has lower level than the player, thus it's a "vertical" dependency to the UI media player in "vertical direction". 
So a reversed one is the lower module also knows(depends on) and manipulates its "super". For example, the AVPlayer now can manipulate your UI media player when it needs to play next video, which makes it not reusable thus it can't be used by everyone -- not a utility tool anymore.   

A horizontal dependency is a much bigger evil than a vertical dependency(not reversed).  


### 5. Use of Closure
#### Pros:  
Using a closure is a convenient way to enclosed operations and then execute it by any object at any time.   
We don't have to care about what's in it or import the related classes since closures capture execution contexts when it's created.  
If you use a closure as a method parameter, as some RAC or RxSwift developers suggest,  "you can see what happens before and after the execution, as opposed to delegate which you have to jump back and forth to investigate what happens".  

#### Cons:  
A closure also increases captured objects' lifetime, not just "self", all objects involved.  
All captured objects will have their retainCounter+1 in the code block, if you don't flag them as "weak" then make it strong inside the closure(block in Objective-C), the object might be nil and：  
1. In Objective-C, the object is nil and do nothing, most importantly, it would mess out your logics since the codes inside the block assumes the object is not nil.  
2. In Swift, the compiler though would warn you if you do "[weak object]" and you didn't use optional. And if you accidentally unwrap it, the program would crash, since Swift, unlike Objective-C, nil object's operation will crash the whole program.  


It might causes retain cycles, particularly three-way retain cycles.  
It's hard to debug.  
A closure can be passed and executed by any object at anytime in any thread -- it's just too flexible and not clear, too many uncertainties, at least this is how I feel about it!  
In a user's perspective, the closure(maybe a completion handler) has a really vague responsibility about what codes should be put in. It might get bigger and bigger, thus more objects involved and more risk of crashing and also reduce readability too.   

#### So we should use it or not??  
Each closure has its own execution context. So when your work really requires different context each time when it's called, for example, a network request method which can be called and given different completions by a 'mainPage' or 'userCenter', etc. you **can** use a closure. A muti-delegate system can do the job too. See below "My advise".

If you have some work required to do but you don't want to expose yourself and create a public protocol. For example, you want to do something when the login page finishes authenticating, then you should include a closure into a dictionary and pass it to the login page and let it extract the closure and execute it for you.  
NOTE: that closure should only have simple data types for its parameters and return value.  

A router like [AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter) which uses closure extensively. The reason is the router doesn't know and doesn't care about how you create your view controller or services. The only thing it cares is to do the already designated work when a user demands it.  

About how to use a closure properly as a user for networking see section [Networking](#networking).  


My advise:  
Use delegate and protocol until you can't! 
Please see [How to Write a Multiple Delegate System](#how-to-write-a -multiple-delegate-system) if you need a broadcast functionality for delegates.  



### 6. Presentation layer
The most common mistake is leaking business logics -- networking data/events/routing are all in one view controller.  
I know it's just convenient to import other modules and use its view controller by presenting or pushing it, or its properties and services.  
But you really should make a protocol and keep a strong reference for a manager object to handle them at least. 
You should always choose maintainability over convenience.  

The manager object mentioned above can also be called ViewModel in MVVM context.  
Here's my understanding of the VM, I think of it as a view controller helper, helping things more leaning toward views. For example, a tableView's dataSource and delegate are ViewModels as well. The dataSource gathers data and provides appropriate cells(view) to the tableView. And the delegate is even more obvious -- help view controller react to cells events.  

NOTE: Every business module should have a networking/persistence layer(as a manager) and presentation layer.   
NOTE: Do not let any data model gets involved into your presentation layer. Use simple data type such as dictionary to pass them from networking/persistence layer to presentation layer.  
NOTE: More details in next article [the Service Router Architecture(SRA)](https://github.com/ivsall2012/A-Highly-Distributed-Architecture-for-Large-scale-Collaborations/blob/master/SRA.md).  

### 7. Networking
There are two ways to design a networking SDK:  
1) It's a class or struct which uses singleton or uses static methods.  
2) It's a regular class or struct which can be created and used by anyone like regular object, like Apple's URLSessinTask -- they are all distributed.

Of course we are going to use the distributed ones haha :)  
Each view controller has its own network manager with strong reference to hold it.  
Imagine this scenario: your view controller is just being pushed into the stack. But the Internet is bad and the user doesn't want to wait too long. So he tapped "Back" or "<" icon to go back previous view controller.  
Now here comes the problems:  
A: How do we prevent the already sent out request crash our app through the completion handler?  
B: How are we going to do with the pending requests?  

The following example explains both of the problems.  


Let's assume a view controller asks its network manager for some user data.
NOTE: the vc holds a strong reference to the manager. And the manager conforms to a protocol of the vc. 
NOTE: the manager is not a delegate! It's a VM in MVVM context. A delegate object is always being held by a weak reference.
```Swift
class NetworkManger: SomeProtocol {
	lazy var networking = NetworkingObject()

	deinit {
		networking.cancelAllRequests()
	}

	func viewController(_ vc: theViewController, shouldLoadUserDataForUserId userId: Int) {
		// check cache...

		// The reason why the closure only weakens the vc, not self, or even both self and vc, is that
		// the vc holds a strong reference, so when the vc dies, this manager will die too then its "deinit" will be called. 
		// Thus when the user presses "back" to pop the vc before this callback gets executed, the callback closure wouldn't crash the app later when it actually lands because the vc would be nil. 
		// Additionally, all pending requests would be canceled as well in "deinit". 
		networking.requestUserData(byUserId userId) {[weak vc] (data) in
			guard let vc = vc else {return}
			guard let data = data else {return}
			// data transforming....
			// notify vc the data is ready
			vc.reload(data)
		}
	}
}
```  

### 8. Local Storage
We are only talking about databases here.  
#### CoreData   
It's definitely over-designed.  
Its data model, NSFetchedResultsController, managed object context are misleading people, particularly newbies into using them at every corner of the view controller, which is really bad idea.  
CoreData always reminds me of VIPER.    

#### Realm
I was using Realm for [AHFM](https://github.com/iOSModularization/AHFM) at the first version. AHFM is the project which uses [the Service Router Architecture(SRA)](https://github.com/ivsall2012/A-Highly-Distributed-Architecture-for-Large-scale-Collaborations/blob/master/SRA.md) entirely. You are about to read it later.  
At first, I found out that Realm just couldn't be packed into a pod module as a dynamic framework which can dramatically reduces the compile time. Because I wanted a persistence layer wrapped around it. The issue is [here](https://github.com/realm/realm-cocoa/issues/5230).  
Then I stupidly hacked my way out and made Xcode compiled it. And I didn't think it would be still a problem remained unsolved. Not to mention that every time I do a 'pod install' to that persistence layer module, it took me 3-5 minutes to finish the compilation which greatly affected my moods and faith in continuing developing the project. But since I'm a strong man so pushed through it until later in the development. I realize that problem which Xcode just wouldn't compile in a normal way thus it couldn't be a pod module.   
Since I'm a strong man again, I gave up using Realm and wrote my own SQLite data modeling framework:  
[AHDataModel](https://github.com/ivsall2012/AHDataModel) -- A protocol-based, object-mapping, lightweight complete database wrapper. It has easy-to-use APIs. Just check it out.  

Realm is like CoreData, tends to mislead people into using it and its data model classes everywhere in the program.  


#### SQLite
[SQLite.swift](https://github.com/stephencelis/SQLite.swift) and [GRDB.swift](https://github.com/groue/GRDB.swift) are both excellent choice.  
In mobile app development, you don't really have to go fancy about database. You are not developing a backend server. If the database claims it's really fast and it's faster loading 1000 records than SQLite. Then I wanna ask who would load 1000 records into a mobile app at once?  
Instead, make sure to use Instruments to test your project and see which method is making trouble for your tableView or collectionView scrolling.   

If the features are not needed, then they are not features, but more dangerous complexities.  

By the way [AHDataModel](https://github.com/ivsall2012/AHDataModel) is used in [AHFM](https://github.com/iOSModularization/AHFM) and so far so good.  

The followings are divided into two parts: 1) how to shard data models 2) How to Transform JSON Data into Models  

#### How to Shard Data Models
Normally, we don't put locally managed attributes into those networking related data models.  Other business modules should have their own local managed data models, for example, an audio progress model for track's last played progress, should be managed by an audioPlayer module, not the dataCenter SDK.   
 
Sample project: [AHFMDataCenter](https://github.com/iOSModularization/AHFMDataCenter)  
NOTE: AHFMShow and AHFMEpisode are two root networking data models, though AHFMShow is not pure, with properties like "totoalFileSize" and "numberOfEpDownloaded". I just didn't think it through at the time -- it's a mistake.  

**Further Improvements**: I should have given each of those data models its own module so that they are distributed -- which business module wants to use it, use it and which of them wants to create new local managed data models, should create a pod module for it too.  

In summary, the dataCenter SDK should only provide networking related data models and leave other local managed models to their business modules.  

NOTE: Every business module should have a networking/persistence layer(as a manager) and presentation layer.   
NOTE: Do not let any data model gets involved into your presentation layer. Use simple data type such as dictionary to pass them from networking/persistence layer to presentation layer.  
NOTE: More details in next article [the Service Router Architecture(SRA)]().  


#### How to Transform JSON Data into Models
By data models, I mean networking data models -- only contains data properties for JSON data from your server.  
You should create a data transformer for each networking data model, help JSON data transforms into that model.  So a business module's networking/persistence layer will do:  
* Receive a request from presentation layer asking for some data.  
* Make networking requests, check local storage if needed.  
* Use the correct data transformer to turn the JSON data into a networking data model.  
* Save that model and/or merge with other local managed models.  
* Turn those models into dictionaries and notify presentation layer through some load method, such as "vc.reload(userArray)"  

NOTE: More details in next article [the Service Router Architecture(SRA)]().  

Samples:  
**A**: [AHFMDataTransformers](https://github.com/iOSModularization/AHFMDataTransformers)  
I should have put both of those dataTransformers into separate modules, but I just didn't think it through. Yeah I was kinda lazy...  
**B**: [AHFMShowPageManager](https://github.com/iOSModularization/AHFMShowPageManger)  
A networking/persistence layer for [AHFMShowPage](https://github.com/iOSModularization/AHFMShowPage)  
**C**: [AHFMAudioPlayerVCManager](https://github.com/iOSModularization/AHFMAudioPlayerVCManager)  
A networking/persistence layer for [AHFMAudioPlayerVC](https://github.com/iOSModularization/AHFMAudioPlayerVC)  


### 9. How to Design A Multiple Delegate System
Q: What is a multiple delegate system?  
A: A multiple delegate system is a mechanism that can hold more than one delegates.  

Q: Why do I need a multiple delegate system?  
A: When you want to broadcast events, but you don't want to use an array of closures or notifications.  

There are two design obstacles:  
* How to keep all delegate objects **weakly**  
You can't use an array because Swift(and Objective-C) array keeps strong references to its members.  
We can create a struct and that struct has a "weak var delegate: SomeProtocol?" property.  
And instead of adding delegate objects into an array, we add those structs into an dictionary, for management convenience purpose.  

* How to add and remove them thread-safely and when
We need to put those two operations into our own serial queue.  
In fact, I lied, we only need one operation -- the add. We then check nil delegates every time that add method is called.  

Example codes:  
```Swift
public class Monitor: NSObject {
    fileprivate static var queue = DispatchQueue(label: "MonitorQueue")
    fileprivate static var delegateDict = [String: DelegateContainer]()
    
    public let shared = Monitor()
    
    public static func addDelegate(_ delegate: MonitorDelegate) {
        // make sure all the addings are in the queue for thread safety.
        self.queue.async {
            /// this array stores keys of DelegateContainer which has nil delegate objects.
            var neededToRemove = [String]()
            for (offset: _, element: (key: uuid, value: delegateContainer)) in self.delegateDict.enumerated() {
                if delegateContainer.delegate === delegate {
                    // duplicate
                    return
                }
                if delegateContainer.delegate == nil {
                    neededToRemove.append(uuid)
                }
            }
            
            for i in neededToRemove {
                self.delegateDict.removeValue(forKey: i)
            }
            /// We use dictionary with uuid as keys. If use array, all elements' indexes changes when you remove an element every time from an array.
            /// It's purely for management purpose.
            let uuid = UUID().uuidString
            let container = DelegateContainer(delegate: delegate)
            self.delegateDict[uuid] = container
        }
    }
    
    func didReceiveData(_ data: Data, size: UInt) {
        for container in self.delegateDict.values {
            container.delegate?.monitor(self, didReceiveData:data, size: size)
        }
    }
}
```

A complete sample project:  
[AHDownloader](https://github.com/ivsall2012/AHDownloader)







