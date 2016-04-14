#TextField
点击`command + “=”`来自适应Label大小

另一种方法是，继承`UITextFieldDelegate`的Protocol，然后实现方法：

```swift
func textFieldSholdReturn(textField: UITextField!) -> Bool {
    textField.resignFirstResponder()
    return true
}
```
然后在使用该方法的控件进行绑定

```swift
xxxxTextField.delegate = self
```
这样在点击下一项时，键盘就会消失


<br />

-----------------------------------------------------------------------
#Slider
slider还要绑定`IBAction`,例如

```swift
@IBAction func heightChanged(sender: AnyObject) {
    // sender可以是任何类型，所以此处要强制转换
    var slider = sender as! UISlider
    var i = Int(slider.value)
    slider.value = Float(i)
}
```

<br />

-----------------------------------------------------------------------
#Date Picker

// 根据输入获取年龄
```swift
let gregorian = NSCalendar(calendarIdentifier: NSCalendarIdentifierGregorian)
let now = NSDate()
```








