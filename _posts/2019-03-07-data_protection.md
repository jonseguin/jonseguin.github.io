---
layout: post
title: "iOS Data protection"
date: 2019-03-07 11:07:00 +0100
categories: [ios]
tags: [ios]
---

I recently had to search what kind of security exists and can be made to secure and restrict access to files saved on the files system of an app. I have found that the documentation can lead to bad understanding, so I decided to put here the result of my research.

* The device password is the encryption key. So it is the _limit_ of the protection : who knows the device password can potentially access all the files of all the apps.
* When we connect a device to a Mac we gave the trust to, the data protection is totally disabled while connected. The _trust_ is necessary to give if you want to plug a device for iTunes access or to build your Xcode project on it. 
* The data protection is not compatible with the files we want to synchronize with iCloud. There is also some interesting security matter about this functionality, that I will talk about in the next paragraph.
* There is 4 levels of data protection.
* When an user enable a password on its device, all apps has the **Complete until first user authentication** protection level since iOS 7. This protection makes the files to be encrypted until the first time the user authenticate after the user reboot or shut down his device.
* We can enable a custom level for our app by enabling "Data Protection" entitlement. By default, the level will be set to **Complete**, which make the files to be accessible only when the device is unlocked, but we can set it to the value we want on the entitlement file.
* There is two more protection levels :
	* **No protection** which make the files always accessibles, and
	* **Complete unless open** which is special case where you have to handle an access to a file already open after the device has been locked.
* An interesting functionality is taht we can setup manually the protection level for each file we save on the files system, no matter what has been set on the entitlements. A good practice is to setup the level we want to a directory, that therefore will be apply to all the files on it.

Here is an exemple where we set the **complete** protection level to a directory named _SecureDirectory_ :

```
let secureDocumentDirectory = try FileManager.default.url(
	for: .documentDirectory, 
	in: .userDomainMask, 
	appropriateFor: nil, 
	create: false
).appendingPathComponent("SecureDirectory")

try FileManager.default.createDirectory(
	at: secureDocumentDirectory, 
	withIntermediateDirectories: false, 
	attributes: [
		FileAttributeKey.protectionKey: 
		FileProtectionType.complete
	]
)
```

* For the **Complete** and **Complete unless open** protection levels, you will have to handle the cases when files are accessed during backgrounds operations. You may need to implement `applicationProtectedDataWillBecomeUnavailable(_:)` and `applicationProtectedDataDidBecomeAvailable(_:)` methods on your app delegate. 

Here is some interesting links I use for my research on this matter :

- [The Apple official document about the Data Protection][data_protection_doc] 
- [An official Apple document](https://www.apple.com/business/site/docs/iOS_Security_Guide.pdf) on everything that concerns security on a iOS device.
- [An old but detailed blog post](http://www.ilounge.com/index.php/articles/comments/ios-encryption-and-data-protection/#1z6VYpREIyuyxLC2.99) about this subject
- [WWDC 2011 - Securing Application Data](https://developer.apple.com/videos/play/wwdc2011/208)

[data_protection_doc]: https://developer.apple.com/documentation/uikit/core_app/protecting_the_user_s_privacy/encrypting_your_app_s_files


### Files access and backup

The iOS backup functionality can be used with iTunes and other third party softwares, via iCloud or manually.
By default, all the files of an app are automatically backed up, except the bundle itself, the cache directory and the temp directory.
All the others files are included in the backup, and maybe you don't want that to be possible (it could be a database file or some pictures files you don't want to be spread).
To do that, all you have to do is to specify the resource value `isExcludedFromBackupKey` to the NSURL of the file or the directory.

```
try (secureDocumentDirectory as NSURL)
	.setResourceValue(true, forKey: .isExcludedFromBackupKey)
```
You can look for more information on the [Apple Documentation about the data storage](https://developer.apple.com/icloud/documentation/data-storage/index.html).

### Conclusion

Here we discussed about two important points when we want to _restrict_ access to the files used by our app. Despite the fact that these point are not well known and the documentation could be confusing, I think it is important for all iOS developers to take this into account before releasing an app that can have sensitive data stored in the file system. I hope what I wrote here can be of any help.