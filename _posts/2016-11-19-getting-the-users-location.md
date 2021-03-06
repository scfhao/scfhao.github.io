---
layout: post
title: "iOS定位与地图1: 获取用户的位置"
date: 2016-11-19 22:02:39 +0800
categories:
---

## 前言

今天再开一个坑，把以前阅读 Apple 的定位官方文档时的翻译陆续整理、发到博客上，这么做的原因是：我不想再用印象笔记了，不想安装客户端，即使是网页版，在编辑、文档组织上还没博客方便，嗯，于是就开坑了。老规矩：这个系列的博文（iOS定位与地图开头的）翻译自：《Location and Maps Programming Guide》，如果通过搜索引擎来到这里，建议去直接看官方文档好了，那里说最新的。而我无法保证我这里是最新的，我也无法保证我翻译的是对的，对于我理解不了的英文语法，我会尊重Google翻译:-P

---

应用程序使用位置数据出于许多目的，从社交网络到逐步导航服务。 他们通过 Core Location 框架的类获得位置数据。 此框架提供了几种服务，您可以使用它们来获取和监视设备的当前位置：

* 标准位置服务提供了一种高度可配置的方式来获取当前位置并跟踪更改。
* 区域监视允许您监视穿过定义的地理区域和蓝牙低功耗信标区域的边界。 （Beacon区域监控仅适用于iOS）。
* 重大变化位置服务提供了一种获取当前位置的方式，并在发生重大变化时通知，但正确使用它以避免使用太多功率是至关重要的。

要了解如何在节省电力的同时为用户提供卓越的位置感知体验，请参阅《Reduce Location Accuracy and Duration》。

要使用 Core Location 框架的功能，必须在Xcode项目中将应用程序链接到CoreLocation.framework。 要访问框架的类和头，请在任何相关源文件的顶部包含一个#import <CoreLocation/CoreLocation.h>语句。

有关Core Location框架的类的一般信息，请参阅《Core Location Framework Reference》。

## 在iOS应用中请求使用位置服务

如果您的iOS应用程序中需要位置服务才能正常工作，请在应用程序的 Info.plist 文件中包含 UIRequiredDeviceCapabilities 键。 App Store使用此键中的信息来防止用户将应用程序下载到不包含列出的功能的设备上。

UIRequiredDeviceCapabilities 的值是一个字符串数组，表示您的应用程序需要的功能。两个字符串与位置服务相关：

* 一般如果您需要位置服务，请包含 location-services 字符串。
* 如果您的应用程序需要仅由GPS硬件提供的精度，请包含 gps 字符串。

> 重要的：如果您的iOS应用程序使用位置服务，但能够在没有它们的情况下成功运行，请不要在 UIRequiredDeviceCapabilities 键中包含相应的字符串。

更多关于 UIRequiredDeviceCapabilities 关键字的信息，参考《Infomation Property List Key Reference》。

## 获取用户的当前位置

Core Location 框架允许您查找设备的当前位置，并在应用程序中使用该信息。该框架会根据您配置此服务的方式将设备的位置报告给你的代码，还会在接收新数据或改进数据时提供定期更新。

两个可以提供用户当前位置的服务：

* 标准位置服务是一种可配置的通用解决方案，用于获取位置数据并跟踪指定精度水平的位置更改。
* 重大变化位置服务仅在设备的位置发生显着变化（例如500米或更长）时递送更新。

收集位置数据是一个功耗密集型操作。对于大多数应用程序，通常只需建立初始定位，然后定期获取更新即可。无论您的应用中位置数据的重要性如何，您都应该选择合适的位置服务，并明智地使用它，以避免耗尽设备的电池。 例如：

* 如果您的iOS应用程序必须保持监视位置，即使它在后台，使用标准位置服务，并指定 UIBackgroundModes 键的位置值，以继续在后台运行和接收位置更新。（在这种情况下，您还应该确保位置管理器的`pausesLocationUpdatesAutomatically`属性设置为YES，以帮助节省功耗。）可能需要此类型位置更新的应用程序示例包括健身或路口导航应用程序。
* 如果GPS级别的精确度对您的应用不重要，并且您不需要连续跟踪，则可以使用重大更改位置服务。至关重要的是，您必须正确使用重大更改位置服务，因为它会至少每15分钟唤醒系统和应用程序，即使没有发生位置更改，并且持续运行，直到您停止它。

