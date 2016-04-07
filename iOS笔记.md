#Swift学习笔记
=============
----------------------
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
    return cell
}
```


##隐藏状态栏（Status Bar）
```swift
override func prefersStatusBarHidden() -> Bool {     return true}
```

