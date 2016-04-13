#使用Mapbox
1. 在`info.plist`里面增加`MGLMapboxAccessToken`选项，值为`pk.eyJ1IjoiaGFpZG9uZ3ciLCJhIjoiY2lseDEyNnF5MDVxaHVka3M4eHI3bWVnaiJ9.3zSqNekaroRp0zGsCCCfYQ`
2. 在`Enbedded Binaries`里面Copy进去`Mapbox.framework`
3. 将 View 的类改为`MGLMapView`
4. 在 ViewController 里面继承`MGLMapViewDelegate `协议
5. 将 Map View 与 ViewController 的 `delegate`绑定


#显示用户定位
1. 在`Attributes inspector`将`Shows User Location`设为`On`
2. 在`info.plist`添加`NSLocationAlwaysUsageDescription`或者`NSLocationWhenInUseUsageDescription`并将内容设置为`”Shows your location on the map and helps improve OpenStreetMap.”`
3. 在模拟器`Debug ‣ Location ‣ Custom Location`开启位置


#设置地图的初始位置
在`viewDidLoad`中，添加

```swift
// 设定初始坐标和缩放比例
mapView.setCenterCoordinate(CLLocationCoordinate2D(latitude: 40.7326808, longitude: -73.9843407), zoomLevel: 12, animated: false)
```
也可以设定一个Bounding Box来代替初始坐标和zoom

```swift
// 设置地图初始的Bounding Box
let lowerCorner = CLLocationCoordinate2D(latitude: -39.1592, longitude: 140.9617)
let upperCorner = CLLocationCoordinate2D(latitude: -33.9806, longitude: 149.9767)
let bounds = MGLCoordinateBounds(sw: lowerCorner, ne: upperCorner)
mapView.setVisibleCoordinateBounds(bounds, animated: false)
```
也可以设置地图的缩放与Annotation相符

```swift
// 让Map缩放与点的位置相符
mapView.showAnnotations(mapView.annotations!, animated: false)
```


#设置初始进入时视角转换
在`ViewDidLoad`中，添加

```swift
// 从中心点开始，旋转 direction 度
let center = CLLocationCoordinate2D(latitude: (-37.8), longitude: 144.96)
mapView.setCenterCoordinate(center, zoomLevel: 10, direction: 0, animated: false)
```
在`viewDidAppear`中，添加

```swift
// 设置相机中心店位置（和地图中心一样），设置起始点高度 fromDistance，倾斜角度 pitch
let camera = MGLMapCamera(lookingAtCenterCoordinate: mapView.centerCoordinate, fromDistance: 2000, pitch: 0, heading: 0)
// 设置相机缩放持续时间为10秒钟
mapView.setCamera(camera, withDuration: 10, animationTimingFunction: CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut))
```

#设置地图样式
在`ViewDidLoad`中，添加

```swift
// 可以使用自定义的地图样式
mapView.styleURL = NSURL(string: "mapbox://styles/mapbox/light-v8")
// 或者利用内置的地图样式
mapView.styleURL = MGLStyle.emeraldStyleURL()
```
<br />

---

#在地图中添加标注（Annotation）
##添加一个点（Marker）
在`ViewDidLoad`中，添加

```swift
// 定义一个点，设置它的坐标、题注
let hello = MGLPointAnnotation()
hello.coordinate = CLLocationCoordinate2D(latitude: -37.80, longitude: 144.96)
hello.title = "Hello world!"
hello.subtitle = "Welcome to my marker"
// 把点添加在地图上
mapView.addAnnotation(hello)
```

##添加一个自定义标志（Custom Marker）
为了添加自定义的图标，需要实现以下方法：

```swift
func mapView(mapView: MGLMapView, imageForAnnotation annotation: MGLAnnotation) -> MGLAnnotationImage? {
    // 试着重用名叫'unimelb'的图像，重复利用Annotation，就像Table Cell一样
    var annotationImage = mapView.dequeueReusableAnnotationImageWithIdentifier("unimelb")
        
    if annotationImage == nil {
        var image = UIImage(named: "unimelb")!
        // 锚点始终在图像中间，为了使其在图像底部，需要坐相应转换
        image = image.imageWithAlignmentRectInsets(UIEdgeInsetsMake(0, 0, image.size.height/2, 0))
        annotationImage = MGLAnnotationImage(image: image, reuseIdentifier: "unimelb")
    }
    return annotationImage
}
```

##添加一条线（GeoJSON Line）
在方法drawPolyline()里面，解析GeoJSON（略）并显示出来

```swift
// 将坐标存在数组里面
func drawPolyline() {
    var coordinates: [CLLocationCoordinate2D] = []
    // 定义一个MGLPolyline类型的常量
    let line = MGLPolyline(coordinates: &coordinates, count: UInt(coordinates.count))
    // 设定Polyline的名称，可用作Call Out
    line.title = "Crema to Council Crest"

    // 在主线程上添加Annotation
    dispatch_async(dispatch_get_main_queue(), {
        // Unowned reference to self to prevent retain cycle
        [unowned self] in
        self.mapView.addAnnotation(line)
    })
}
```
另外，还要实现以下方法，来设定颜色、线宽：

```swift
// 所有图形的透明度
func mapView(mapView: MGLMapView, alphaForShapeAnnotation annotation: MGLShape) -> CGFloat {
    return 1
}

// Polyline的宽度 
func mapView(mapView: MGLMapView, lineWidthForPolylineAnnotation annotation: MGLPolyline) -> CGFloat {
    return 2.0
}

// 设定颜色
func mapView(mapView: MGLMapView, strokeColorForShapeAnnotation annotation: MGLShape) -> UIColor {
    // 通过Title来确定颜色
    if (annotation.title == "Crema to Council Crest" && annotation is MGLPolyline) {
        // Mapbox cyan 蓝绿色
        return UIColor(red: 59/255, green:178/255, blue:208/255, alpha:1)
    } else {
        return UIColor.redColor()
    }
}

```



