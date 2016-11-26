---
layout: post
title: "iOS定位与地图2: 区域监控和iBeacon"
date: 2016-11-26 22:05:39 +0800
categories:
---

## 前言

老规矩：这个系列的博文（iOS定位与地图开头的）翻译自：《Location and Maps Programming Guide》，如果通过搜索引擎来到这里，建议去直接看官方文档好了，那里说最新的。而我无法保证我这里是最新的，我也无法保证我翻译的是对的，对于我理解不了的英文语法，我会尊重Google翻译:-P

---

Core Location框架提供了两种方法来检测用户进入和退出特定区域：地理区域监控（iOS 4.0及更高版本和OS X v10.8及更高版本）和信标区域监控（iOS 7.0及更高版本）。 地理区域是由围绕地球表面上的已知点的指定半径的圆定义的区域。 相比之下，信标区域是由设备接近蓝牙低能量信标定义的区域。 信标本身只是广播特定蓝牙低功耗有效载荷的简单设备 - 您甚至可以通过核心蓝牙框架的一些帮助将您的iOS设备变成信标。

当用户跨越地理边界或当用户进入或退出信标附近时，应用可以使用区域监控被通知。 当信标在iOS设备的范围内时，应用还可以监视与信标的相对距离。 您可以使用这些功能开发许多类型的创新的基于位置的应用程序。 由于地理区域和信标区域不同，因此对于使用哪种区域监控取决于你的应用的使用场景。

在iOS中，与您的应用相关联的区域在任何时候都会被跟踪，包括应用未运行时。 如果在应用程序未运行时跨越了区域边界，则该应用程序将重新启动到后台以处理事件。 同样，如果应用程序在事件发生时被挂起，它会被唤醒并且得到较短的时间（约10秒）处理事件。 必要时，应用程序可以使用 UIApplication 类的`beginBackgroundTaskWithExpirationHandler:`方法请求更多后台执行时间。

在OS X中，区域监视仅在应用程序运行时（前台或后台）并且用户系统处于唤醒状态时工作。 因此，系统不会启动应用程序来交付区域相关通知。

## 确定区域监控的可用性

在尝试监视任何区域之前，您的应用程序应检查当前设备是否支持区域监视。 以下是区域监控可能无法使用的一些原因：

* 设备没有支持区域监控所需的硬件。
* 用户拒绝该应用授权使用区域监控。
* 用户在“设置”应用中停用了位置服务。
* 使用者在「设定」应用程式中，针对装置或应用程式停用「背景应用程式重新整理」。
* 设备处于飞行模式，无法为必要的硬件上电。

在iOS 7.0及更高版本中，在尝试监视区域之前，始终调用 CLLocationManager 的`isMonitoringAvailableForClass:`和`authorizationStatus`类方法。 （在OS X v10.8及更高版本和以前的iOS版本中，请改用`regionMonitoringAvailable`类。）`isMonitoringAvailableForClass:`方法告诉您底层硬件是否支持指定类的区域监视。 如果该方法返回NO，您的应用程序不能在设备上使用区域监控。 如果返回YES，请调用`authorizationStatus`方法以确定应用程序是否当前有权使用位置服务。 如果授权状态为`kCLAuthorizationStatusAuthorized`，您的应用程序可以为其注册的任何区域接收边界通过通知。 如果授权状态设置为任何其他值，则应用程序不会接收这些通知。

> 注意：即使应用程序没有授权使用区域监控，它仍然可以注册区域以供稍后使用。 如果用户随后向应用授予授权，则对这些区域的监视将开始并且将生成随后的边界跨越通知。 如果您不希望在应用未获得授权时仍保持安装区域，可以使用`locationManager:didChangeAuthorizationStatus:`代理方法来检测应用状态的更改，并根据需要删除区域。

最后，如果您的应用程序需要在后台处理位置更新，请务必检查 UIApplication 类的`backgroundRefreshStatus`属性。 您可以使用此属性的值来确定是否可以这样做，如果不是，则警告用户。 请注意，当全局或专门为您的应用停用后台应用刷新设置时，系统不会唤醒您的应用的区域通知。

## 监测地理区域