### 确定位置服务是否可用

在某些情况下，位置服务可能不可用。 例如：

* 用户在设置应用或系统偏好中禁用了位置服务。
* 用户对一个 App 禁用了位置服务。
* 设备处于飞行模式，无法启动必要的硬件。

由于这些原因，建议您在尝试启动标准或重大更改位置服务之前，始终调用 CLLocationManager 的`locationServicesEnabled`类方法。 如果返回 NO，并且尝试启动位置服务，系统将提示用户确认是否应重新启用位置服务。 因为用户可能故意禁用位置服务，所以提示可能不受欢迎。

### 启动标准位置服务

标准位置服务是获取用户当前位置的最常用方式，由于它支持所有的iOS和OS X设备。在使用此服务之前，您可以指定位置数据的所需精确度和必须在报告新位置之前行驶距离。当您启动服务时，它使用指定的参数来确定要启用的硬件，然后向应用程序报告位置事件。由于此服务考虑了这些参数，因此它最适合需要对位置事件传递进行更细粒度控制的应用程序。 导航应用或任何需要高精度位置数据或常规更新流的应用需要标准位置服务的精度。 由于此服务通常需要将位置跟踪硬件启用更长的时间段，因此可能导致更高的功率使用。

要使用标准位置服务，创建一个 CLLocationManager 类的实例并配置其`desiredAccuracy`和`distanceFilter`属性。要开始接收位置通知，请为该对象分配一个委托并调用`startUpdatingLocation`方法。 当位置数据可用时，位置管理器通知其委托对象。如果已经交付位置更新，您还可以直接从 CLLocationManager 对象获取最新的位置数据，而无需等待要传送的新事件。要停止位置更新的传递，请调用位置管理器对象的`stopUpdatingLocation`方法。

清单1显示了配置位置管理器以供使用的示例方法。示例方法是一个类的一部分，将其位置管理器对象缓存在成员变量中以供稍后使用。（该类也符合 CLLocationManagerDelegate 协议，因此充当位置管理器的委托）。由于应用程序不需要精确的位置数据，因此它将位置服务配置为仅在用户移动至少半公里报告用户的一般区域。

```
- (void)startStandardUpdates
{
    // Create the location manager if this object does not already have one.
    if (nil == locationManager)
    locationManager = [[CLLocationManager alloc]init];

    locationManager.delegate = self;
    locationManager.desiredAccuracy = kCLLocationAccuracyKilometer;

    // Set a movement threshold for new events.
    locationManager.distanceFilter = 500; // meters

    [locationManager startUpdatingLocation];
}
```

从位置服务接收位置数据更新的代码见从服务接收位置数据。

### 启动重要更改位置服务

要使用重大更改位置服务，创建一个 CLLocationManager 类的实例，为其分配一个委托，并调用`startMonitoringSignificantLocationChanges`方法，如清单2所示。 当位置数据变得可用时，位置管理器通知其分配的委托对象。如果已经交付位置更新，您还可以直接从 CLLocationManager 对象获取最新的位置数据，而无需等待要传送的新事件。

> 注意：重大更改位置更新需要授权状态为`kCLAuthorizationStatusAuthorizedAlways`。

清单2

```
- (void)startSignificantChangeUpdates
{
    // Create the location manager if this object does not
    // already have one.
    if (nil == locationManager)
        locationManager = [[CLLocationManager alloc] init];

    locationManager.delegate = self;
    [locationManager startMonitoringSignificantLocationChanges];
}
```

与标准位置服务一样，位置数据被传递到委托对象，如“从服务接收位置数据”中所述。 要停止重要更改位置服务，请调用`stopMonitoringSignificantLocationChanges`方法。

