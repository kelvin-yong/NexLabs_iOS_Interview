#iOS Interview Answer Guide for NexLabs#


##A. Language##

###1. Ivars, Properties, Class Extensions and Categories###

8 ivars: `_machineLastName` isn't

- `serviceDate`
- `_createdDate`
- `_machineFirstName`
- `_machineLastName`
- `_rentalRatePerHour`
- `totalRunningTime`
- `isOn`
- `_turnedOnDate`

===

Class extension: Encapsulation, do not expose unnecessary methods/properties in header files.

===
Multi-threading: `atomic`, `non-atomic`    
Read/write: `readwrite`, `readonly`  
Memory: `strong` (`retain`), `weak`, `copy`, `unsafe_unretained` (`assign`)  
 
===

Properties: can write your own getter / setter code that checks for logic. Example you could have a `- (void)setRentalRatePerHour:(float)rentalRatePerHour` to ensure the rate is more than 0.

===

Legacy code. Previous versions of Xcode required that you be explicit. Now you don't have to

===

Class category - usually to break up code in manageable chunks, or to 'extend' objects for which you have no code for.

In a class category, you can't use synthesize to generate the implementation of your properties

===

Strong if it is not added to subview of the ViewController's view

===
 
##B. Basics##

###1. Collections: Array and Dictionary###

Array: Ordered, indexed, allows duplicates  
Dictionary: Unordered, key-value entries, unique keys, hash lookup  
NSSet: Unordered, no duplicates, hash lookup  

Consider Big(0) complexity  

Immutable collections have benefits:

- Thread safety- Memory and speed optimizations

===

Part A: Doesn't compile. Non-mutable array doesn't have remove object.

Part B: Run time exception - Not suppose to mutate/modify object while being enumerated in fast enumeration.

===
Part C: Perfectly fine.

    (
        {name = xxx; },
        {name = xxx; },
        {name = ellen; },
        {name = rose; },
        {name = xxx; }
    )

===

###2. NSUserDefaults and Settings.bundle###


- setObject:forKey:
- removeObjectForKey:
- synchronize
 
====

`PrefHTTPSDisabled` with default value as `YES`  
You are testing with HTTP now, so HTTPSDisable should be YES.   
Also if the key is missing in production version, it would then be defaulted to NO, and you really want to use HTTPS in production. Minimises code change.

===

###3. View and Application Lifecycle###

- `viewDidLoad`
- `viewDidUnload`(to discuss)
- `viewWill/DidAppear`
- `viewWill/DidDisappear`

===

- `application:didFinishLaunchingWithOptions`
- `applicationDidBecomeActive`
- `applicationWillResignActive` 
- `applicationWillEnterForeground`
- `applicationDidEnterBackground` 

===

##C. Experience##

###1. Xcode###

- Shortcuts like Toogle Editor/Assistant Editor, Show/Hide Navigator, Show/Hide Utilities, Quick Open, Move Focus, Move to Jump Bar

- Directives like `#pragma mark -`, `#warning`, `// TODO:`

- Build scripts

- Any other things: e.g. messing with Prefix.pch


===

###2. Adhoc build###

Possible causes

- Signed with wrong profile
- Profile does not contain the device id (Standard distribution, not for Enterprise Distribution)
- Debug entitlements left in

Remedy

- Unzip the IPA file
- Check manually... or `codesign -d --entitlements  - ./Payload/SingTelChat.app`


===

###3. Debugging and Instrumenting###

First step: Add exception breakpoint

====

Profiling / Instrumentation: Activity monitor, Core 

====
Profiling / Instrumentation: Allocations, Leaks 

Static Analysis

====

###4. Third party libraries###

Can mention any library on Github: AFNetworking, MagicalRecord etc. To get a sense of what the candidate has been working with.

===

##D. Specific Knowledge##

###Asynchronous code, NSOperations, Grand Central Dispatch ###
`_spinner`, assumed to be an instance of `UIActivityIndicatorView`, does not animate when `downloadLargePhotoAndApplyFilter`, a synchronous operation, is blocking on the main thread.

Why is it bad? When `downloadLargePhotoAndApplyFilter` is running, the user cannot interact with the UI. There is also no way of cancelling the process.

Bonus for mentioning that if this type is run at `application:didFinishLaunchingWithOptions:`, the iOS watchdog would terminate the app for taking too long to launch. Prompt candidate if he does not mention this.

Extra bonus for mentioning the error code produced would be `0x8badf00d` i.e. ate bad food.

===

Candidate should suggest this [pattern](https://gist.github.com/kelvin-yong/333cfc21eb70db3e9468)

    [_spinner startAnimating];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
        UIImage *image = [self downloadLargePhotoAndApplyFilter];
        dispatch_async(dispatch_get_main_queue(), ^{
            self.myImageView.image = image;
            [_spinner stopAnimating];
        });
    });

=== 

Candidate can mention any of the following:

- dispatch_async, dispatch_sync, dispatch_after
- main queue, several global concurrent queues, custom serial and concurrent queues
- dispatch_once
- dispatch_semaphore_t / dispatch_semaphore_wait / dispatch_semaphore_signal 
- dispatch_group_t / dispatch_group_enter / dispatch_group_leave / dispatch_group_wait

===


###2. CoreData###

Minimally, I expect NSManagedObjectContext, NSManagedObject, NSPersistentStoreCoordinator and their relationship.

===

Transient properties are propertiesthat are not persisted to the object store. They are calculated at runtime, usually on the basis of other property values. 

Compute fullname (based on first and last name), or plaintext (based on ciphertext, salt, key)

===

###3. Apple Push Notification Service###

A discussion on feedback service, registration, payload sizes, uses.

Specific scenario: discuss XMPP Server push notification for point to point message.

===

###4. What would you do###

It will be wrong to say to update the timestamp on viewDidAppear/viewWillAppear.

One interesting way would be to schedule timer to trigger every 24 hours. But this will not work for daylights savings or timezones.

Best / Correct is to register for `UIApplicationSignificantTimeChangeNotification`.

===
Not looking at exact code here. But the candidate should know how such things work. 

For example, if it is a simple text in UITableView section header, you would return a string in `UITableViewDataSource |tableView:titleForHeaderInSection:`. 

For a complicated section header, you would return a view in `UITableViewDelegate | tableView:viewForHeaderInSection:`

In this case, instead of `self.title = @"Chats"`, you would use `self.navigationItem.titleView = someView`.

===

You could check the availability of the class (NSClassFromString) or the availability of the method (respondsToSelector).

Or even the system version. In certain cases, you may need to weakly linked the libraries.

===

`performSelector` or using `NSInvocation` if the method needs more than 2 parameters

===