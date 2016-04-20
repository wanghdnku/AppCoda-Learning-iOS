#Swift学习笔记
=============
----------------------
##iOS9不能连接HTTP
在`info.plist`中添加

```
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

##隐藏键盘
使输入框失去焦点，即可隐藏键盘：

```swift
override func touchesEnded(touches: NSSet!, withEvent event: UIEvent!) {
    xxxx.resignFirstResponder()
}
```


##在swift种设置弹出框
```swift
@IBAction func showMessage() {
    // 定义一个alertController，设定了title，message和style
    let alertController = UIAlertController(title: "Welcome to My First App", message: "Hello World", preferredStyle: UIAlertControllerStyle.Alert)
    // 向alertController添加了一个action，显示OK
    alertController.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: nil))
    // 设置显示alertController
    self.presentViewController(alertController, animated: true, completion: nil)
}
```


-----
##查看App在各个屏幕尺寸下的表现
>按住option点`assistant pop-up menu`的`preview`选项

-----
##TableViewController
要继承`UITableViewDataSource`和`UITableViewDelegate`协议
>control拖拽TableView将其与DataSource和Delegate两个Outlets建立联系

###UITableViewDataSource
实现两个方法：
**tableView(_:numberOfRowsInSection:)**

```swift
// 说明tableView具体有几行
func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return restaurantNames.count
}
```

**tableView(_:cellForRowAtIndexPath:)**

```swift
// 说明每一个下标对应的Cell显示的具体内容
func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    // 设置Cell的identifier
    let cellIdentifier = "Cell"
    // 声明Cells可以复用，节省内存
    let cell = tableView.dequeueReusableCellWithIdentifier(cellIdentifier, forIndexPath: indexPath)
    cell.textLabel?.text = restaurantNames[indexPath.row]
    // restaurant是图片名字，放在Assets.xcassets里面
    cell.imageView?.image = UIImage(named: "restaurant")
    // 下面几行决定一个Cell是否被做过了标记
    if restaurantIsVisited[indexPath.row] {
        cell.accessoryType = .Checkmark
    } else {
        cell.accessoryType = .None
    }
    // 清除Cell的背景色
    cell.backgroundColor = UIColor.clearColor()
    return cell
}
```
如果自己设定了UITableViewCell，则需要在第6行进行强制转换

```swift
let cell = tableView.dequeueReusableCellWithIdentifier(cellIdentifier, forIndexPath: indexPath) as! RestaurantTableViewCell
// 这样就能使用自定义的UITableViewCell里定义的Outlet了
cell.nameLabel.text = restaurantNames[indexPath.row]cell.thumbnailImageView.image = UIImage(named: restaurantImages[indexPath.row])