如果您使大量更改位置服务正在运行，并且您的iOS应用程序随后被暂停或终止，则当新位置数据到达时，该服务会自动唤醒您的应用程序。 在唤醒时，应用程序将进入后台，您将获得少量时间（大约10秒）以手动重新启动位置服务并处理位置数据。（在后台，你必须手动地重新开启定位服务以便接收后续的位置更新，如“知道何时启动位置服务”所述）由于您的应用程序位于后台，因此它必须执行最少量的工作，并避免任何任务（例如查询网络），这可能会阻止它在分配的时间到期之前返回。如果没这么做，您的应用程序将被终止。如果一个iOS应用程序需要更多时间来处理位置数据，则可以使用 UIApplication 类的`beginBackgroundTaskWithName:expirationHandler:`方法请求更多后台执行时间。

> 注意：当用户全局或为您的应用停用后台应用刷新设置时，重大更改位置服务不会重新启动您的应用。此外，虽然后台应用程序刷新关闭，应用程序不会收到重大更改或区域监控事件，即使它在前台。

### 从服务接收位置数据

无论您使用标准或重大更改位置服务获取位置事件，您接收位置事件的方式都相同。从OS X v10.9和iOS 6开始，位置管理器会在事件可用时向其代理的`locationManager:didUpdateLocations:`方法报告事件。（在两个操作系统的早期版本中，位置管理器将事件报告给`locationManager:didUpdateToLocation:fromLocation:`方法。）如果检索事件时出错，位置管理器将调用其代理的`locationManager:didFailWithError:`方法。

清单3显示了接收位置事件的委托方法。由于位置管理器对象有时返回缓存的事件，因此建议您检查接收的任何位置事件的时间戳。（可能需要几秒钟才能获得粗略的位置修正，因此旧数据只是作为反映最后一个已知位置的一种方法。）在此示例中，该方法抛弃在假设下超过十五秒的任何事件，假定低于这个时间的事件是足够好的。如果要实施导航应用程序，您可能需要降低阈值。

清单3 处理收到的位置事件

```
// Delegate method from the CLLocationManagerDelegate protocol.
- (void)locationManager:(CLLocationManager *)manager
didUpdateLocations:(NSArray *)locations {
    // If it's a relatively recent event, turn off updates to save power.
    CLLocation* location = [locations lastObject];
    NSDate* eventDate = location.timestamp;
    NSTimeInterval howRecent = [eventDate timeIntervalSinceNow];
    if (abs(howRecent) < 15.0) {
    // If the event is recent, do something with it.
    NSLog(@"latitude %+.6f, longitude %+.6f\n",
        location.coordinate.latitude,
        location.coordinate.longitude);
    }
}
```

除了位置对象的时间戳之外，您还可以使用该对象报告的精度来确定是否要接受事件。当接收更准确的数据时，位置服务可以返回附加事件，其中精确度值反映了相应的改进。 丢弃对于你的应用减少在不太准确的无法有效使用的事件上的时间。

## 知道何时启动位置服务

使用位置服务的应用程序应该仅在需要的时候启动这些服务。除了少数例外，避免在启动时或在合理使用此类服务​​之前启动位置服务。否则，您可能会在用户的头脑中提出有关您的应用如何使用位置数据的问题。用户知道您的应用何时启动位置服务，因为系统会在您的应用启动服务时立即提示用户获取权限。等待，直到用户执行实际需要这些服务的任务有助于建立您的应用程序正确使用它们的信任。要在用户和应用程序之间建立信任，使用 Core Location 的应用程序必须在其 Info.plist 文件中包含 NSLocationAlwaysUsageDescription 或 NSLocationWhenInUseUsageDescription 键，并将该键的值设置为描述应用程序如何使用位置数据的字符串。如果调用`requestWhenInUseAuthorization`方法而不包括这些键之一，系统将忽略您的请求。

如果您正在监控地区或在应用程序中使用重大更改位置服务，则在某些情况下您必须在启动时启动位置服务。 使用这些服务的应用程序在中途可能被终止，并在新的位置事件到达时重新启动。 虽然应用程序本身重新启动，位置服务不会自动启动。当应用程序由于位置更新而重新启动时，传递给应用程序的启动选项字典：`willFinishLaunchingWithOptions:`或应用程序：`didFinishLaunchingWithOptions:`方法包含`UIApplicationLaunchOptionsLocationKey`键。 该键的存在表示新的位置数据正在等待传递到您的应用程序。要获取该数据，必须创建一个新的 CLLocationManager 对象，并重新启动在应用程序终止之前运行的位置服务。当您重新启动这些服务时，位置管理器将所有待处理的位置更新传递给其代理。

