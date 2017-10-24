# The Service Router Architecture(SRA)
## Requirements
[Cocoapods](https://github.com/CocoaPods/CocoaPods)  
Note: The word "module" mentioned in the rest of this article is referred to as [Cocoapods's pod module](https://guides.cocoapods.org/making/using-pod-lib-create.html).  

[AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter)  

## Content
- [Abstract](#abstract)
- [Introducing the Service Router Architecture(SRA)](#introducing-the-service-router-architecturesra)
	- [SRA Unit in Graph](#sra-unit-in-graph)
	- [Adding a Feature](#adding-a-feature)
	- [Deleting a Feature](#deleting-a-feature)
- [Why Use It](#why-use-it)
- [Compare to Uber's New Architecture](#compare-to-ubers-new-architecture)
- [More In Details](#more-in-details)
	- [Main](#main)
	- [Service](#service)
	- [Manager](#manager)
- [Podspec and Podfile](#podspec-and-podfile)
- [Mini But Complete Example](#mini-but-complete-example)
- [How To Develop and Test](#how-to-develop-and-test)
- [Real World Samples](#real-world-samples)
	- [Root Project](#root-project)
	- [AHFM](#ahfm)
- [Conclusion](#conclusion)

## Abstract
The reason why traditional MVC pattern becomes too large(AKA. massive view controller) is that the controller has too many logics in it -- all kinds of logics! And we need to help the controller to go on a diet by splitting its logics into several categorized classes. Additionally, the other two components, model and view, could get really big too. So that's why MVVM, MVCS, VIPER and other design patterns come to rescue by dissembling one or all of MVC components.  
And this kind of dividing responsibilities is called **vertical division** -- MVC being chopped into pieces, not complete anymore.  
On the other hand, a horizontal division is dividing responsibilities by its volume size -- the MVC is still complete but there's no much work for each of them.  
It's the same idea when you try to shard a database:  
1. Separate user information into different database by info categories, such as VIP related attributes, or order related attributes, etc. -- vertical sharding.  
2. Separate user information into several databases yet remains complete info for each record. -- horizontal sharding.  

The reason why MVC is so popular is because it's a vague and highly abstract(we humans love abstractions!) design pattern and it's also easy to understand and intuitive!    
Now let take a step back and ask, what is purpose of doing all these dividing? -- to reduce the weight of MVC pattern overall right? (VIPER evolves from MVC, don't be fooled by its fancy component names!)  
So what if we don't let MVC grow bigger, which means we don't have to chop it into pieces at all, do we?  
What if each small-enough MVC is a black box -- like a single lone galaxy weakly bounded by gravities from its distanced neighbor galaxies?  

NOTE: Gravitational force is the weakest force in the universe -- an object becomes significant attractive only when it has enough mass -- an elephant can't lift an ant by its gravity of mass -- that's why it takes a big LIGO to detect gravitational waves!  

It's like a horizontal sharding with size limits?  

If you want to learn more about designing a good iOS architecture and you also know Chinese,  
please make sure to checkout [this series of articles about designing iOS architecture](https://casatwy.com/iosying-yong-jia-gou-tan-kai-pian.html)! Make sure to come back though :)   


## Introducing The Service Router Architecture(SRA)  
SRA is a flat architecture and it divides all services and business logics into fined-grained SRA units, **horizontally** -- you can still use your old pal MVC!  
All SRA units are directed by [AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter) which acts like a central traffic control.  
A SRA unit consists of three modules: 1) main 2) service 3) manager.  


1. **Main**  
It is a small enough business UI module using MVC or MVVM Architecture(VIPER would be super redundant in such small module).   
It usually is just one page of screen.
It is INDEPENDENT and reusable -- you can reuse it in another project without changing a line!  
This module should be developed by exactly ONE developer!  

2. **Service**  
It is also an independent module and contains keys of services that the main module provides to the public.   This module contains only strings and is purely for routing purpose.  

3. **Manager**  
It registers service module so that other usrs can import it then use [AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter) to route to it or use its services. (AHServiceRouter routes by strings of keys).  
It handles and/or processes data and events from or to the main module.   
It also imports other service modules and decides which page to route according to those events.

#### SRA Unit in Graph
You might need to read it a few times
![](https://github.com/ivsall2012/A-Highly-Distributed-Architecture-for-Large-scale-Collaborations/blob/master/SRA_unit.png)   
As you can see from the image above, the only direct dependencies are from those services and AHServiceRouter which are all related to routing and not about business logics or UI logics. And they are distributed. Considering a main module has a weak delegate "dependency" so it's independent.    
Now only left the managers which send out network requests and process data, are somewhat dirty but it's very limited.  

Now you can imagine that a project has 10,20,30 of SRA units and they will be still "flat" like a hash map!   

#### Adding a Feature
Say if you want to add a feature of SRA unit.  
You should import related service and manager modules into your current manager module. You need to then register it by call that manager's "activate()" method, the method name is up to you when you design it. Then you use the services of that feature from there.  

#### Deleting a Feature
If you want to delete a feature of a SRA unit, you just don't register it.   
And you can use AHServiceRouter's fallback method to create a default view controller.  
Further, you can delete the routing logics(it's only AHServiceRouter usage codes) of that feature by searching its service's name in all managers and simply delete them. Total drama-freed!  

## Why Use It
First of all, as the project grows, the overall complexity of integrating a feature is constant. Because the whole architecture is flat and all its SRA units are like plug-ins -- you want to use it, use it. IF don't, unplug it.   
Second of all, the main and service modules are ABSOLUTE INDEPENDENT -- no business logic involved -- easy to debug.  
Third, the manager has clear and highly limited responsibilities to only event handling and networking -- you know where to hunt the bugs if any.  
Fourth, for the main module, you can literally use any architecture you want due to its independent nature. But MVC and MVVM are highly recommended and VIPER is just too fancy to use for such small fined-grained module.  
Fifth, who doesn't like a hash-map-styled architecture??  


## Compare to Uber's New Architecture
Uber has an [article](https://eng.uber.com/new-rider-app/) about using an improved version of VIPER to develop their app.
Basically, they use an improved version of VIPER, which has inheritance involved in its router, as a basic unit of feature.(correct me if I'm wrong. The article is just so vague!)  

NOTE:  
**A cross-module horizontal dependency** is the one which has strong business logic but not related inherently to current module. And you just need some of its classes or properties in order to use its services, such as use its view controller in order to present or push it. Another example, authenticate user's status by importing the Auth module. I call this kind of dependency "name-space invasion" -- they really don't need to know each other in order to work. Using [AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter) will do the job for you.(shameless ad :p )  

**A vertical dependency** is the one which has relatively lower level than current module. For example a UI media player depends on a AVPlayer which doesn't care about where you get the URL resources from. And AVPlayer doesn't depend on the media player. AVPlayer here acts like a utility tool that other project can use it too. AVPlayer has lower level than the player, thus it's a "vertical" dependency to the UI media player in "vertical direction".  
Vertical dependency is usually OK in a SRA unit.  

**Question 1**: How are you going to handle cross-module horizontal dependencies?  
If you use horizontal dependencies like those vertical dependencies, then what is the difference than a large MVC project with a fined "division of labor"?   
One module changes, other modules which has one-way or two-way dependencies with it, have to change too -- a dependency hell rolls out!  

YES, VIPER has a router yet it has to import everything in order to route -- makes it not distributed not an independent "judge", not to mention it uses inheritance!  
Those "super-child", "father-son" relationships sound "fancy" and "smart" -- if the child couldn't react to a particularly event, then its super class would handle it for you, how handy!  
It is just mirroring UIResponder. Ask yourself, do people use UIResponder or UIView in large scales?  
Inheritance has a higher chance to become shady.  
Make things as clear and explicit as you can!!  
Use more protocol and its default implementation feature, and (private)extensions, until you can't!!  


What SRA does is use keys of services which should be carefully planed like you do for public APIs and routes by [AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter) which is a good independent "judge". A manager only knows services needed and the router, plus vertical dependencies, e.g. networking and database SDKs. Main module is independent, freed of networking and database codes!  



**Question 2**： How are you going to dispatch developers to those vertically divided modules?  
If you dispatch several developers for a single unit, then   
A) you might have to wait till all modules finish at least some level in order to test it as a whole due to the nature of vertical division. 
B) you might have a messy internal dependencies due to complex VIPER structure.  

If you have one developer for one VIPER unit then the advantage of having a VIPER structure is lost, that is every VIPER component has clear responsibilities so that everybody has small portion of the whole thing and knows what they are doing.  
Well, you might say, the unit will grow bigger. Then you should see above A and B.

What SRA does is strictly suggests that one developer for one SRA unit. One SRA unit can be independently tested without any business environment setup involved!    



### More In Details
NOTE: If you feel confused or overwhelmed by the followings, it's OK. Go though those concepts and instructions first. Then there are more sample SRA units for you to learn from. You have this article as theories/instructions and these sample projects, you will get it!  
Remember that the greater good of learning this architecture is to make your project development a lot more easier in a large team, less bugs, more time to do real things!  
Even in a small team, SRA is also easy to setup, why wait till it gets bigger??  

NOTE: SRA uses [AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter) extensively, you should go through its usages if needed.  

#### Main  
What the main module is usually responsible for only **one page of screen**. If there are more complex UI logics involved, you should make that logics into another SRA unit.  
The main module must be INDEPENDENT and reusable!   
Here how you archive this goal by the following rules:  
1. Do NOT use any outside data models, such as post model, user model, or audio item, etc.  
Instead, use dictionary to receive data and turn them into your own data model used only within your module.  
```Swift

internal struct User {
	var id: Int
	var name: String
	init(_ dict: [String: Any]) {
		self.id = dict["id"] as! Int
		self.name = dict["name"] as! String
	}
}
/// This should be called by the manager. the VC itself doesn't usually make the call.
public func loadUserData(_ data: [String: Any]?) {
	guard let data = data else {return}
	/// We assume the data has the correct info here since the manager will verify this.
	self.user = User(data)
}
```
2. Delegate all the page routing logics, such as pushing a VC after a button being tapped.  
3. Delegate all the requests for incoming data, such as requesting user data.  
4. When using delegate to pass data into delegate methods, use simple data embedded into method parameters such as userId, trackId, or even an array of IDs, not your data model!  
```Swift
func audioPlayerVCListBarTapped(_ vc: AHFMAudioPlayerVC, trackId: Int, albumnId: Int)

```   
5. If you actually have more **simple** pages in your main module and it would be too small for them to become a SRA unit, such as a profile module which might contains many simple pages.  
You should delegate all events happened in those simple pages and pass them back to the main view controller, then to the manager which could help route to other pages or save data into database. 

You can think of the main modules like a UITableViewController or UICollectionViewController which has data and event delegates.  And you, as a user, have to call reload() method to tell them to refresh their UI components. Or you have to implement "tablView:didSelect" method in order to react to selection. Here those two view controllers only do what they need to do and delegate out anything not related to them.  

The same idea applied here: the main module notifies its delegate what's going on and the manager which is the delegate object, reacts to it.  
After some processes and data manipulations, the manager then call some sort of **loading method** to tell main module to refresh its UI components.  

So the only things should be exposed to outsiders are the active notifying **delegate methods** and passive **loading methods**. You don't care about how delegate object making networking requests, what data model it's using, nor who is this delegate object actually is.  You will see some of the "loading methods" later at [Mini But Complete Example](#mini-but-complete-example).   


#### Service
Service modules are public-facing providing services of strings to outside users. They have nothing to do with either their own main module or manager module.  
It is basically a bunch of public strings. The followings is an example:
```Swift
public struct UserListServices {
		public static var service: String {
        return "\(Self.self)"
    }
    
    /// To navigate to a VC
    public static var taskNavigation: String {
        return "taskNavigation"
    }
    
    /// To create a VC and return it to the caller
    public static var taskCreateVC: String {
        return "taskCreateVC"
    }
    
    /// The key to extract the VC from a userInfo dictionary
    public static var keyGetVC: String {
        return "keyGetVC"
    }

}
```

#### Manager  
1. It has a public "activate()" method which is in charge of initializing/registering the services that this whole SRA unit provides to the public.(You have register services first before use them)  
2. It handles data and events from the main module, as a delegate object.  
For example, it could initiate a networking request to get user data according to user id passed by its main module through its delegate method.  
Then the manager notifies the main controller, in the main module, by using KVC to perform selectors for "loading methods". See [Mini But Complete Example](#mini-but-complete-example)

3. It also imports other service modules and decides which page to route according to those events, provided that all needed services are actually activate(or registered) through AHServiceRouter before your manager uses them.  

NOTE: Though you can add the main module as the manager's dependency in ".podspec" file. And use it there by conforming the delegate. The manager is still the **only user** of the main module.  
But this dependency is unnecessary. You as the one who develops and maintains the manager and the "fined-grained" main module, should clearly know what methods to implement, which page to route to!   
Why introduce extra name-space of complexity when you don't have to?  

So a manager handles A) its own public services and B) uses other public services then C) interacts with its main module through delegate and uses KVC performSelector for "loading methods" to notify when data is ready.  

### Podspec and Podfile
Before you begin to develop and test this whole idea of SRA, you should know the differences between a ".podspec" and "Podfile":  
1) A '.podspec' file describes the behaviors that how Cocoapods should pack your framework or project hosted in a version control sites when a user do a 'pod install'.  
2) A 'Podfile' file describes how Cocoapods should download, such as what version, from where.  
For more information, checkout [Cocoapods Guides](https://guides.cocoapods.org/) and [Using Pod Lib Create](https://guides.cocoapods.org/making/using-pod-lib-create.html). 


### Mini But Complete Example
Here's a mini but complete example of a SRA unit:
```Swift
///###### ShowPageServices Module --> ShowPageServices/Classes/ShowPageServices.swift
public struct ShowPageServices {
	let static taskNavigation = "taskNavigation"
	/// This key is mandatory
	let static keyShowId = "keyShowId"
}


///###### In ShowPageManager module --> ShowPageManager/Classes/ShowPageManager.swift
import AHServiceRouter
// we need its own services in order to extract 'showId'
import SettingPageServices

/// A base protocol to discipline managers, though it only has one method now, there could be more later in the development. 
public protocol ModuleManager {
		/// initialize/register all related services in this method
    static func activate()
}

/// The activate() is usually called within the "application didFinishLaunchingWithOptions" method.
public struct ShowPageManger: ModuleManager {
	/// this method should be called before the module being used!
  public static func activate() {
  	/// register ShowPageServices
    AHServiceRouter.registerVC(ShowPageServices.service, taskName: ShowPageServices.taskNavigation) { (userInfo) -> UIViewController? in
   			guard let showId = userInfo[ShowPageServices.keyShowId] as? Int else {
            print("You must pass a showId into userInfo")
            return nil
        }
        /// Use KVC to initiate ShowPageVC without import that module
    		let vcStr = "ShowPage.ShowPageVC"
      	guard let vcType = NSClassFromString(vcStr) as? UIViewController.Type else {
            return nil
      	}
        let vc = vcType.init()
        let manager = Manager()
        /// manager keeps initial data and react to requests from the showPage VC through delegate methods.
        manager.showId = showId

        /// Since ShowPage VC holds strong reference to the manager, when the VC dies, this manager dies too.
        vc.setValue(manager, forKey: "manager")
        return vc
    }
  }
}

/// 'ShowPageManager/Classes/Manager.swift'
/// This is the actual 'Manager' that handles data and events.
class Manager: NSObject {
    public var showId: Int?
    lazy var networking = NetworkingObject()

    /// Since ShowPage VC holds strong reference to this manager, when the VC dies, this manager dies too.
    deinit {
    	/// Cancel all networking requests when manager dies.
    	networking.cancelAllRequests()
    }

    /// Call loadInitialShow(_ data: [String: Any]?) when ready
    func showPageShouldLoadInitalShow(_ vc: ShowPage) {
    	guard let showId = self.showId else {return}
    	// networking for that showId
    	networking.requestShow(byShowId: showId) {[weak vc] (showData) in
    		/// prevent the VC being destroyed before the data lands.
    		guard let vc = vc else {return}
    		guard let showData = showData else {return}
    		// check showData formats!! It's a [String: Any].
    		vc.perform(Selector(("loadInitialShow:")), with: showData)
    	}
	}
		/// Route to some other page
    func showPage(_ vc: ShowPage, didTapShowCover showId: Int) {
    	guard let navVC = vc.navigationController else {
            return
      }
    	/// navigate to SettingPage, not using the showId parameter here though.
    	AHServiceRouter.navigateVC(SettingPageServices.service, taskName: SettingPageServices.taskNavigation, userInfo: [:], type: .push(navVC: navVC), completion: nil)
	}
}

///##### In ShowPage module --> ShowPage/Classes/ShowPageVC.swift
/// This delegate is the one in charge the whole module's events.
public protocol ShowPageDelegate: class {
	func showPageShouldLoadInitalShow(_ vc: ShowPage)
	func showPage(_ vc: ShowPage, didTapShowCover showId: Int)
}

class ShowPageVC: UIViewController {
	var manager: ShowPageDelegate?

	///**** Loading Methods
	/// This method will be called by the manager.
	func loadInitialShow(_ data: [String: Any]?) {
		guard let data = data else {return}
		/// assume an internal "Show" model takes a dictionary.
		self.show = Show(data)
		/// refresh UI and
		/// do something else
	}

	///**** VC Life Cycle
	override func viewWillAppear(_ animated: Bool) {
		super.ViewWillAppear(animated)
		/// ask manager for initial show data, "loadInitialShow(_:)" will be called when data is ready.
		self.manager?.showPageShouldLoadInitalShow(self)
	}

}
```  

## How To Develop and Test  
NOTE: The followings are the formal steps in developing and testing a SRA unit. (there's another way too later this section)   
### Develop the Main
You should develop your main module first by creating a project using "pod lib create YourMainProject". Then you develop your delegate methods and think about what data you really need and what events you should tell the delegate. Now you assume your delegate works perfectly and start develop UI logics for your main module.   

### Develop the Service  
You need to be really clear about what services you can provide to the public and you shouldn't change those service keys often -- a version is required.  

### Develop the Manager
When developing the manager, you first copy all the delegate methods from the main module, then implement them. Remember to add other service modules(NOT their main nor manager module) as dependencies in the **".podspec"** file in your manager's pod project, if it requires routing to other pages. 
Finally you should use KVC "performSelector" to notify the main view controller that the data it requested is ready by calling its "loading methods", if any. 

NOTE: Usually the one who develops main module's delegate methods is the one who develops the manager, if you have more than one person working on the same SRA unit, which it's highly NOT recommended. If a SRA unit is too big, you should always separate them into two or three units! **One developer for one SRA unit**.  

### Test Them In Example Project All Together
In ShowPage module's "Podfile", add dependencies for testing.  
```ruby
### 'ShowPage/Example/Podfile'
... other codes

### In order to use AHServiceRouter to route to ShowPage, we need its service module and activate() from the manager, in the Example project. 
pod 'ShowPageShowPageServices'
pod 'ShowPageManager'

# we need to use SettingPageManager to activate() too since we use its services in the manager
# when testing we always import the needed managers and main modules!
pod 'SettingPageManager'
# SettingPage's main module
pod 'SettingPage'

... other codes
```  

In the Example project  
```Swift
///#### 'ShowPage/Example/ShowPage/ViewController.swift'
import UIKit
import AHServiceRouter
import ShowPageShowPageServices
import ShowPageManager
import ShowPage

/// We use SettingPageServices in the manager, so we need to activate it here
import SettingPageManager

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        /// Activate first!!
        ShowPageManager.activate()
        SettingPageManager.activate()
    }
    /// touch to test it
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let navVC = self.navigationController else {
            return
        }
        let type = AHServiceNavigationType.push(navVC: navVC)
        let info: [String: Any] = [ShowPageServices.keyShowId: 12345]
        AHServiceRouter.navigateVC(SettingPageServices.service, taskName: SettingPageServices.taskNavigation, userInfo: info, type: type, completion: nil)
    }

}
```


Hooray! You just finish your first SRA unit development!  


#### Where to Import the Real stuffs, the main modules in beta test and production??
So now:
1. The manager has dependencies of services, including its own(need to extra info by the keys).  
2. And it uses KVC to notify its main view controller in main module.  

Then who's going to activate(or register) those services and who's going to import the actual main modules eventually?  
The answer is in the root project. The root project has all needed managers and main modules dependencies in its ".podspec" file and activate them in its manager's "activate()" method. See [Root Project](#root-project)  


#### Another Way of Developing and Testing a SRA Unit  
NOTE: You have to really understand the differences between a ".podspec" file and a "Podfile" file!!  
[Podspec and Podfile](#podspec-and-podfile)  

The way I develop and test a SRA unit is I will first develop the main module. Then develop the service and manager modules in the main module's example project. Add related pod dependencies in the example project's "Podfile":
```ruby
### 'ShowPage/Example/Podfile'
.... other codes
# NOTE: We don't need to add dependencies for 'ShowPageServices' and 'ShowPageManager' since we are developing them in this example project.

# Those two dependencies will be handle in the root project in beta and production.
# SettingPage's main module.
pod 'SettingPage'
pod 'SettingPageManager'

.... other codes
```  

In Example Project
```Swift
///#### 'ShowPage/Example/ShowPage/ViewController.swift'
import UIKit
import AHServiceRouter
import SettingPageManager

// We don't need to import 'ShowPageServices' and 'ShowPageManager' since we are developing them in this example project already.
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        /// Activate first!!
        ShowPageManager.activate()
        SettingPageManager.activate()
    }
    /// touch to test it
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let navVC = self.navigationController else {
            return
        }
        let type = AHServiceNavigationType.push(navVC: navVC)
        let info: [String: Any] = [ShowPageServices.keyShowId: 12345]
        AHServiceRouter.navigateVC(SettingPageServices.service, taskName: SettingPageServices.taskNavigation, userInfo: info, type: type, completion: nil)
    }

}
```  

Then when I finish testing them in the main module, I will rearrange them into the form that [Mini But Complete Example](#mini-but-complete-example) describes.  


If you feel confused about all those, it's OK. In the following section you will have a lot of sample SRA units(more than 10 of them) you can go through and read this article and that [SRA Unit in Graph](#sra-unit-in-graph) again. And you'll be good to go!  
[SRA Unit in Graph](#sra-unit-in-graph)
## Real World Samples
### Root Project
A root sample module -- it merges all main modules and managers and activate them.  
The reason why a root module has service and manager is that I lied;)  
[AHFM](https://github.com/iOSModularization/AHFM) is really the root but it doesn't have anything that could demonstrate how a root project set up in a SRA manner.   
[AHFMMain](https://github.com/iOSModularization/AHFMMain)  
[AHFMMainServices](https://github.com/iOSModularization/AHFMMainServices)  
[AHFMMainManager](https://github.com/iOSModularization/AHFMMainManager)  

And another wrapper for that root sample(AHFMMain). It's basically just a skeleton.    
[AHFM](https://github.com/iOSModularization/AHFM)  

### AHFM
I have created a big project using purely SRA [AHFM](https://github.com/iOSModularization/AHFM) and all its modules are in [iOSModularization](https://github.com/iOSModularization)(more than 50 of them). 

[Sample SRA units](#https://github.com/iOSModularization/AHFM#feature-demos-by-modules)



## Conclusion
There's no "free lunch" in the world.    
Using SRA also has costs:  
1. It requires that you plan each SRA unit carefully, just enough small.  
2. SRA uses a lot of modules.  You need to formalize naming conventions for them, particularly those services modules and their keys.  
3. It requires your teammates to document their public service module extensively, answering questions like "does this module supports both navigation and creation of the main view controller, or just navigation", "what key-value pairs should be included when do a task and which ones are mandatory", etc.  
4. It has a somewhat steep learning curve for newbies, particularly those who don't even use pod modules, other than the "Podfile" to add dependency of "Alamofire".  But hey, which large-scale system or architecture doesn't require a relatively steep learning curve? Building a large scale application requires lots of disciples to make it clear and maintainable!!  

And that's it for "The Service Router Architecture".  
Shoot me an issue if you have any questions or agreements/disagreements.  

## Reference
Again, make sure to checkout [this series of articles about designing iOS architecture](https://casatwy.com/iosying-yong-jia-gou-tan-kai-pian.html) if you at least can read Chinese!!  


