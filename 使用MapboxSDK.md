#使用Mapbox
1. 在`info.plist`里面增加`MGLMapboxAccessToken`选项，值为`pk.eyJ1IjoiaGFpZG9uZ3ciLCJhIjoiY2lseDEyNnF5MDVxaHVka3M4eHI3bWVnaiJ9.3zSqNekaroRp0zGsCCCfYQ`
2. 在`Enbedded Binaries`里面Copy进去`Mapbox.framework`
3. 将 View 的类改为`MGLMapView`
4. 在 ViewController 里面继承`MGLMapViewDelegate `协议
5. 将 Map View 与 ViewController 的 `delegate`绑定