## 在后台获取位置事件（仅适用于iOS）

iOS支持将位置事件传递到已暂停或不再运行的应用。在后台中提供位置事件会支持应用程序的功能在没有它们的情况下受到影响，因此请将应用程序配置为只有在这样做时为用户提供切实的利益才能接收后台事件。例如，轮询导航应用程序需要始终跟踪用户的位置，并通知用户何时进行下一轮。如果您的应用程序可以使用其他方式（例如区域监控）来执行此操作，则应该这样做。

您有多个选项可用于获取背景位置事件，并且每个事件在功耗和位置精度方面都有优缺点。应尽可能使用重大更改位置服务（如启动重要更改位置服务中所述），该服务可以使用Wi-Fi来确定用户的位置并消耗最少的电量。但如果您的应用需要更高精度的位置数据，则可以将其配置为后台位置应用，并使用标准位置服务。

> 重要的：用户可以明确禁用任何应用的后台功能。如果用户在“设置”应用中停用后台应用刷新（针对所有应用或针对特定应用的全局禁用），则您的应用将无法在后台使用任何位置服务。您可以通过检查 UIApplication 类的`backgroundRefreshStatus`属性的值来确定您的应用程序是否可以在后台处理位置更新。

### 在后台使用标准位置服务

如果iOS应用提供需要连续位置更新的服务，则iOS应用可以在后台使用标准位置服务。因此，此功能最适合帮助用户进行导航和健身相关活动的应用程序。 要在应用程序中启用此功能，请在Xcode项目（位于项目的“功能”选项卡中）中启用“背景模式”功能，并启用位置更新模式。 您编写的用于启动和停止标准位置服务的代码未更改。表1列出了您可以启用后台模式功能的位置更新模式的API。

表1

API | Requires location updates in background mode | Can use location updates in background mode
:--:|:--:|:--:
startUpdateingLocation | Yes | Yes
CLVisit | No | Yes
CLRegion | No | Yes
startMonitoringSignificantLocationChanges | No | Yes
CLBeaconRegion | No | Yes, on a case-by-case basis

当后台位置应用程序处于前台、在后台运行或暂停时，系统会向其提供位置更新。在挂起的应用程序的情况下，系统唤醒应用程序，将更新传递到位置管理器的代理，然后尽快将应用程序返回到挂起状态。虽然它在后台运行，您的应用程序应该尽可能少的工作来处理新的位置数据。

当启用位置服务时，iOS必须保持位置硬件加电，以便它可以收集新数据。保持位置硬件运行会降低电池寿命，当您不需要生成的位置数据时，应始终在应用程序中停止位置服务。

如果您必须在后台使用位置服务，您可以通过在配置位置管理器对象时执行其他步骤来帮助Core位置为用户的设备维持良好的电池续航时间：

* 确保位置管理器的`pausesLocationUpdatesAutomatically`属性设置为YES。当此属性设置为YES时，每当有意义时，Core Location将暂停位置更新（并关闭位置硬件），例如，当用户不大可能移动时。（核心位置还会在无法获取位置定位时暂停更新。）
* 为位置管理器的`activityType`属性分配一个适当的值。 此属性中的值有助于位置管理器确定何时安全地暂停位置更新。对于提供逐向汽车导航的应用程序，将属性设置为`CLActivityTypeAutomotiveNavigation`会导致位置管理器仅在用户在一段时间内未移动显着距离时暂停事件。
* 调用`allowDeferredLocationUpdatesUntilTraveled:timeout:`方法尽可能延迟更新的交付，到稍后的时间，如“应用程序在后台时推迟位置更新”。