地理区域监控使用位置服务来检测进入和退出已知地理位置（在“获取用户位置”中了解有关位置服务的更多信息）。 您可以使用此功能在用户靠近特定位置时生成警报或提供其他相关信息。 例如，当接近特定的干洗机时，应用可以通知用户拿起现在准备好的衣服。

### 定义要监视的地理区域

要开始监视地理区域，必须定义区域并将其注册到系统。 在iOS 7.0及更高版本中，您可以使用 CLCircularRegion 类定义地理区域。 （在OS X v10.8及更高版本以及以前的iOS版本中，您将使用 CLRegion 类。）每个创建的区域都必须包含定义所需地理区域的数据和唯一标识符字符串。 标识符字符串是您的应用程序以后确定区域的唯一保证方法。 要注册区域，请调用 CLLocationManager 对象的`startMonitoringForRegion:`方法。

清单1显示了一个基于圆形覆盖区域创建新地理区域的示例方法。 覆盖图的中心点和半径形成了区域的边界，但是如果半径太大而无法监视，则会自动缩小。 您不需要保存对所创建区域的强引用，但如果您计划稍后访问该区域信息，可能希望存储该区域的标识符。

清单1 基于Map Kit覆盖创建和注册地理区域

```
- (void)registerRegionWithCircularOverlay:(MKCircle*)overlay andIdentifier:(NSString*)identifier {
    // If the overlay's radius is too large, registration fails automatically,
    // so clamp the radius to the max value.
    CLLocationDistance radius = overlay.radius;
    if (radius > self.locManager.maximumRegionMonitoringDistance) {
        radius = self.locManager.maximumRegionMonitoringDistance;
    }

    // Create the geographic region to be monitored.
    CLCircularRegion *geoRegion = [[CLCircularRegion alloc]
        initWithCenter:overlay.coordinate
                radius:radius
            identifier:identifier];
    [self.locManager startMonitoringForRegion:geoRegion];
}
```

监控地理区域在授权应用程序注册后立即开始。 然而，不要期望立即接收事件，因为只有跨越边界时生成事件。 具体地，如果用户的位置在注册时间已经在区域内，则位置管理器不会自动生成事件。 相反，您的应用程序必须等待用户跨越区域边界后才能生成事件并发送到代理。 要检查用户是否已位于区域的边界内，请使用 CLLocationManager 类的`requestStateForRegion:`方法。

指定要监视的区域集时应谨慎。 区域是共享系统资源，系统范围内可用的区域总数有限。 因此，Core Location 将单一应用程式同时监控的区域数量限制为20个。 要解决此限制，请考虑仅注册用户紧邻的那些区域。 随着用户的位置更改，您可以删除现在更远的区域，并添加在用户路径上出现的区域。 如果尝试注册区域时空间不足，则位置管理器调用其代理的`locationManager:monitoringDidFailForRegion:withError:`方法并使用`kCLErrorRegionMonitoringFailure`错误代码。

### 处理地理区域的边界交叉事件

默认情况下，每当用户的当前位置越过区域边界时，系统会为您的应用程序生成适当的区域事件。 应用程序可以实现以下方法来处理边界交叉：

* `locationManager:didEnterRegion:`
* `locationManager:didExitRegion:`

定义和注册区域时，可以通过显式设置 CLRegion 类的`notifyOnEntry`和`notifyOnExit`属性来自定义哪些边界跨越事件通知您的应用程序。 （两个属性的默认值均为`YES`。）例如，如果要仅在用户退出区域边界时收到通知，可以将区域的`notifyOnEntry`属性的值设置为`NO`。

直到超过边界加上系统定义的缓冲距离后系统才会报告边界跨越事件。 该缓冲值防止系统在用户行进靠近边界的边缘时快速连续地生成许多输入和退出事件。

当跨过区域边界时，最可能的响应是提醒用户接近目标项目。 如果您的应用在后台运行，您可以使用本地通知提醒用户; 否则，您可以简单地发布警报。

## 监控信标区域

