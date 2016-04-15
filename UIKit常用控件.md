#关掉UITextField输入的三种方法

###1. 点击Button之后关闭键盘

```swift
@IBAction func buttonPressed(sender: UIButton) {
    textBox.resignFirstResponder()
}
```

###2. 点击Return之后关闭键盘
首先要在ViewController继承`UITextFieldDelegate`的Protocol

```swift
func textFieldShouldReturn(textField: UITextField) -> Bool {
    self.view.endEditing(true)
    return true
}
```
```swift
/// 另一种方法
override func ViewDidLoad() {
    textField.delegate = self
}
func textFieldSholdReturn(textField: UITextField!) -> Bool {
    textField.resignFirstResponder()
    return true
}
```


###3. 点击输入框之外的空白处时关闭键盘
同上，继承`UITextFieldDelegate`的Protocol

```swift
override func touchesBegan(touches: Set<UITouch>, withEvent event: UIEvent?) {    
    textBox.resignFirstResponder()
}
```

<br />

----------------------------------------------------------------------
#UIPickView的用法
##1. 使用UIPickerView
**1. 继承Protocols，然后再StoryBoard连接DataSource和Delegate**

```swift
class ViewController: UIViewController, UIPickerViewDataSource, UIPickerViewDelegate {
    //...
}
```
**2. 实现`UIPickerViewDataSource`的以下方法**

```swift
func numberOfComponentsInPickerView(pickerView: UIPickerView) -> Int {
    return 1
}
func pickerView(pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
    return color.count
}
func pickerView(pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String! {
    return color[row]
}
```
**3. 实现`UIPickerViewDelegate`的以下方法**

```swift
func pickerView(pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
    textLabel.text = color[row]
}
```

<br />

----------------------------------------------------------------------
#UITextField与UIPickerView相结合
##1. 将UIPickerView作为输入框



