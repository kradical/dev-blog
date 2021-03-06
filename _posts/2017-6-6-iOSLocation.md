---
layout: post
title: iOS Locations
---

Everybody has a highly accurate tracking device on themselves at almost all times. To take advantage of this many organizations are prioritizing location data, and location based applications. The application I am currently working on attempts to make business expenses simple and less time consuming by using location data to track possible expenses and prefill sensible values. This post will walk through some simple use cases and have example code for how to do locations in swift (3.1) on iOS. I will also attempt to clarify things and present gotcha's that I have encountered.

I try to separate out location functionality from my app delegate and the rest of my app. A stripped down example of the class I would use:

```swift
import UIKit
import CoreLocation

class LocationManager: NSObject, CLLocationManagerDelegate {
    static let shared = LocationManager()
    private override init() {}

    private let locationManager = CLLocationManager()
    var location: CLLocation? { return self.locationManager.location }

    func initLocationManager() {
        self.locationManager.delegate = self

        self.locationManager.requestAlwaysAuthorization()
        self.startUpdatingLocation()
    }

    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        print("didUpdateLocations \(locations)")
    }

    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
        if status == .authorizedAlways {
            self.initLocationManager()
        } else {
            self.locationManager.requestAlwaysAuthorization()
        }
    }

    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("locationManager error: \(error)")
    }

    func stopUpdatingLocation() {
        self.locationManager.stopUpdatingLocation()
    }

    func startUpdatingLocation() {
        self.locationManager.startUpdatingLocation()
    }
}
```

This skeleton allows you to fill in more complex behaviour without directly adding more bloat to your AppDelegate.

One of the biggest problems I encountered was background locations. I think this is such a tough problem because fundamentally GPS uses a lot of battery. There is not a good way to maintain good battery life and good accuracy. You have to make a tradeoff somewhere.

My overall strategy after weeks of reading and testing was to backoff the accuracy when relatively still and then after moving ~200m to turn on high accuracy again. This works very well in wifi rich locations where the 100-500m accuracy can be determined easily. One problem with this approach is it keeps the app on in the background all the time. I think it may be worth experimenting with periodically waking up in the background to sample location / location change.

No matter what approach you take you are ultimately at the mercy of the almighty operating system. Preparing for unexpected termination is one of the hardest problems with this sort of application.