信标区域监控使用iOS设备的板载无线电来检测用户是否位于正在广播iBeacon信息的蓝牙低功耗设备附近。 与地理区域监控一样，您可以使用此功能在用户进入或退出信标区域时生成警报或提供其他相关信息。 然而，信标区域不是由固定地理坐标来标识，而是通过设备接近蓝牙低能量信标来标识信标区域，该蓝牙低能量信标广播以下值的组合：

* 接近度UUID（通用唯一标识符），它是唯一标识一个或多个信标用于特定类型或特定组织的128位值。
* 主值，它是一个16位无符号整数，可用于对具有相同接近度UUID的相关信标进行分组。
* 次要值，它是一个16位无符号整数，用于区分具有相同接近度UUID和主值的信标

因为单个信标区域可以表示多个信标，所以信标区域监视支持若干有趣的使用情况。 例如，专用于增强特定百货商店的客户体验的应用可以使用相同的接近度UUID来监控百货商店链中的所有商店。 当用户接近商店时，应用检测商店的信标并使用那些信标的主值和次要值来确定附加信息，例如遇到了哪个特定商店或用户所在的商店的哪个部分。（注意，尽管每个信标必须广播邻近UUID，但主要和次要值是可选的。

### 定义要监视的信标区域

要开始监视信标区域，请定义该区域并将其注册到系统。 您可以使用 CLBeaconRegion 类的适当初始化方法定义信标区域。 当您创建 CLBeaconRegion 对象时，可以指定要监控的信标的`proximityUUID`，`major`和`minor`属性（接近度UUID是必需的;主要和次要值是可选的）。 您还必须提供一个唯一标识该区域的字符串，以便您可以在代码中引用它。 注意，区域的标识符与信标广播的标识信息无关。

要注册信标区域，请调用 CLLocationManager 对象的`startMonitoringForRegion:`方法。 清单2显示了创建和注册信标区域的示例方法。

清单2创建和注册信标区域

```
- (void)registerBeaconRegionWithUUID:(NSUUID *)proximityUUID andIdentifier:(NSString*)identifier {
    // Create the beacon region to be monitored.
    CLBeaconRegion *beaconRegion = [[CLBeaconRegion alloc]
        initWithProximityUUID:proximityUUID
                   identifier:identifier];

    // Register the beacon region with the location manager.
    [self.locManager startMonitoringForRegion:beaconRegion];
}
```

与地理区域监控一样，在针对授权的应用注册之后立即开始监视信标区域。 当用户的设备检测到正在广播由注册的信标区域（接近度UUID，主值和次要值）定义的标识信息的信标时，系统为应用生成适当的区域事件。

> 注意：仅使用UUID值配置信标区域并不罕见。 这样做产生当设备进入具有指定的UUID的任何信标的范围内时激活的区域。 进入信标区域后，您将开始测量信标以获取关于附近的特定信标的详细信息。 有关信标测距的更多信息，请参阅“使用测距确定信标的接近度”。

### 处理信标区域的边界交叉事件

当用户进入注册的信标区域时，位置管理器调用其委托对象的`locationManager:didEnterRegion:`。 类似地，当用户不再在注册的信标区域中的任何信标的范围内时，位置管理器调用其委托对象的`locationManager:didExitRegion:`。 请注意，用户必须跨越区域的边界才能触发这些调用; 特别是，位置管理器不调用`locationManager:didEnterRegion:`如果用户已经在区域内。 您可以实现这些委托方法以适当地警告用户或显示位置特定的UI。

您可以通过设置信标区域的`notifyOnEntry`和`notifyOnExit`属性来指定哪些边界跨越事件应通知您的应用程序。 （两个属性的默认值均为`YES`。）例如，如果要仅在用户退出区域边界时通知您，可以将区域的`notifyOnEntry`属性的值设置为`NO`。

您还可以在进入入信标区域后推迟到用户打开设备的显示屏再通知用户。 为此，只需将信标区域的`notifyEntryStateOnDisplay`属性值设置为`YES`，并在注册信标区域时将区域的`notifyOnEntry`属性设置为`NO`。 要防止将冗余通知传递给用户，请每个区域条目仅发布一次本地通知。

#### 使用测距确定信标的接近度