##2. 删除UITextField的光标与Copy/Paste
[http://blog.apoorvmote.com/](http://blog.apoorvmote.com/how-to-show-detail-info-with-popover/)

**(1) 为UITextField新建一个类，并与TextField连接**

```swift
import UIKit

class TextField: UITextField {
    // 以下方法，删除了Cursor，禁用Copy/Paste
    override func caretRectForPosition(position: UITextPosition) -> CGRect {
        return CGRect.zero
    }
    override func selectionRectsForRange(range: UITextRange) -> [AnyObject] {
        return []
    }
    override func canPerformAction(action: Selector, withSender sender: AnyObject?) -> Bool {
        if action == #selector(NSObject.copy(_:)) || action == #selector(NSObject.selectAll) || action == #selector(NSObject.paste) {
            return false
        }
        return super.canPerformAction(action, withSender: sender)
    }
}
```
**(2) 将UITextField作为Action，并把Event设成`Editing Did Begin`**

![](http://blog.apoorvmote.com/wp-content/uploads/2016/01/textfield-as-action.png)

```swift
@IBAction func textFieldEditing(sender: TextField) {
    let datePicker = UIDatePicker()
    datePicker.datePickerMode = UIDatePickerMode.Date
    sender.inputView = datePicker
    datePicker.addTarget(self, action: #selector(ViewController.datePickerValueChanged), forControlEvents: UIControlEvents.ValueChanged)
}
```
**(3) 实现以下方法，监测PickerView值的变化，并更新TextField**
在`UIPickerView`中使用`didSelectRow`
在`UIDataPickerView`中使用`datePickerValueChanged `

```swift
func datePickerValueChanged(sender: UIDatePicker) {
    let dateformatter = NSDateFormatter()
    dateformatter.dateStyle = NSDateFormatterStyle.MediumStyle
    dateformatter.timeStyle = NSDateFormatterStyle.NoStyle
    dateTextField.text = dateformatter.stringFromDate(sender.date)
}

// 关闭UIPickerView
override func touchesBegan(touches: Set, withEvent event: UIEvent?) {
    self.view.endEditing(true)
}
```
![效果图](http://blog.apoorvmote.com/wp-content/uploads/2016/01/final-datepicker-textfield.gif)

<br />

----------------------------------------------------------------------
#Action Sheet的用法

```swift
@IBAction func deletePressed(sender: UIButton) {
    // 1. 新建一个ActionSheet
    let actionSheet = UIAlertController(title: "Are you sure you want to delete?", message: "You cannot recover once deleted", preferredStyle: UIAlertControllerStyle.ActionSheet)
    // 2. 定义一个Delete按键
    let deleteAction = UIAlertAction(title: "Delete", style: UIAlertActionStyle.Destructive) { (alert:UIAlertAction) -> Void in
        print("Delete pressed")
    }
    // 3. 定义一个Cancel按键
    let cancelAction = UIAlertAction(title: "Cancel", style: UIAlertActionStyle.Cancel) { (alert:UIAlertAction) -> Void in
        print("Cancel pressed")
    }
    // 4. 在ActionSheet上添加案件
    actionSheet.addAction(deleteAction)
    actionSheet.addAction(cancelAction)
    // 5. 显示ActionSheet
    self.presentViewController(actionSheet, animated: true, completion: nil)
}
```

<br />

----------------------------------------------------------------------
#UIStepper的用法
**(1) 为Stepper绑定`@IBOutlet`，初始化设定**

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // 1. Wraps : If its true then if you increment after maximum value then minimum value appears.
    stepper.wraps = false
    // 2. Autorepeat & continuous : Both do potentially same thing with small difference. If you click and hold then you can either increase or decrease value consistently.
    stepper.autorepeat = true
    stepper.continuous = true
    // 3. Tint color : You can change default color of stepper
    stepper.tintColor = UIColor.redColor()
    // 4. Minimum & maximum value : This is very self explanatory
    stepper.minimumValue = -10
    stepper.maximumValue = 10
    // 5. Value : This would set initial value. If you don’t set this then default value is zero.
    stepper.value = 5
}
```

**(2) 绑定`@IBAction`，Event是`Value Changed`**

```swift
@IBAction func stepperValueChanged(sender: UIStepper) {
    numLabel.text = "\(Int(sender.value))"
}
```

<br />

----------------------------------------------------------------------
#UISwitch的用法
**绑定`@IBOutlet`**

```swift
override func viewDidLoad() {
    super.viewDidLoad() 
    mySwitch.on = false
    self.view.backgroundColor = UIColor.lightGrayColor()
}
```
**绑定`@IBAction`**

```swift
@IBAction func switchChanged(sender: UISwitch) {
    if sender.on {
        self.view.backgroundColor = UIColor.whiteColor()
    } else {
        self.view.backgroundColor = UIColor.lightGrayColor()
    }
}
```

<br />

----------------------------------------------------------------------
#UISlider的用法
**将UISlider绑定`IBAction`**

```swift
@IBAction func heightChanged(sender: AnyObject) {
    // sender可以是任何类型，所以此处要强制转换
    var slider = sender as! UISlider
    // 让Slider只能选择整型
    var i = Int(slider.value)
    slider.value = Float(i)
}
```

<br />

----------------------------------------------------------------------
#Segmented Control的用法
**绑定`@IBOutlet`和`@IBAction`**

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    segment.selectedSegmentIndex = 0     
    content.text = "New segment feature is included"
}

@IBAction func indexChanged(sender: UISegmentedControl) {
    switch segment.selectedSegmentIndex {
    case 0:
        content.text = "New segment feature is included"
    case 1:
        content.text = "We make iPhone apps"
    case 2:
        content.text = "apoorv@apoorvmote.com"
    default:
        break    
    } 
}
```










------
[How to show detail info with popover](http://blog.apoorvmote.com/how-to-show-detail-info-with-popover/)
[How to delete & reorder rows uitableview in iOS Swift](http://blog.apoorvmote.com/delete-reorder-rows-uitableview-ios-swift/)
[How to create uitableview with multiple sections in iOS Swift](http://blog.apoorvmote.com/uitableview-with-multiple-sections-ios-swift/)
[How to show and hide static cells with uiswitch](http://blog.apoorvmote.com/how-to-show-and-hide-static-cells-with-uiswitch/)
[How to pop up datepicker inside static cells](http://blog.apoorvmote.com/how-to-pop-up-datepicker-inside-static-cells/)
[How to move uitextfield up when keyboard appears](http://blog.apoorvmote.com/move-uitextfield-up-when-keyboard-appears/)