##添加一个图形（Polygon）
方法同Polyline，解析GeoJSON，把坐标存成一个点的数组

```swift
func drawPolygon() {
    // 建立一个CLLocationCoordinate2D数组，首尾相同
    var coordinates: [CLLocationCoordinate2D] = []
    // 在这里解析JSON将数组填满
    let shape = MGLPolygon(coordinates: &coordinates, count: UInt(coordinates.count))
    // 向地图中添加图形
    mapView.addAnnotation(shape)
}
```
除了上面的`alphaForShapeAnnotation`和`strokeColorForShapeAnnotation`，还要定义填充颜色：

```swift
// 返回值是UIColor
func mapView(mapView: MGLMapView, fillColorForPolygonAnnotation annotation: MGLPolygon) -> UIColor {
    return UIColor(red: 59/255, green: 178/255, blue: 208/255, alpha: 1)
}
```
<br />

#设定呼出界面（Call Out）
##使用默认的呼出界面
先设定一个Annotation，与前面的相同，只是封装在一个func里面

```swift
// 设定一个Annotation，在里面添加Callout
func addAnnotation() {
    // 设定一个点的Annotation
    let annotation = MGLPointAnnotation()
    annotation.coordinate = CLLocationCoordinate2DMake(35.03946, 135.72956)
    annotation.title = "Kinkaku-ji"
    annotation.subtitle = "\(annotation.coordinate.latitude), \(annotation.coordinate.longitude)"
    // 向Map上添加点
    mapView.addAnnotation(annotation)
    // 让Map缩放与点的位置相符
    mapView.showAnnotations(mapView.annotations!, animated: false)
    // 呼出Callout View（有没有这句区别不大？）
    mapView.selectAnnotation(annotation, animated: true)
}
```
![Callout](https://www.mapbox.com/ios-sdk/images/callout-delegate.png)

以下几个函数激活Callout功能，并定制Callout的外观

```swift
// 激活Annotation的Callout功能
func mapView(mapView: MGLMapView, annotationCanShowCallout annotation: MGLAnnotation) -> Bool {
    return true
}

// 在Annotation左边添加描述信息的Label
func mapView(mapView: MGLMapView, leftCalloutAccessoryViewForAnnotation annotation: MGLAnnotation) -> UIView? {
    if (annotation.title! == "Kinkaku-ji") {
        let label = UILabel(frame: CGRectMake(0, 0, 60, 50))
        label.textAlignment = .Right
        label.textColor = UIColor(red: 0.81, green: 0.71, blue: 0.23, alpha: 1)
        label.text = "金閣寺"
        return label
    }
    return nil
}
    
// 在Annotation右边添加详细说明按钮
func mapView(mapView: MGLMapView, rightCalloutAccessoryViewForAnnotation annotation: MGLAnnotation) -> UIView? {
    return UIButton(type: .DetailDisclosure)
}
    
// 设定点击了详细信息按钮后的呼出界面
func mapView(mapView: MGLMapView, annotation: MGLAnnotation, calloutAccessoryControlTapped control: UIControl) {
    // 隐藏Callout
    mapView.deselectAnnotation(annotation, animated: false)
    // 定义一个Alert信息，Style为弹出式Alert
    let alertMessage = UIAlertController(title: annotation.title!, message: "A lovely (if touristy) place.", preferredStyle: .Alert)
    // 向Alert弹框里面增加一个选项
    alertMessage.addAction(UIAlertAction(title: "OK", style: .Default, handler: nil))
    // 显示弹框
    self.presentViewController(alertMessage, animated: true, completion: nil)
}
```

<br />

---
#将屏幕上的点转化成坐标
![Point conversion](https://www.mapbox.com/ios-sdk/images/point-conversion.gif)

```swift
import UIKit
import Mapbox

class ViewController: UIViewController {

    @IBOutlet var mapView: MGLMapView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // double tapping zooms the map, so ensure that can still happen
        let doubleTap = UITapGestureRecognizer(target: self, action: nil)
        doubleTap.numberOfTapsRequired = 2
        mapView.addGestureRecognizer(doubleTap)
        
        // delay single tap recognition until it is clearly not a double
        let singleTap = UITapGestureRecognizer(target: self, action: #selector(ViewController.handleSingleTap(_:)))
        singleTap.requireGestureRecognizerToFail(doubleTap)
        mapView.addGestureRecognizer(singleTap)
        
        // convert `mapView.centerCoordinate` (CLLocationCoordinate2D)
        // to screen location (CGPoint)
        let centerScreenPoint: CGPoint = mapView.convertCoordinate(mapView.centerCoordinate, toPointToView: mapView)
        print("Screen center: \(centerScreenPoint) = \(mapView.center)")
    }
    
    
    func handleSingleTap(tap: UITapGestureRecognizer) {
        // convert tap location (CGPoint)
        // to geographic coordinates (CLLocationCoordinate2D)
        let location: CLLocationCoordinate2D = mapView.convertPoint(tap.locationInView(mapView), toCoordinateFromView: mapView)
        print("You tapped at: \(location.latitude), \(location.longitude)")
        
        // create an array of coordinates for our polyline
        var coordinates: [CLLocationCoordinate2D] = [mapView.centerCoordinate, location]
        
        // remove existing polyline from the map, (re)add polyline with coordinates
        if (mapView.annotations?.count != nil) {
            mapView.removeAnnotations(mapView.annotations!)
        }
        let polyline = MGLPolyline(coordinates: &coordinates, count: UInt(coordinates.count))
        mapView.addAnnotation(polyline)
    }   
}
```