当用户的设备在注册的信标区域内时，应用可以使用 CLLocationManager 类的`startRangingBeaconsInRegion:`方法来确定该区域中的一个或多个信标的相对接近度，并且当该距离改变时被通知。 （在尝试在信标区域中定位信标之前，始终调用 CLLocationManager 类的`isRangingAvailable`类方法。）知道与信标的相对距离对于许多应用程序可能很有用。 例如，想象一个博物馆在每个展品上放置一个信标。 博物馆专用应用可以使用特定展览的接近度作为提示来提供关于该展品的信息。

只要指定信标区域中的信标落在范围内，超出范围或它们的邻近度改变，位置管理器就调用其委托对象的`locationManager:didRangeBeacons:inRegion:`。 此委托方法提供了一个 CLBeacon 对象数组，表示当前在范围内的信标。 信标数组通过与设备的相对距离排序，其中最近的信标在数组的开头。 您可以使用这些对象中的信息来确定用户与每个信标的接近程度。 CLBeacon 对象的`proximity`属性中的值给出了与信标的相对距离的一般意义。

> 注意：信标测距取决于检测蓝牙低能无线电信号的强度，并且这些信号的精度会被墙壁、门和其他物理对象衰减（或减弱）。 信号也受水的影响，这意味着人体本身将影响信号。 在规划iBeacon部署时，了解这些因素很重要，因为它们会影响每个信标报告的接近值（proximity）。 如果需要，在部署期间请使用每个 CLBeacon 对象报告的`accuracy`和`rssi`值来调整信标的位置。

受本节前面所述的示例博物馆应用程序的启发，清单3显示了如何使用信标的`proximity`属性来确定它与用户的设备的相对距离。 当数组中最近的信标的接近度相对接近用户（由`CLProximityNear`常数定义的）时，代码呈现提供关于特定博物馆展览的更多信息的UI。

清单3 确定信标和设备之间的相对距离

```
// Delegate method from the CLLocationManagerDelegate protocol.
- (void)locationManager:(CLLocationManager *)manager
        didRangeBeacons:(NSArray *)beacons
               inRegion:(CLBeaconRegion *)region {

    if ([beacons count] > 0) {
        CLBeacon *nearestExhibit = [beacons firstObject];

        // Present the exhibit-specific UI only when
        // the user is relatively close to the exhibit.
        if (CLProximityNear == nearestExhibit.proximity) {
            [self presentExhibitInfoWithMajorValue:nearestExhibit.major.integerValue];
        } else {
            [self dismissExhibitInfo];
    }
}
```

要在您的应用中推广一致的结果，请仅在您的应用处于前台时使用信标测距。 如果您的应用程序在前台，则可能是设备在用户的手中，并且设备对目标信标的视图具有较少的障碍物。 在前台运行还通过仅在用户主动使用设备时处理传入的信标信号来促进更好的电池寿命。

> 注意：如果多个信标设备广播的接近度UUID、主要值和次要值的组合相同，则它们可能由`locationManager:didRangeBeacons:inRegion:`方法报告为具有不同的近似和准确度。 建议对每个信标设备进行唯一标识。
> 此外，如果您对已配置为信标的iOS设备进行测距，则可能会有一段很短的时间内`locationManager:didRangeBeacons:inRegion:`方法报告为具有相同接近度UUID、主要和次要值的两个设备。 出现此行为是因为iOS设备的蓝牙标识符会定期更改隐私权问题。 基于原始蓝牙标识符的`proximity`属性在标识符更改的2秒内报告`CLProximityUnknown`值。 在10秒内，标识符解决并且仅报告一个信标区域。

## 将iOS设备转换为iBeacon

任何支持使用蓝牙低功耗共享数据的iOS设备都可以用作iBeacon。 因为您编写的应用程序必须在前台运行，iOS设备上的iBeacon支持旨在用于测试目的，以及始终在前台运行的应用程序（如销售点应用程序）。 对于其他类型的iBeacon实现，您需要从第三方制造商处获取专用信标硬件。

因为将iOS设备转换为信标需要使用Core Bluetooth框架，请务必在您的Xcode项目中将应用程序链接到CoreBluetooth.framework。 要访问框架的类和头文件，请在任何相关源文件的顶部包含`#import <CoreBluetooth/CoreBluetooth.h>`语句。

