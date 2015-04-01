# p2pkit.io Quickstart

## Table of Contents

**[Download](#download)**  

### Download

Download the P2PKit library here: http://www.uepaa.ch

### Signup

Request your personal application key here: http://www.uepaa.ch

### Initialization

Import the P2PKit header

```objc
#import <P2PKit/P2PKit.h>
```

Initialize P2PKit with your personal application key

```objc
[PPKController enableWithConfiguration:@"<YOUR APPLICATION KEY>" observer:self];
```

Implement `PPKControllerDelegate` protocol to start desired discovery- and messaging features when P2PKit is ready

```objc
-(void)PPKControllerInitialized {
	[PPKController startP2PDiscovery];
	[PPKController startGeoDiscovery];
	[PPKController startOnlineMessaging];
	
	NSLog(@"My ID is %@ - please send me a message", [PPKController userID]);
}
```

### P2P Discovery

Add BLE (Bluetooth low energy) permissions to your `Info.plist` file
```
<key>UIBackgroundModes</key>
<array>
    <string>bluetooth-central</string>
    <string>bluetooth-peripheral</string>
</array>
```

Implement `PPKControllerDelegate` protocol to receive P2P discovery events

```objc
-(void)p2pPeerDiscovered:(NSString*)peerID {
	NSLog(@"%@ is here", peerID);
	destination_ = peerID;
}

-(void)p2pPeerLost:(NSString*)peerID {
	NSLog(@"%@ is no longer here", peerID);
	destination_ = nil;
}
```

### Online Messaging

Implement `PPKControllerDelegate` protocol to receive online messages

```objc
-(void)messageReceived:(NSData*)messageBody header:(NSString*)messageHeader 
	   from:(NSString*)peerID 
{
	NSLog(@"Message received from %@: %@", peerID,
		[[NSString alloc] initWithData:messageBody encoding:NSUTF8StringEncoding]);
}
```

Send online messages to discovered peers

```objc
if (destination_) {
	[PPKController sendMessage:[@"Hello World" dataUsingEncoding:NSUTF8StringEncoding] 
				   withHeader:@"SimpleChatMessage" to:destination_];
}
```

### GEO Discovery

Add location permissions to your `Info.plist` file
```
<key>NSLocationAlwaysUsageDescription</key>
<string>So that you can continuously discover, be discovered and receive notifications when your people of interest are nearby</string>

<key>NSLocationWhenInUseUsageDescription</key>
<string>So that you can discover and be discovered by others</string>

<key>UIBackgroundModes</key>
<array>
	<string>location</string>
</array>
```

Use `CLLocationManager` and implement `CLLocationManagerDelegate` protocol to report your GEO location

```objc
#import <CoreLocation/CoreLocation.h>

#define GPS_DEFAULT_DESIRED_ACCURACY 200
#define GPS_DEFAULT_DISTANCE_FILTER 200

-(void)startLocationUpdates {
    locMgr_ = [CLLocationManager new];
    [locMgr_ setDelegate:self];
    [locMgr_ setDesiredAccuracy:GPS_DEFAULT_DESIRED_ACCURACY];
    [locMgr_ setDistanceFilter:GPS_DEFAULT_DISTANCE_FILTER];
    [locMgr_ setPausesLocationUpdatesAutomatically:NO];
    
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 80000
    if ([locMgr_ respondsToSelector:@selector(requestAlwaysAuthorization)]) {
        [locMgr_ requestAlwaysAuthorization];
    }
#endif
    
    [locMgr_  startUpdatingLocation];
}

- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations {
    if (locations.count < 1) {
        return;
    }
    
    CLLocation *loc = [locations objectAtIndex:0];
    if (!loc) {
        return;
    }
    
    [PPKController updateUserLocation:loc];
}
```

Implement `PPKControllerDelegate` protocol to report your GEO location when you have internet connectivity

```objc
-(void)geoDiscoveryStateChanged:(PPKGeoDiscoveryState)state {
 	switch (state) {
    	case PPKGeoDiscoveryRunning:
        	[self startLocationUpdates];
            break;
        case PPKGeoDiscoverySuspended:
        case PPKGeoDiscoveryStopped:
            [self stopLocationUpdates];
            break;
	}
}
```

Implement `PPKControllerDelegate` protocol to receive GEO discovery events

```objc
-(void)geoPeerDiscovered:(NSString*)peerID {
	NSLog(@"%@ is around", peerID);
 	destination_ = peerID;
}

-(void)geoPeerLost:(NSString*)peerID {
	NSLog(@"%@ is no longer around", peerID);
	destination_ = nil;
}
```