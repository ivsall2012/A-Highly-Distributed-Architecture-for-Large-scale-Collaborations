[Switch to English](https://github.com/ivsall2012/A-Highly-Distributed-Architecture-for-Large-scale-Collaborations/blob/EN/warm-up.md)

# 前言
以下的这些都是我之前所犯过得错误和一些自认为比较好的经验。在看我的[组件化方案](https://github.com/ivsall2012/A-Highly-Distributed-Architecture-for-Large-scale-Collaborations/blob/master/SRA.md)之前, 我觉得这些错误和经验值得分享一下。
如果有什么地方不对的，值得讨论的，请各位一定要提出来😃  


## 目录
- [1. 整个团队都在一份项目里进行开发](#1-整个团队都在一份项目里进行开发)
- [2. 项目中黑洞似的"core","common"](#2-项目中黑洞似的core-common)
- [3. 横向依赖](#3-横向依赖)
- [4. 垂直依赖](#4-垂直依赖)
- [5. 闭包(block)的使用](#5-闭包block的使用)
    - [Pros](#pros)
    - [Cons](#cons)
    - [什么时候用](#什么时候用)
- [6. 展示层](#6-展示层)
- [7. 网络层](#7-网络层)
- [8. 本地持久化](#8-本地持久化)
    - [CoreData](#coredata)
    - [Realm](#realm)
    - [SQLite](#sqlite)
    - [怎样切割数据模型](#怎样切割数据模型)
    - [怎样把JSON转成原始模型](#怎样把json转成原始模型)
- [9. 一个多代理设计模式](#9-一个多代理设计模式)


### 1. 整个团队都在一份项目里进行开发
千万不要这样做！！！
你至少要把你的各个功能用[Cocoapods的pod module](https://guides.cocoapods.org/making/using-pod-lib-create.html)进行项目颗粒化。把这些pod都放到你们自己的私有库里面，要什么就导入什么，用就得了。然后创建一个main project 作为一个主项目把所有的功能都组织进去。这样你还可以方便地在各个颗粒化的pod里面做单元测试。而且你也少了很多在GitHub上的conflit，特别是storyboard, xib的conflict。  
你可能觉得你们团队也就那么2、3个人，根本不要紧的。但是你的公司，你的团队，你的用户需求是会增长的啊（期待是这样吧）。为什么要等到问题来了再解决呢。一开始就应该把一个健康的架构给搞好，这样更会提高开发效率 -- 每个开发者都有他自己的小天地，每个功能都对应着一个pod 一目了然。  
我的下一篇[组件化的文章](https://github.com/ivsall2012/A-Highly-Distributed-Architecture-for-Large-scale-Collaborations/blob/master/SRA.md)提出的方案会比这样只用[Cocoapods的pod module](https://guides.cocoapods.org/making/using-pod-lib-create.html)更好。当然先看完这篇文章先吧 哈哈。  

### 2. 项目中黑洞似的"core", "common"
不要把"core","common"当作垃圾堆，使劲往里扔东西，等到真正需要东西的时候，你才到垃圾堆里找东西，各种“腐臭味”。  
我建议你“用到什么就导入什么”。你尽力把各个模块的职能分类好，比如ServerDomainNames，AudioPlayerNotifications。在比如StringExtension就可以专门用来处理string的，甚至可以 StringPrettyForLogin 专为login模块用的string extension，名字长不要紧的。  
把这些模块都做成pod module。要用什么导入什么，一清二楚。而不是 你只需要用那么1、2个功能，而把那个又肥又大的"common"给导入进来， 入侵你现有的名字空间，没必要。导入的模块越少越好，这样独立性会好很多。     
你可能会问很多开源框架不是都有"core","common"吗？  
但我想问，你的“core”真的是CORE吗？人家框架的"core"：1）多数情况，随着时间的推移，并不会变得很大。而你的会。 2）而且在框架内，用到的那个"core"里的功能的确非常非常频繁，有点像系统的kernel,没有它框架运行不起来。而你的"core"里的东西 用得不那么频繁或者专一性高 如只处理string的一些功能。  
我并不是说绝对不用，如果你真正有core等级的功能和类，就放里面去吧。   
但是分门别类是个很好的习惯哦:)  


### 3. 横向依赖
横向依赖就是2个同等级的模块之间的依赖，比如说一个商品模块和一个订单模块，一个播发器界面模块和一个专辑首页模块。但一个登陆验证模块和一个主页模块就不是横向依赖，是下面会提到的垂直依赖。  
总的来说是业务模块之间的依赖。我们需要横向依赖 来使用其他业务模块的服务或者跳转到它们的页面。  
其实横向依赖本身是很“合理的”，比如一个商品模块里的“立即购买”被点击之后，肯定要跳到订单模块的啊，依赖订单模块有错吗？    
在应用还小的时候，几乎没有，但随着需求和应用的增大，模块与模块之间的就不独立了。  
如果你要测试商品模块，那么它所依赖的订单模块肯定会被导入，因为你需要依赖订单模块的跳转代码通过编译。你可能说测试的时候注释掉呗。但如果你又有一个商铺详情模块，还有一个购物车模块，还有一个客服模块等等，而且跳转代码分布在各个角落，怎么办？  
最好的办法就是去掉这些模块依赖，知道外界的信息越少越好越容易测试和调试。这样我们就需要一个中间件或者一个router来帮我们跳转到这些界面或者使用其他模块提供的服务。 下一篇组件化的文章，主要就是解决这个横向依赖的问题，让所有业务模块独立出来，整个项目就是一个即插即用的系统。  


### 4. 垂直依赖
垂直依赖就是服务模块与被服务模块之间的关系。比如，一个登陆验证模块和一个主页模块，一个播放器模块和播放器界面模块，一个网络模块和一个下载模块。简单地说就是高层业务模块依赖于底层服务模块。这个**单向**的，从上到下的依赖就叫做垂直依赖。  
需要注意的是，这里的底层服务模块是绝对不知道谁用了它的（没有对上层依赖），更不会去修改被依赖模块里的对象的数据。它只做它份内的事去服务外界而不影响外界。  


### 5. 闭包(block)的使用
#### Pros:  
使用闭包或者block是一个非常快捷方便的方法来保持各种操作以便晚些时候使用。  闭包"包闭"的就是一个执行上下文(execution context) -- 记录当时代码执行的状态还有操作，是哪个地址，哪个对象，哪个值需要改变成什么。你可以把闭包传来传去，谁都可以执行一次或多次。有点像其他语言的lambda函数或者匿名函数。  

闭包之所以方便就是它有了执行上下文后，它不需要提供者（定义闭包的是提供者，写闭包操作的是使用者）定义一个协议 然后使用者遵循协议。提供者定义好闭包后，使用者直接写他要在闭包内干什么就行了, 提供者决定什么时候执行闭包。弹性非常大。这也是为什么[AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter)，我下一篇文章组件化方案的核心库，运用闭包来提供注册(组件对外提供的服务和操作)的原因。因为[AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter)根本没办法知道哪个模块需要注册，定义怎样的协议方法。但是如果运用闭包，就不用关心这些了。一个组件只要注册的时候，给router一个闭包就可以了。然后当需要的时候，router执行那个闭包。router根本不关心那个组件需要怎样的方法和操作。另外一个是网络请求库，如AFNetworking(Alamofire), 它们不知道，也不关心是谁发出的请求，更不知道该怎样处理返回的结果，因为每个请求者都用不同的数据转换需求。所以闭包很适合做网络请求的回调。  

当你使用链式的闭包的时候，比如使用ReactiveCocoa或者RxSwift的时候，它们都是一个操作接着一个操作，看它们的代码跟看故事一样，一件事接着一件事 -- 所有事件处理都集中到了一个地方，方便阅读。  

当然也有其他高级炫酷的高级用法，在这里只提一个。  
在[AHAudioPlayer](https://github.com/ivsall2012/AHAudioPlayer)的代理方法中，有一个方法是需要代理对象提供音频的专辑图片。但代理对象可能需要发送网络来获取图片，所有那个代理方法是这样设计的：  
```Swift
func playerMangerGetAlbumCover(_ player: AHAudioPlayerManager,trackId: Int, _ callback: @escaping(_ coverImage: UIImage?)->Void)
```  
有必要说清楚的是，这里的定义闭包的提供者是“AHAudioPlayer”，它自己同样也是写闭包操作的使用者。这里让**代理对象**来决定什么时候执行闭包。一般的闭包使用是，提供者定义闭包和决定什么时候执行闭包(联想网络库)，然后使用者写闭包操作就可以了。这里是把闭包执行权给了一个代理对象。因为闭包是可以传来传去的。   
[AHAudioPlayer](https://github.com/ivsall2012/AHAudioPlayer)作为提供者，定义了一个闭包，规定使用者在获取到图片后，把它放进闭包的参数里执行就可以了。“AHAudioPlayer”在使用代理对象调用那个代理方法的时候，就已经创建好了闭包然后传进代理方法里。  


#### Cons:  
闭包延长了内部对象的生命周期， 它把内部的对象的 retainCount 都加了个1，不只是"self"。 只有所有闭包内的操作完成后，所有闭包的对象才会 retainCount-1。  
闭包还会很容易引起循环引用，特别是3方循环引用， 如 VC -> obj -> block -> VC。  
还有闭包能被任何人在任何线程，任何传递和执行 -- 这个实在是太广泛了，职能不够清晰，太多不确定性。   

#### 什么时候用  
如果你有事情要外界做，但你又不想定义一个协议和代理对象，比如，你想让用户登录后才能做某操作，那么你需要给登录器一个闭包（登录器定义好的），然后当登录器成功登录后，它会帮你执行闭包。登录器根本不需要知道闭包是干什么的,有什么对象，因为每次登录成功后的操作很可能不同。  

还有个好的例子，就是组件化的router设计。[AHServiceRouter](https://github.com/ivsall2012/AHServiceRouter)就是用闭包注册服务和任务的。它根本不关心谁会注册什么服务和任务，更不关心谁需要执行服务提供的任务。  它只知道保存服务和任务，然后当有人需要的时候，执行那个注册好了的闭包就可以了。  

NOTE: 闭包的传入参数最好是基本数据类型，这样闭包的执行者就不需要导入闭包参数对应的类。  

我建议你再思考是否需要用闭包的时候，把闭包作为你最后的选项，如果你需要广播效果你可以用[多代理的设计模式](#9-一个多代理设计模式)。



### 6. 展示层
展示层，在我的脑子里就是整个MVC里不涉及网络和底层工具的代码层或者说整个业务模块吧。这一层是专门展示数据和用户交互的。  
但一个最常见的不好的习惯是你们有些把网络啊，事件响应啊，页面跳转全部塞到一个VC里面去。我知道这样写很方便，用到什么带入什么，直接在VC里用不就得了嘛，干嘛搞那么麻烦？  
但是你想想啊，你写的代码又不是给你一个人看的，如果你离开这个公司后，后面的人还是要看的啊。就算是你自己写的代码，几个月过后，你需要重构你的代码，你可能都看不懂了，一个又大又肥又难看的VC摆在你面前。。。  
你应该把网络请求，事件响应，页面跳转都交给对应的一个对象或者说helper又或者说是一个manager来处理。VC是就像个领导，他的职能是组织好各个团队，而不是关心那些繁杂的事，这些事都应该交给它的小喽啰去做。  
这样我们把代码分到不同职能的对象上，分门别类，维护起来会轻松好多。  
你可以用协议和代理把处理权交给其他的对象。  


### 7. 网络层
2种方法来设计一个网络请求库:  
1) 使用class或者struct的单例模式，或者它们的static/class方法来发请求 -- 集约结构，每个人都用 要么是从同一个单例里面的方法，或者是静态或类方法。    
2) 一种是把所有方法做成对象方法，谁要用都得先创建一个新的对象, 像苹果的网络库类URLSessinTask -- 他们都是分布式的，建议用这个设计。  

我建议是每个VC有一个特定的NetworkManger的对象来请求网络资源和数据处理。在这个NetworkManger里，会有你的网络库对象。  
那么当VC被销毁的时候，那个NetworkManger也会被销毁，在deinit里，我们可以用网络库对象取消所有的请求。   
```Swift
class NetworkManger: SomeProtocol {
    // 网络库对象
    lazy var networking = NetworkingObject()

    deinit {
        networking.cancelAllRequests()
    }
    // VC调用的代理方法的实现
    func viewController(_ vc: theViewController, shouldLoadUserDataForUserId userId: Int) {
        // check cache...
        // vc这里必需是弱的引用，因为如果vc都死了，就说明这个NetworkManger也已经死了，networking对象也就死了。  
        networking.requestUserData(byUserId userId) {[weak vc] (data) in
            // 检查vc是否为nil
            guard let vc = vc else {return}
            guard let data = data else {return}
            // data transforming....
            // notify vc the data is ready
            vc.reload(data)
        }
    }
}
```  

### 8. 本地持久化
我们这里只简单聊聊数据库，UserDefault,KeyChian和NSKeyedArchiver之类的都只适合存储小数据。  
我个人是比较喜欢“去model化”的（Google，百度一下 如果你感兴趣的话），我只传需要的基本信息。如果另一个模块需要完整的数据，可以自己去数据库取。  
数据库在给数据的时候也是只给一个字典，然后里面装的都是基本数据类型。“去model化”可以给模块更多的独立性，它可以有内部定义的model，但它绝不用外界提供的共享model。因为带着一个 整个项目共享的model，测试起来比较别扭而且移植性很不好。  
很多时候，字典就可以把信息带到下一个模块了。  
当然如果用全局共享模型的话，还是很爽的。但没办法，一切都为了更好的维护性。  
我的项目[AHFM](https://github.com/iOSModularization/AHFM)里，只有在模块里的Manager层（包括了网络和本地数据持久化）里有数据库的模型，而且是只在那里有。不会在模块里的展示层有。因为我们需要反复测试展示层

#### CoreData   
个人感觉CoreData有点过度设计了，或者说太散了。但是它提供的模型化功能，NSFetchedResultsController 都蛮不错的。
还有它的managedObjectContext有点危险，如果你暴露出去的话。最好不要把模型相关的对象暴露出去。数据模型不应该出网络和数据层。  

#### Realm
[AHFM](https://github.com/iOSModularization/AHFM)的第一版就是用Realm的。 
真的很容易用，花那么3-4个小时阅读官方的文档，你就可上了。  
但我在开发的过程中，Realm把我给折磨得。。。   
Realm的每一下编译都至少3、4分钟，有时候5分钟。又因为几乎每一个模块都用到网络库，又要把模块push到私有repo里 也需要编译一次，这简直都快把我弄疯了。AHFM一共有10个主模块啊。    
然后我尝试着把Realm的framework打包进一个单独的模块，这样的话就省了编译的时间。  
但很不幸的是，一旦把Realm的framework包括到单独的pod模块，编译就是不过。你可以看我的[issue](https://github.com/realm/realm-cocoa/issues/5230)。   
然后没办法，自己用SQLite写了一个封装库，还是带模型操作和数据迁移的。感觉还不错。    
[AHDataModel](https://github.com/ivsall2012/AHDataModel)   
NOTE: OC貌似没有这个问题，直接打包成静态库。  

#### SQLite
[AHDataModel](https://github.com/ivsall2012/AHDataModel)， [SQLite.swift](https://github.com/stephencelis/SQLite.swift)和[GRDB.swift](https://github.com/groue/GRDB.swift) 都很不错，拿来直接用了，如果你比较熟悉SQL的话。  
如果你关心哪个数据库快的话，我觉得有点没必要。因为移动应用不像服务器，一般的应用不会达到说因为数据库的速度慢而拖慢整个应用。 如果真是这样，恭喜你，你的公司前途无量啊！  

  

#### 怎样切割数据模型
下面我觉得有必要说一下数据的切割。因为我之前试过，整个项目都用一个模型来操作，各种相关的属性都在里面。但随着项目慢慢变大，模型的属性被各个模块修改，最终存到数据库里的数据很可能就不正确了。  
因为这个模型的职能是不清晰的，其他模块觉得方便就改，然后都往数据库里一扔。数据很容易被污染。而且在读取数据的时候，必要不必要的数据全在一个地方，多余 占内存。  
一般情况下，我们是不把本地添加的属性跟原始数据的属性混在一起的，比如 artistName和playedProgress.  artistName是从服务器来的原始数据，而playedProgress是开发者自己在本地加上去的。  
如果一个模块需要保持数据，那么它就应该有自己的数据模型。  
在上面的artistName和playedProgress例子，artistName可以在一个原始数据模型SongModel里面。而playedProgress可以被开发音频模块的同事创建一个AudioCache的模型。 playedProgress作为播放缓存，用SongId作为id，这样如果查询这个AudioCache对应的是哪个原始数据模型的话，用SongId作为ID去查就可以了。  

示例程序 [AHFMDataCenter](https://github.com/iOSModularization/AHFMDataCenter)  
NOTE: AHFMShow和AHFMEpisode是2个原始数据模型。其他的是本地数据模型。其实每一个本地数据模型应该被做成一个pod，这样哪个模块需要哪个模型，导入就可以了。不要像这个AHFMDataCenter一样，所有模型都在一个pod里，一导入，所有模型都被导入了。   

无论原始还是本地数据模型都不应该被网络和数据层以外的层或模块使用。  
建议每个业务模块都有自己的网络和数据层。一个模块可以由网络/数据层和展示层组成。每个层可以是一个pod。  
展示层问网络/数据层要数据，网络/数据层处理事件和通知展示层数据准备好了。  
了解更多请移步[the Service Router Architecture(SRA)](https://github.com/ivsall2012/A-Highly-Distributed-Architecture-for-Large-scale-Collaborations/blob/master/SRA.md)   


#### 怎样把JSON转成原始模型
在网络封装库里，先用开源库,如SwiftJSON或者MJExtension, 把服务器返回的JSON转成array或者dictionary.  
这样，在“网络/数据层”里，用你的这个网络封装库的时候，直接拿到了基本数据类型的array或者dictionary。  
你还需要提供一个或多个data transformer。 这些transformer的作用是把基本数据类型的数据转成数据库模型。  
之所以我们需要一个transformer，是因为如果服务器改了字段名称，我们只要拿到transformer改就好。外面根本不需要知道字段改名了。
这样就把数据转数据模型的过程相对独立出来了。  

有了数据模型，你可以把它们存进数据库。  
到这里，网络请求的数据就已经到达本地的数据库了。下一步是把它们读取出来。  
如果你用的是[AHDataModel](https://github.com/ivsall2012/AHDataModel)，那么数据出来后就是数据模型了。  
现在这些数据模型还是在“网络/数据层”里。下一步是把它们给“去model”化 -- 只把必要信息的基本数据类型用dictinary或array传递到展示层。  
如果你用的数据封装库是[AHDataModel](https://github.com/ivsall2012/AHDataModel)，那么它的协议早就要求提供 模型->dict的方法了。  
如果必要，要把这些从模型到dict的数据和现有的本地数据模型(如 下载进度，播放进度等)merge成一个dict。这个工作也是在“网络/数据层”里做的。  
展示层只知道要什么数据和什么时候要。作为“网络/数据层”，它应该把所有数据都按照展示层要求的格式把数据传递到展示层里。  

示例:      
**data transformer**: [AHFMDataTransformers](https://github.com/iOSModularization/AHFMDataTransformers)  
**一个网络/数据层**: [AHFMShowPageManager](https://github.com/iOSModularization/AHFMShowPageManger)  
配套的展示层[AHFMShowPage](https://github.com/iOSModularization/AHFMShowPage)  
**另一个网络/数据层**: [AHFMAudioPlayerVCManager](https://github.com/iOSModularization/AHFMAudioPlayerVCManager)  
配套的展示层[AHFMAudioPlayerVC](https://github.com/iOSModularization/AHFMAudioPlayerVC)  


### 9. 一个多代理设计模式
Q: 什么是多代理系统?  
A: 多代理设计模式，顾名思义 就是一个对象管理多个代理的设计模式。我们日常用的代理都是一对一个代理模式， 比如一个VC需要一个代理通知外界做事情。多代理设计模式是支持一对多的，把事件广播出去。  

Q: 那么什么时候用多代理设计模式?  
A: 当你想用多个代理来替换Notification的时候。多代理设计模式与Notification的唯一区别就是，在Notification模式中，你只需要知道Notification的名字就可以通过NotificationCenter来监听事件。  
但多代理设计模式需要导入协议和拥有代理的那个对象，如 XXX.delegate = self 或者 XXX.addDelegate(self), 这里你需要导入XXX对应的类或库。  

在设计过程中的2个困难:  
* 怎么样让所有加进来的代理保持**弱引用**   
无论在Swift里还是OC里，Array或者Dictionary总是会对它里面的对象持有一个强引用。多代理模式也是代理模式的一种，绝对不允许有强引用的代理对象。  
这里我们可以用一个代理容器来弱引用那些代理对象。 这样在这个模式里，只对容器产生强引用，而代理对象是藏在容器里的一个弱引用 -- 代理对象死了也没关系了。  

* 怎么样在线程安全的情况下移除代理？
我们只需要在添加代理的时候，在所有操作放到一个自定义的serial队列 里面去，然后移除已经被销毁的代理对象的代理容器。  

以下是Swift的实现，如果是OC的话，你可以用一个class作为容器。  
Example codes:  
```Swift
public protocol MonitorDelegate: NSObjectProtocol{
    // 代理方法
}

internal struct DelegateContainer {
    var delegate: MonitorDelegate?
}

public class Monitor: NSObject {
    // 自定义serial队列
    fileprivate static var queue = DispatchQueue(label: "MonitorQueue")
    // 用字典来存储代理容器
    fileprivate static var delegateDict = [String: DelegateContainer]()
    
    // 这个Monitor可以是一个单例
    public let shared = Monitor()
    
    public static func addDelegate(_ delegate: MonitorDelegate) {
        // 为了线程安全，我们把所有操作都丢到自定义的队列里。  
        self.queue.async {
            /// 用来存储需要移除的代理容器。 我们用UUID作为代理容器的key。  
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
            /// 之所以我们不用Array,是因为每移除一个代理容器，每个元素的index都会变。所以干脆就用字典+UUID了。  
            let uuid = UUID().uuidString
            let container = DelegateContainer(delegate: delegate)
            self.delegateDict[uuid] = container
        }
    }
    // 模拟一个下载器的一个接收数据的方法
    func didReceiveData(_ data: Data, size: UInt) {
        for container in self.delegateDict.values {
            container.delegate?.monitor(self, didReceiveData:data, size: size)
        }
    }
}
```

一个下载器案例就是用了这个多代理模式来给外界监听下载进度的:  
[AHDownloader](https://github.com/ivsall2012/AHDownloader)


## Conclusion
Introducing [the Service Router Architecture(SRA)]()