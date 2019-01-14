# NSTableView怎么用Section分组

英语原文: [链接](http://blog.krzyzanowskim.com/2015/05/29/lets-talk-about-sections-for-nstableview/)

  今天我们来讲讲在OS X里AppKit的表格视图。OS X编程历史由来已久。iOS程序猿并不是总了解这段历史，然而它却是UIKit之母。

  > An NSTableView object displays data for a set of related records, with rows representing individual records and columns representing the attributes of those records.

  NSTableView是一个将数据显示在有行有列的表格上的轻量级视图控件，尽管它有一个大兄弟：

  > NSOutlineView is a subclass of NSTableView that uses a row-and-column format to display hierarchical data that can be expanded and collapsed, such as directories and files in a file system.

  NSOutlineView存在一个问题就是它真的很巨型，需要做很多配置工作。它对于表格视图来说是一把瑞士军刀，但是对于简单的表格来说就真的是小题大做了。但从另一方面来说，简单的表格视图又过于简陋。NSTableView允许行列的显示。就这样。
![view](http://blog.krzyzanowskim.com/content/images/2015/05/plain.png)

  如果你熟悉UIKit框架，你一定知道UITableView看起来和NSTableView一样。但不是的。

# Sections
  一个却是的但是我需要的功能是Section。NSTableView并不支持任何简单方式去分组显示数据。那什么是section？

  Section是row单元分组的展现在一个列里，有带标题（或者不带标题）如下：
![view](http://blog.krzyzanowskim.com/content/images/2015/05/section.png)

这是“PEOPLE”的一组，有“Frank”，“Monica”， “Natalie”， “Alice”四个内容项。

下面我将讨论如何添加分组到原色NSTableView的一种可能。实现它并不是一件很难的事，只是要劳作一番。（故弄玄虚一下）

# 数据源
  首先，我创建了新的协议，将默认的数据源协议扩展为支持sections，有三个方法需要去计算分组。

  我起了个名字 NSTableViewSectionDataSource：

  {% codeblock lang:swift %}
  protocol NSTableViewSectionDataSource: NSTableViewDataSource {  
    func numberOfSectionsInTableView(tableView: NSTableView) -> Int
    func tableView(tableView: NSTableView, numberOfRowsInSection section: Int) -> Int
    func tableView(tableView: NSTableView, sectionForRow row: Int) -> (section: Int, row: Int)
  }
  {% endcodeblock %}

# SectionForRow
    tableView(tableView: NSTableView, sectionForRow row: Int) 的实现是负责把无分组的row表现未分组的结构（row的分组）。
    {% codeblock lang:swift %}
    func tableView(tableView: NSTableView, sectionForRow row: Int) -> (section: Int, row: Int) { ... }
    {% endcodeblock %}
    
    针对指定输入row的索引，它返回分组里沿着row索引排下来的的section索引。
    ![view](http://blog.krzyzanowskim.com/content/images/2015/05/sections.gif)
    
    给定了4个分组，纯表格里第5行就是分组3里的第1行，索引值是（section：2， row： 0）

# 难点

  难点是计算但是你不想计算。好吧，我来。我的方法是通过sectionForRow(row, counts)里的row索引，和每一个分组里row的总和，以及提供分组号：

  {% codeblock lang:swift %}
  private func sectionForRow(row: Int, counts: [Int]) -> (section: Int?, row: Int?) {  
    let total = reduce(counts, 0, +)

    var c = counts[0]
    for section in 0..<counts.count {
        if (section > 0) {
            c = c + counts[section]
        }
        if (row >= c - counts[section]) && row < c {
            return (section: section, row: row - (c - counts[section]))
        }
    }
    
    return (section: nil, row: nil)
  }
  {% endcodeblock %}

  那个counts是可用分组的数量。一般这样用：
  {% codeblock lang:swift %}
  let (section, sectionRow) = sectionForRow(row, counts: [2,3,5]) 
  {% endcodeblock %}

  我会用这个方法去实现剩下的协议里定义的方法：NSTableViewDelegate, NSTableViewDataSource 和 NSTableViewSectionDataSource.

  NSTableViewSectionDataSource协议里最重要的方法和使用最普遍的是(tableView: NSTableView, sectionForRow row: Int) -> (section: Int, row: Int). 以下是我的实现方式：

  {% codeblock lang:swift %}
  func tableView(tableView: NSTableView, sectionForRow row: Int) -> (section: Int, row: Int) {  
    if let dataSource = tableView.dataSource() as? NSTableViewSectionDataSource {
        let numberOfSections = dataSource.numberOfSectionsInTableView(tableView)
        var counts = [Int](count: numberOfSections, repeatedValue: 0)

        for section in 0..<numberOfSections {
            counts[section] = dataSource.tableView(tableView, numberOfRowsInSection: section)
        }
    
        let result = self.sectionForRow(row, counts: counts)
        return (section: result.section ?? 0, row: result.row ?? 0)
    }
    
    assertionFailure("Invalid datasource")
    return (section: 0, row: 0)
  }
  {% endcodeblock %}

  # 配置
  配置是通过NSTableViewSectionDataSource协议制作的。它是基于众所周知的UIKit里的UITableDataSource协议。那里有代理方法去获得分组号码和指定项。

  # 视图
  视图通过我的NSTableView实例的代理（NSTableViewDete）来处理。下面你可以到它能如此轻易的处理问题了：
  {% codeblock lang:swift %}
  func tableView(tableView: NSTableView, viewForTableColumn tableColumn: NSTableColumn?, row: Int) -> NSView? {  
    let cellView = tableView.makeViewWithIdentifier("CellView", owner: self) as! NSTableCellView

    if let dataSource = tableView.dataSource() as? NSTableViewSectionDataSource {
        let (section, sectionRow) = dataSource.tableView(tableView, sectionForRow: row)
    
        // here! build view for given section and row index
    }
    
    return cellView
  }
  {% endcodeblock %}

  当然，我在这个例子里未提及分组的标题。这不是很难实现。我需要：
  1. 调节每个分组里列的数量，每个加一
  2. 在每个分组里每个索引为0的行返回特定的头视图

  就这样。现在可以构建全能的，可以组织的分组表格视图，做了一个在[这里](http://blog.krzyzanowskim.com/2015/05/29/lets-talk-about-sections-for-nstableview/)

  结果

  结果如下：

  ![view](http://blog.krzyzanowskim.com/content/images/2015/05/NSTableView-Sections.gif)

  看起来只是个列表，但是内置有两个分组显示了。