# 弃用表格视图的可行性

英文原文链接:[The Case for Deprecating UITableView](https://pspdfkit.com/blog/2017/the-case-for-deprecating-uitableview/)
UITableView是传统iOS开发的基石也是最古老的类。自从iPhone OS 2.0开始，在大量的iOS app就有着极其广泛的应用。那么为什么我们提出弃用这个使用率最高的类呢？

简而言之，因为有UICollectionView，这个在iOS 6（2012）新加入的，像一个UITableView的超集合的类，能够做更多的事情。

细想一下我们的免费应用 PDF Viewer里的UI里使用的也是一个类似table view布局的UICollectionView。
![view](https://pspdfkit.com/images/blog/2017/the-case-for-deprecating-uitableview/toggle-list-grid-view-d8bb4e20.gif)

我们并不是第一个号召让UITableView退出江湖的人，Apple也明白这一点。只要稍微看一下iOS10框架里的一些新类。（h/t [@BunuelCuboSoto's tweet](https://twitter.com/BunuelCuboSoto/status/818169193073934337)）

* [UICollectionViewTableLayout.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableLayout.h)
* [UICollectionViewTableLayoutAttributes.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableLayoutAttributes.h)
* [UICollectionViewTableLayoutInvalidationContext.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableLayoutInvalidationContext.h)
* [UICollectionViewTableLayoutSwipeAction.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableLayoutSwipeAction.h)
* [UICollectionViewTableLayoutSwipeActionButton.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableLayoutSwipeActionButton.h)
* [UICollectionViewTableLayoutSwipeActionPullView.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableLayoutSwipeActionPullView.h)
* [UICollectionViewTableCell.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableCell.h)
* [UICollectionViewTableHeaderFooterView.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableHeaderFooterView.h)
* [UICollectionViewTableSeparatorView.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/Frameworks/UIKit.framework/UICollectionViewTableSeparatorView.h)
* [UICollectionViewDelegateTableLayout.h](https://github.com/JaviSoto/iOS10-Runtime-Headers/blob/master/protocols/UICollectionViewDelegateTableLayout.h)

API 同时有公有部分和私有（以_开头的方法），看起来UIKit团队曾提供这样的API但是还没有达到发出来的标准。当你更仔细的看看时，会发现UICollectionViewTableCell的内部还是在使用UITableViewCell为了使用像扫滑删除这些功能，所以像这些过往产生的快捷方式是这些新API没有放在公共接口的原因之一。我们希望他们能够在iOS 11的版本中做成改变。请复制[rdar://30098111](http://openradar.appspot.com/30098111)来为这个功能投票。甚至你可以[自己撰写并提交bug报告那更好](https://pspdfkit.com/blog/2016/writing-good-bug-reports/)。

部分框架已经开始转变了。思考一下UISearchDisplayController是怎么被UISearchController这个合乎情理的API在iOS 8中替代了。新的API是界面不可知论者，不再直接绑定到表格视图，所以它能够用在集合视图或者一些自定义类里。（如果你还有历史遗留代码，可以参考[这篇很好的文章](http://useyourloaf.com/blog/updating-to-the-ios-8-search-controller)，它可以帮你理解这两个API的区别并且指导你怎么升级你的代码。）

UICollectionView很灵活，API设计也很好。追溯到WWDC2012那年，当Apple公布iOS 6预览版本时，我们重写了它并花费了时间去实现一个全功能的子集（除了动画外），大家应该还记得这个类[PSTCollectionView](https://github.com/steipete/PSTCollectionView)。它帮助开发者更快速的去实现集合视图，同样的它也支持iOS 4.3的版本，同时也能很方便的通过一些[疯狂的运行时技巧](https://github.com/steipete/PSTCollectionView/blob/master/PSTCollectionView/PSTCollectionView.m#L2210-L2276)切换到“真实”类。

# 历史

Nitin Ganatra 分享了一首不错的 [episode of the Debug podcast](http://www.imore.com/debug-41-nitin-ganatra-episode-iii-iphone-ipad),关于UITable， UIScroller 和从iPhone到iPad转化历程。

简单的来说，iPhone OS 1.0处于混乱的状态，当有框架共享出来时，每一个团队都定制出自己一套的框架。最终，一个表格类独立成UITable常驻在iOS框架里，一直到iOS 6的版本才被移除。可以在[运行时头文件的历史](https://github.com/nst/iOS-Runtime-Headers/commits/5766c608eca76222e2d34fe6d37aa1cf91a2246b/Frameworks/UIKit.framework/UITable.h)中看出来。

iPhone OS 2 做了很大的整理，UITableView就随之而来了，成为了每一个iOS版本升级都会绑在一起更新的类，特别是在iOS 7版本中做了颠覆性的新设计。

# API的缺陷
UITableVIew非常的古老，大部分的设计和实现都在blocks和自动布局这些特性出现之前。这表明。这是你在UITableView中如何更新一个section：

{% codeblock lang:swift %}
tableView.beginUpdates()
tableView.reloadSections(IndexSet(integer: sectionIndex), with: UITableViewRowAnimation.none)
tableView.endUpdates()
{% endcodeblock %}


同样在UICollectionView
{% codeblock lang:swift %}
collectionView.performBatchUpdates({
    collectionView.reloadSections(IndexSet(integer: sectionIndex))
}, completion: nil)
{% endcodeblock %}

这看起来可能不太像，但是很容易被 beginUpdates和endUpdates这样的配对写法搞昏，然后这不可能出现在UICollectionView的API里。因为这个类实在是很老了，它有很多弃用的方法还要继续支持。在Swift的范围里已经看不到很多弃用很久的API，但是当你切换到Objctive-C时，还是能看到Cell的所用扩展方法：

{% codeblock lang:swift %}
@interface UITableViewCell (UIDeprecated)

// Frame is ignored.  The size will be specified by the table view width and row height.
- (id)initWithFrame:(CGRect)frame reuseIdentifier:(nullable NSString *)reuseIdentifier NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;

// Content properties.  These properties were deprecated in iPhone OS 3.0.  The textLabel and imageView properties above should be used instead.
// For selected attributes, set the highlighted attributes on the textLabel and imageView.
@property (nonatomic, copy, nullable)   NSString *text NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;                        // default is nil
@property (nonatomic, strong, nullable) UIFont   *font NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;                        // default is nil (Use default font)
@property (nonatomic) NSTextAlignment   textAlignment NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;               // default is UITextAlignmentLeft
@property (nonatomic) NSLineBreakMode   lineBreakMode NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;               // default is UILineBreakModeTailTruncation
@property (nonatomic, strong, nullable) UIColor  *textColor NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;                   // default is nil (text draws black)
@property (nonatomic, strong, nullable) UIColor  *selectedTextColor NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;           // default is nil (text draws white)

@property (nonatomic, strong, nullable) UIImage  *image NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;                       // default is nil. appears on left next to title.
@property (nonatomic, strong, nullable) UIImage  *selectedImage NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;               // default is nil

// Use the new editingAccessoryType and editingAccessoryView instead
@property (nonatomic) BOOL              hidesAccessoryWhenEditing NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;   // default is YES

// Use the table view data source method -tableView:commitEditingStyle:forRowAtIndexPath: or the table view delegate method -tableView:accessoryButtonTappedForRowWithIndexPath: instead
@property (nonatomic, assign, nullable) id        target NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;                      // target for insert/delete/accessory clicks. default is nil (i.e. go up responder chain). weak reference
@property (nonatomic, nullable) SEL               editAction NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;                  // action to call on insert/delete call. set by UITableView
@property (nonatomic, nullable) SEL               accessoryAction NS_DEPRECATED_IOS(2_0, 3_0) __TVOS_PROHIBITED;             // action to call on accessory view clicked. set by UITableView

@end
{% endcodeblock %}

# 自动布局

自动布局相对于UITableView来说很新。难怪它们合不来。Apple在表格视图单元的自动调整方面做得很不错，首次的迭代它们很愉快的合作了。然而还是会出现奇怪的UI问题或者出来啊的效果和想象的有些偏差。

其中一个例子就是你不能用默认的表格视图单元来支持多行文本显示的文本框。在使用自动布局之前，这个操作是不可能行得通的，比如说，设置textLabel的numberOfLines为0 去支持多行文本标签，只因为你无论什么方法都计算不出单元格的高度，正如你并不知道这个单元格如何摆放这些标签。现在伴随着自动布局和可自动调整的表格视图单元格的到来，Apple将这些都自动化了。现在只需要试图去设置numberofLines为0，就可以让表格视图帮你去计算出给定文本时的单元格高度。这方法在大部分情况下能正常工作，然而你终究会遇上当单元格里使用超出textLabel和detailTextLabel这两个控件时控件不受约束的情形。询问过Apple的工程师为什么会不像WWDC2016大会上所说的那样好用时，我们得到了一个简单的答案，[UITableView](https://developer.apple.com/reference/uikit/uitableviewcell)比自动布局出现得更早，它的设计初衷是很简单的并不适用在这样的使用方法上。事实上他告诉我们不应该这样直接的去操作它，除了标签的text属性和图片视图的image属性。这意味着你需要子类化[UITableViewCell](https://developer.apple.com/reference/uikit/uitableviewcell)来定制你自己想要的标签。

另外一种很容易出现的问题，在表格视图单元格使用自动布局时你会突然看到像UIViewAlertForUnsatisfiableConstraints这些信息提示出现在控制台里。如果你没有小心处理你的布局，你将会在高清屏设备上得到一个有0.5pt偏差的单元格高度，事实上那是一个分割器的高度。这样的情况通常出现在你使用自动布局的时候使用了单元格高度自动计算。表格视图单元格会试图为一两个分割器创造空间（一个还是连个分割器那就要看单元格的索引路径和表格视图的样式了）。如果你对设计布局有很严格的要求，你不要用自动调节来处理这些不同的高度，单元格不会满足你所有的约束，最终你将得不到想要的视图效果。

# 表格视图单元格什么都干

通过以上的讨论，应该比较清楚[UITableViewCell](https://developer.apple.com/reference/uikit/uitableviewcell)已经处理了很多事情，然而，还有更多的事情等着它。

一个表格视图单元格从一个样式开始初始化。看看它做了些什么：
{% codeblock lang:swift %}
public enum UITableViewCellStyle : Int {
    case `default` // Simple cell with text label and optional image view (behavior of UITableViewCell in iPhoneOS 2.x)
    case value1 // Left aligned label on left and right aligned label on right with blue text (Used in Settings)
    case value2 // Right aligned label on left with blue text and left aligned label on right (Used in Phone/Contacts)
    case subtitle // Left aligned label on top and left aligned label on bottom with gray text (Used in iPod).
}
{% endcodeblock %}

通过样式去设置好的标签和图片视图是符合全有或全无原则。比如说，你使用了一个文本框的表格视图单元格([UITalbeViewCell](https://developer.apple.com/reference/uikit/uitableviewcell))，但是又自行添加了第二个标签，你大部分的情况下都会出现布局的问题。所以你应该在子类化一个表格视图单元格的时候把默认的API也带进去，尽管你不需要用到它们。所以当你看看你的API时，你会发现单元格内建的标签和你自己创建的一模一样。你看到了哪个？这很让人困惑。

自出现在iOS 6版本以来，UITableViewCellStyle和registerClass:forCellReuseIdentifier:也是合作得不是那么好。如果你用刚刚那个方法注册了一个标准的[UITableViewCell](https://developer.apple.com/reference/uikit/uitableviewcell)类，表格视图就会总是会以默认的方式初始化。你可以转变一下思路用创建子类来覆盖掉initWithStyle:reuseIdentifier:方法来忽略掉默认样式并使用你传入的样式。但这是一种曲线救国的方法，而且看起来很奇怪。

如果我用默认样式设置副标题会怎样？事实上那会涉及很多旧逻辑，如果你要使用[UITableViewCell](https://developer.apple.com/reference/uikit/uitableviewcell)，最好的方法就是添加一个自己的view到contentView，那你就不需要和价值10年的自定义逻辑代码打交道了。

# Display Sizes and Size Classes
UITableView 是从单屏幕单类设备iPhone时代开始应用的。但是时过境迁，沧海桑田。如果你建立一个多设备应用时，你会想适配所有的支持机型，有可能你的表格视图并不能适用在像iPad这样的大屏幕设备上。

列表控件只包含了一小部分的文本，于是在iPhone阅读起来很爽但是在iPad上就空缺。你可以看到一堆的文字和一大片的空白处，相当浪费空间。虽然你可能不应该试图在大屏幕设备上得到和小屏幕设备上相等信息密度，你至少还能比iPhone的显示屏展示更多的信息在大屏幕设备上。所以可以在iPad上展示两个、三个甚至更多列。

这正是UICollectionView要做的。所以，与其在iPad上用collection View和在iPhone上用UITableView，还不如都用collection view，只要在小屏幕的设备上设置成单列，大屏幕的时候设置成多列就好了。

这也正是Apple在iTunes Connect 这个app上所做的思路，他们在[WWDC’14 Session 232 - Advanced User Interfaces with Collection Views](https://developer.apple.com/videos/play/wwdc2014/232/) 做了关于这个思路的演讲。他们做了我们打算做的事情：抛弃UITableView，用UICollectionViewLayout的一个类似Table View样子的子类代替了它。他们做的最大的好事是：他们把这个重要的部分迁移到了示例代码里，所以这是一个好的开始，希望大家也开始把自己的表格视图换成集合视图。

数据代理

正如之前讨论过，还有很多场景你希望表格视图能在水平时有一个compact型的尺寸,而在regular型的尺寸下呈现多列的集合视图效果。如果你用现行的API来做这件事的话，你需要有两套data source和data delegate的实现，[UITableView](https://developer.apple.com/reference/uikit/uitableview) 和 [UICollectionView](https://developer.apple.com/reference/uikit/uicollectionview)各一套。

一种曲径是写一个适配器类来配合一种或者更多种的情况，但是如果有一个系统级别的类似UICollectionView又能适配成表格视图的布局，那就会简单很多。你不再需要复制粘贴你那些适配器类的代码来让你的app看起来更贴心。

瞧瞧Android那边
Andoird那边也是类似的情况。自从第一个版本面世后，Android的框架包含了一个类似UITableView那样展示项目的ListView。伴随着Android 5.0 Lollipop在2014的发布，Goole公布了一个继任者，RecyclerView。 RecyclerView被封装在Android Support Library里，说明了它可以被所有版本的Android使用，包括2010年发布的Android2.1。RecyclerView同时代替了其他基础视图，像GridView和其他的布局视图。（像staggered grid等等）。
Google明确了逐渐抛弃ListVIew，可以看看文档中的第一句话
{% codeblock lang:swift %}
The RecyclerView widget is a more advanced and flexible version of ListView.
{% endcodeblock %}
因为一些历史遗留的功能没有默认的实现，都需要开发者写一些模版类来迁徙他们旧的代码，总的来说，RecyclerView提供了更好的性能和灵活性，是非常值得一试的。

# 总结
不需要很急着去采用集合视图去代替表格视图，但是Apple该给出更好的API接口让集合视图用起来像表格视图，这会让我们制作更简单易用的多屏幕尺寸的应用。