### 创建和广播信标区域

要将iOS设备用作信标，您首先生成128位的UUID作为您的信标区域的邻近UUID。 打开终端并在命令行上键入uuidgen。 您在收到一个以连字符分隔的ASCII字符串形势的128位值，如本示例所示。

```
$ uuidgen
39ED98FF-2900-441A-802F-9C398FC199D2
```

接下来，使用您为信标的邻近UUID生成的UUID创建信标区域，根据需要定义主要和次要值。 确保还为新区域使用唯一的字符串标识符。 此代码显示如何使用上面的示例UUID创建一个新的信标区域。

```
    NSUUID *proximityUUID = [[NSUUID alloc]
        initWithUUIDString:@"39ED98FF-2900-441A-802F-9C398FC199D2"];

    // Create the beacon region.
    CLBeaconRegion *beaconRegion = [[CLBeaconRegion alloc]
        initWithProximityUUID:proximityUUID
                  identifier:@"com.mycompany.myregion"
```

现在您已经创建了一个信标区域，您需要使用Core Bluetooth框架的 CBPeripheralManager 类来广播您的信标的邻近UUID（以及您指定的任何主要或次要值）。 在Core Bluetooth中，外围设备是使用蓝牙低功耗来广播和共享数据的设备。 广播您的信标的数据是其他设备可以检测和范围你的信标的唯一方式。

要在Core Bluetooth中广播外围数据，请在 CBPeripheralManager 对象的实例上调用 CBPeripheralManager 类的`startAdvertising:`方法。 此方法需要广播的数据的字典（NSDictionary的一个实例）。 如下面的例子所示，使用 CLBeaconRegion 类的`peripheralDataWithMeasuredPower:`方法接收一个字典，该字典编码您的信标的标识信息以及Core Bluetooth作为外设广播信标所需的其他信息。

```
    // Create a dictionary of advertisement data.
    NSDictionary *beaconPeripheralData =
        [beaconRegion peripheralDataWithMeasuredPower:nil];
```

> 注意：上面的代码将值`nil`传递给`measuredValue`参数，它代表距离一米远的信标的接收信号强度指示器（RSSI）值（以分贝为单位）。 该值在测距期间使用。 指定`nil`会使用设备的默认值。 （可选）如果需要进一步校准设备以在某些环境中获得更好的测距性能，请指定RSSI值。

接下来，创建 CBPeripheralManager 类的实例，并要求它向其他设备广播您的信标以进行检测，如下面的代码所示。

```
    // Create the peripheral manager.
    CBPeripheralManager *peripheralManager = [[CBPeripheralManager alloc]
        initWithDelegate:self queue:nil options:nil];

    // Start advertising your beacon's data.
    [peripheralManager startAdvertising:beaconPeripheralData];
```

> 重要：当创建外围管理器对象时，外设管理器调用其委托对象的`peripheralManagerDidUpdateState:`方法。 您必须实现此委派方法，以确保支持蓝牙低功耗，并可在本地外围设备上使用。 有关此委托方法的更多信息，请参阅 CBPeripheralManagerDelegate 协议参考。

将应用广告为信标后，您的应用必须继续在前台运行才能广播所需的蓝牙信号。 如果用户退出应用程序，系统会停止将您的设备作为外围设备进行广告。

有关如何使用外围设备管理器使用蓝牙低功耗广播数据的更多信息，请参阅 CBPeripheralManager 类参考和Core Bluetooth编程指南。

## 测试iOS应用程序的区域监控支持

当在iOS模拟器或设备上测试区域监视代码时，意识到区域事件可能不会在跨越区域边界后立即发生。 为了防止虚假通知，iOS不会传送区域通知，直到满足某些阈值条件。 具体地，用户的位置必须跨越区域边界，远离边界移动最小距离，并且在报告通知之前保持在该最小距离处至少20秒。

特定的阈值距离由当前可用的硬件和定位技术确定。 例如，如果禁用Wi-Fi，则区域监视显着不太准确。 但是，出于测试目的，您可以假定最小距离约为200米。