当位置管理器暂停位置更新时，它通过调用其其委托对象的`locationManagerDidPauseLocationUpdates:`方法来通知。当位置管理器恢复更新时，它调用代理的`locationManagerDidResumeLocationUpdates:`方法。 您可以使用这些委托方法来执行任务或调整应用程序的行为。 例如，当位置更新暂停时，您可以使用委派通知将数据保存到磁盘或完全停止位置更新。在逐向导航路线中间的导航应用可能提示用户并询问是否应暂时禁用导航。

> 注意：如果您的应用程序由用户或系统终止，系统不会在新位置更新到达时自动重新启动应用程序。用户必须在恢复位置更新的递送之前明确重新启动您的应用。让应用程式自动重新启动的唯一方法是使用区域监控或重大变更位置服务。

> 但是，如果用户全局或专门为您的应用停用后台应用刷新设置，系统不会针对任何位置事件（包括重大更改或区域监控事件）重新启动您的应用。此外，当后台应用刷新关闭，您的应用程序将不会收到重大更改或区域监控事件，即使它在前台。当用户为您的应用重新启用后台应用刷新时，Core Location 将恢复所有后台服务，包括任何先前注册的区域。

### 推迟位置更新，而您的应用程序在后台

在iOS 6及更高版本中，您可以在应用处于后台时延迟位置更新的递送。建议您在应用程序稍后处理位置数据时没有任何影响时，可以使用此功能。 例如，跟踪用户在远足路径上的位置的健身应用可以推迟更新，直到用户爬上一定距离或直到经过一定量的时间，然后一次处理更新。推迟更新可通过让应用程序长时间休眠来节省功耗。由于推迟位置更新需要目标设备上存在GPS硬件，因此请确保调用 CLLocationManager 类的`deferredLocationUpdatesAvailable`类方法来确定设备是否支持延迟位置更新。

调用 CLLocationManager 类的`allowDeferredLocationUpdatesUntilTraveled:timeout:`方法以开始推迟位置更新。如清单4所示，调用此方法的常见地方是位置管理器委托对象的`locationManager:didUpdateLocations:`方法。此方法显示一个示例徒步旅行应用程序可能推迟位置更新，直到用户爬到最小距离。 为了防止自己调用`allowDeferredLocationUpdatesUntilTraveled:timeout:`方法多次，代理使用内部属性来跟踪更新是否已被延迟。 它在此方法中将属性设置为YES，并在延迟更新结束时将其设置为NO。

清单4 推迟位置更新

```
// Delegate method from the CLLocationManagerDelegate protocol.
- (void)locationManager:(CLLocationManager *)manager
didUpdateLocations:(NSArray *)locations {
    // Add the new locations to the hike
    [self.hike addLocations:locations];

    // Defer updates until the user hikes a certain distance
    // or when a certain amount of time has passed.
    if (!self.deferringUpdates) {
        CLLocationDistance distance = self.hike.goal - self.hike.distance;
        NSTimeInterval time = [self.nextAudible timeIntervalSinceNow];
        [locationManager allowDeferredLocationUpdatesUntilTraveled:distance
            timeout:time];
        self.deferringUpdates = YES;
    }
}
```

当满足在`allowDeferredLocationUpdatesUntilTraveled:timeout:`方法中指定的条件时，位置管理器调用其委托对象的`locationManager:didFinishDeferredUpdatesWithError:`方法以通知您已停止推迟位置更新的传递。每次您的应用程序调用`allowDeferredLocationUpdatesUntilTraveled:timeout:`方法时，位置管理器都会调用此委派方法一次。延迟更新结束后，位置管理器继续将任何位置更新传送到您的代理的`locationManager:didUpdateLocations:`方法。

您可以通过调用 CLLocationManager 类的`disallowDeferredLocationUpdates`方法显式停止位置更新的延迟 当您调用此方法或使用`stopUpdatingLocation`方法停止位置更新时，位置管理器调用您的委托的`locationManager:didFinishDeferredUpdatesWithError:`方法，让您知道延迟更新确实已停止。

如果位置管理器遇到错误并且无法推迟位置更新，则可以在实现`locationManager:didFinishDeferredUpdatesWithError:delegate`方法时访问错误的原因。 有关可能返回的可能错误的列表，以及可以解决的问题（参见 Core Location Constants Reference 中的 CLError 常量）。