```

####如果直接用TableViewController而不是ViewController
要override`cellForRowAtIndexPath`和`numberOfRowsInSection`，还要重写一个函数`numberOfSectionsInTableView`

```swift
override func numberOfSectionsInTableView(tableView: UITableView) -> Int {    return 1}
```
这样可以设定多个Section，如果删除此函数的话，默认返回值是1。

####将Cell中的图片设定成圆形
在`tableView(_:cellForRowAtIndexPath:)`中加上代码：

```swift
// 设置圆形图标
cell.thumbnailImageView.layer.cornerRadius = 30.0
cell.thumbnailImageView.clipsToBounds = true
```

###UITableViewDelegate
`tableView(_:didSelectRowAtIndexPath:)`函数决定了被选中的Cell所执行的操作

```swift
override func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
    // 设置一个弹出菜单optionMenu。ActionSheet选项时，菜单从下方滑出；为Alert是，从屏幕中间弹出
    let optionMenu = UIAlertController(title: nil, message: "What do you want to do?", preferredStyle: .ActionSheet)
    // 设置一个Cancel按钮
    let cancelAction = UIAlertAction(title: "Cancel", style: .Cancel, handler: nil)
    // 定义一个handler
    let callActionHandler = { (action:UIAlertAction!) -> Void in
    // 定义一个Alert信息，Style为弹出式Alert
    let alertMessage = UIAlertController(title: "Service Unavailable", message: "Sorry, the call feature is not available yet. Please retry later.", preferredStyle: .Alert)
    // 向Alert弹框里面增加一个选项
    alertMessage.addAction(UIAlertAction(title: "OK", style: .Default, handler: nil))
    // 显示弹框
        self.presentViewController(alertMessage, animated: true, completion: nil)
    }
    // 定义一个beenHere的选项，并用闭包的方式定义Handler，与上面的Alert弹窗不同
    let isVisitedAction = UIAlertAction(title: "I've \((self.restaurantIsVisited[indexPath.row]) ? "not " : "") been here", style: .Default, handler: {
        (action:UIAlertAction!) -> Void in
            let cell = tableView.cellForRowAtIndexPath(indexPath)
            // 将已经打勾的做标记
            self.restaurantIsVisited[indexPath.row] = !self.restaurantIsVisited[indexPath.row]
            // 打勾 / 取消对勾
            cell?.accessoryType = (self.restaurantIsVisited[indexPath.row]) ? .Checkmark : .None
    })
        
    // 设置一个CallAction，拨打电话
    let callAction = UIAlertAction(title: "Call 123-000-\(indexPath.row)", style: .Default, handler: callActionHandler)
    // 将cancelAction添加到弹出菜单中
    optionMenu.addAction(cancelAction)
    optionMenu.addAction(isVisitedAction)
    optionMenu.addAction(callAction)
    // 设置显示optionMenu
    self.presentViewController(optionMenu, animated: true, completion: nil)
        
    // 执行完操作之后，释放该行的选择状态
    tableView.deselectRowAtIndexPath(indexPath, animated: false)
}
```

`tableView(_:commitEditingStyle:forRowAtIndexPath:)`使用默认滑动选项

```swift
override func tableView(tableView: UITableView, commitEditingStyle
    editingStyle: UITableViewCellEditingStyle, forRowAtIndexPath indexPath: NSIndexPath) {
    // 当滑动选项为删除选项的时候
    if editingStyle == .Delete {
        // 从Data Source中删除该行的数据
        restaurantNames.removeAtIndex(indexPath.row)
        restaurantLocations.removeAtIndex(indexPath.row)
        restaurantTypes.removeAtIndex(indexPath.row)
        restaurantIsVisited.removeAtIndex(indexPath.row)
        restaurantImages.removeAtIndex(indexPath.row)
        // 从Data Source删除数据之后，还要更新列表，不然将不会有删除效果（事实上数据已经删除，但View没有更新）
        // tableView.reloadData() // 这样也可以，但是效果不自然
        tableView.deleteRowsAtIndexPaths([indexPath], withRowAnimation: .Fade)
    }
}
```

当然，也可以用`tableView(_:editActionsForRowAtIndexPath:)`定制滑动选项

```swift
override func tableView(tableView: UITableView, editActionsForRowAtIndexPath
    indexPath: NSIndexPath) -> [UITableViewRowAction]? {
    // 定制分享按钮
    let shareAction = UITableViewRowAction(style: UITableViewRowActionStyle.Default, title: "Share", handler: 
    {   // 这是一个闭包，定义一个handler
        (action, indexPath) -> Void in
            let defaultText = "Just checking in at " + self.restaurantNames[indexPath.row]
            if let imageToShare = UIImage(named: self.restaurantImages[indexPath.row]) {
                let activityController = UIActivityViewController(activityItems: [defaultText, imageToShare], applicationActivities: nil)
                self.presentViewController(activityController, animated: true, completion: nil)
            }
    })
    shareAction.backgroundColor = UIColor(red: 28.0/255.0, green: 165.0/255.0, blue: 253.0/255.0, alpha: 1.0)
    // 定制删除按钮
    let deleteAction = UITableViewRowAction(style: UITableViewRowActionStyle.Default, title: "Delete",handler: 
    {   // 这是一个闭包，定义一个handler
        (action, indexPath) -> Void in
            // Delete the row from the data source
            self.restaurantNames.removeAtIndex(indexPath.row)
            self.restaurantLocations.removeAtIndex(indexPath.row)
            self.restaurantTypes.removeAtIndex(indexPath.row)
            self.restaurantIsVisited.removeAtIndex(indexPath.row)
            self.restaurantImages.removeAtIndex(indexPath.row)
            self.tableView.deleteRowsAtIndexPaths([indexPath], withRowAnimation: .Fade)
    })
    deleteAction.backgroundColor = UIColor(red: 202.0/255.0, green: 202.0/255.0, blue: 203.0/255.0, alpha: 1.0)
    return [deleteAction, shareAction]
}
```

###UITableView的个性化定制
先要在UIViewController里面添加tableView的Outlet，然后才能设定。在ViewDidLoad中

```swift
tableView.backgroundColor = UIColor(red: 240.0/255.0, green: 240.0/255.0, blue: 240.0/255.0, alpha: 0.2)
```
要清除没有数据的行的横线，在ViewDidLoad里添加

```swift
tableView.tableFooterView = UIView(frame: CGRectZero)
```
改变行间分隔线的颜色

```swift
tableView.separatorColor = UIColor(red: 240.0/255.0, green: 240.0/255.0, blue: 240.0/255.0, alpha: 0.8)
```
###设定行间距自适应的方法
1.在ViewDidLoad里面添加：

```swift
tableView.estimatedRowHeight = 36.0tableView.rowHeight = UITableViewAutomaticDimension
```
2.设置可换行文字的Lines为0；
3.设定Label的Constrains，使得可以推断出与上下两边的间距。



##Navigation Controller

###个性化的Navigation Bar
修改Navigation Bar需要在`AppDelegate.swift`中的`application(_:didFinishLaunchingWithOptions:)`方法中添加：

```swift
// 修改Navigation Bar的颜色
UINavigationBar.appearance().barTintColor = UIColor(red: 242.0/255.0, green: 116.0/255.0, blue: 119.0/255.0, alpha: 1.0)
// 修改字体颜色
UINavigationBar.appearance().tintColor = UIColor.whiteColor()
// 修改字体
if let barFont = UIFont(name: "Avenir-Light", size: 24.0) {
    UINavigationBar.appearance().titleTextAttributes = [NSForegroundColorAttributeName:UIColor.whiteColor(), NSFontAttributeName:barFont]
}
```

###当滑动的时候隐藏Navigation Bar

```swift
navigationController?.hideBarOnSwipe = true
// 另一个方法可以使Navigation Bar强行不消失
navigationController?.setnavigationBarHidden(false, animated: true)
```

###修改Navigation Bar的标题
在所在Controller的ViewDidLoad里加入

```swift
title = “所要显示的标题”
```
###取消Navigation Bar返回按钮上的文字
在上一个ViewController里面的ViewDidLoad里加入

```swift
navigationItem.backBarButtonItem = UIBarButtonItem(title: "", style: .Plain, target: nil, action: nil)
```

###通过Segue在Views之间传递信息

```swift
override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    if segue.identifier == "showRestaurantDetail" {
        if let indexPath = tableView.indexPathForSelectedRow {
            let destinationController = segue.destinationViewController as! RestaurantDetailViewController
            destinationController.restaurant = restaurants[indexPath.row]
        }
    }
}
```

##VIewDidAppear和ViewWillAppear

```swift
override func viewWillAppear(animated: Bool) {
    super.viewWillAppear(animated)
    // Code Here
}
```




##状态栏（Status Bar）
###隐藏状态栏
```swift
override func prefersStatusBarHidden() -> Bool {     return true}
```

###改变状态栏颜色
```swift
override func preferredStatusBarStyle() -> UIStatusBarStyle {
    return .LightContent
}
// 不过好像不起作用
```
另一种方法是在`info`里面增加Key：`View controller-based status bar appearance`并设置为`NO`，然后在AppDelegate.swift中的`application(_:didFinishLaunchingWithOptions:)`添加

```swift
UIApplication.sharedApplication().statusBarStyle = .LightContent
```

##UIImageView
直接从URL来获取image

```swift
if let url = NSURL(string: "https://api.mapbox.com/styles/v1/haidongw/cilxc3u6h005v9mm3udbspca1/static/127.963238,-26.48076,2.72,0.00,0.00/600x400?access_token=pk.eyJ1IjoiaGFpZG9uZ3ciLCJhIjoiY2lseDEyNnF5MDVxaHVka3M4eHI3bWVnaiJ9.3zSqNekaroRp0zGsCCCfYQ") {
    if let data = NSData(contentsOfURL: url) {
        imageView.image = UIImage(data: data)
    }
}
```
<br />

---
#地图上的Button被覆盖
在ViewDidLoad里面添加`mapView.bringSubviewToFront(button)`

<br />

---
#设定弹出框样式

```swift
// 设置Alert信息
let alertMessage = UIAlertController(title: "Alert Title", message: “Alert Message”, preferredStyle: .Alert)

// 向Alert信息添加选项
alertMessage.addAction(UIAlertAction(title: "DETAIL", style: .Default, handler: detailActionHandler))
alertMessage.addAction(UIAlertAction(title: "CLOSE", style: .Destructive, handler: nil))

// 将Alert显示出来
self.presentViewController(alertMessage, animated: true, completion: nil)
```

addAction时，Handler可以自定义，方法如下：

```swift
// 以下Handler定义了又一个弹窗，当点击AlertMessage的“Detail”选项时，弹出
let detailActionHandler = { (action:UIAlertAction!) -> Void in
    // 定义一个Alert信息，Style为弹出式Alert
    let detailMessage = UIAlertController(title: "Detail Message", message: "Sorry, the call feature is not available yet. Please retry later.", preferredStyle: .ActionSheet) 
    // 向Alert弹框里面增加一个选项
    detailMessage.addAction(UIAlertAction(title: "OK", style: .Default, handler: nil))
    // 显示弹框
    self.presentViewController(detailMessage, animated: true, completion: nil)
}
```

可以设置弹出信息的样式，有两种方法：
(1) 直接通过设置AlertMessage的setValue：

```swift
alertMessage.setValue(
    NSAttributedString(string: extendedMarker.getProperties(), attributes: [NSFontAttributeName: UIFont.systemFontOfSize(9), NSForegroundColorAttributeName: UIColor.darkGrayColor()]), forKey: "attributedMessage"
)
```

(2) 要想设置对其方式，还要麻烦一些

```swift
// 设置了左对齐的段落格式
let paragraphStyle = NSMutableParagraphStyle()
paragraphStyle.alignment = NSTextAlignment.Left
let messageText = NSMutableAttributedString(
    string: "Message text",
    attributes: [
        NSParagraphStyleAttributeName: paragraphStyle,
        //NSFontAttributeName: UIFont.preferredFontForTextStyle(UIFontTextStyleBody),
        NSFontAttributeName: UIFont(name: "Menlo-Regular",size: 9.0)!,
        NSForegroundColorAttributeName : UIColor.blackColor()
    ]
)
// 此处与上面方法一样，把设定好的Message通过设置setValue的方法，赋值给AlertMessage
alertMessage.setValue(messageText, forKey: "attributedMessage")
```
如果forKey选为`"attributedTitle”`，则更改的是标题的样式


<br />

---
#用代码的方式添加SubView

```swift
// 先在前面定义Spinner
let spinner = UIActivityndicatorView(activityIndicatorStyle: .WhiteLarge)

// 添加到一个叫login的Button上面去
self.login.addsubView(self.spinner)
// 移动spinner的位置（初始时候是在左上角）
self.spinner.frame.origin = CGPointMake(12, 12)
// 启动Spinner的动画
self.spinner.startAnimating()
```