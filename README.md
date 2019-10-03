# Tapgo Api
## 獲取USB儲存裝置資訊

### TGMassStorageDevice(<UsbMassStorageDevice>).getPartitions()

### Returns:

ArrayList, UsbMassStorageDevice的磁區資訊

### type:ArrayList

[[String VolumeLabel, int PartitionType, Long Capacity, Long OccupiedSpace, Long FreeSpace, int ChunkSize]]

### PartitionType:

2 : FAT32

6 : ExFAT

### Ex:
[["PartitionNo1", 2, 10000, 5000, 5000, 32768],["PartitionEx", 6, 10000, 6000, 4000, 532]]


#### Java

```java
UsbMassStorageDevice[] devices = UsbMassStorageDevice.getMassStorageDevices(this /* Context or Activity */);

//通常Android只有一個 Usb 傳輸插槽
final int currentDevice = 0;
//before interacting with a device you need to call init()!
devices[currentDevice].init()
//印出UsbMassStorageDevice的磁區資訊
ArrayList partitions = new TGMassStorageDevice(devices[currentDevice]).getPartitions();
Log.e(partitions.toString())

//磁區index
Int currentPartition = 0;
FileSystem currentFs = devices[currentDevice].getPartitions().get(0).getFileSystem();

```
## 檔案操作

### TGFileOperation(<FileSystem>, <UsbFile>)

### throw:
IOException

### TGFileOperation(<FileSystem>, UsbFile rootDir).getFile(<String filePathName>)
    

### Returns:

UsbFile, 絕對路徑的UseFile檔案

### throws:

TGFileOperationException : 當filePathName檔案不存在時拋出

#### Java

```java
TGFileOperation fileOperation = new TGFileOperation(currentFs, currentFs.getRootDirectory());
UsbFile file = fileOperation.getFile("/dir1/subdir2/picture.jpg");

```

### TGFileOperation(<FileSystem>, UsbFile rootDir).getFileListIn(<String dirPathname>)
    

### Returns:

ArrayList, 目錄下所有檔案, 暫不包含子資料夾

### throws:

TGFileOperationException : 當找不到目錄時拋出

#### Java

```java
TGFileOperation fileOperation = new TGFileOperation(currentFs, currentFs.getRootDirectory());
ArrayList arrayList = fileOperation.getFileListIn("/dir/subdir");

```

### TGFileOperation(<FileSystem>, UsbFile rootDir).fileCopyToPhone(<Context>, <ArrayList<UsbFile>>, <String toPath>);

將單/多筆UsbFile寫入指定路徑, 單一檔案寫入完畢後會開啟

### throws:

TGFileOperationException : 

當找不到路徑時拋出


#### Java

```java
TGFileOperation fileOperation = new TGFileOperation(currentFs, currentFs.getRootDirectory());
ArrayList arrayList = fileOperation.getFileListIn("/123");

fileOperation.fileCopyToPhone(MainActivity.this, arrayList,
    Environment.getExternalStorageDirectory().getAbsolutePath());

```



libaums
=======
[![Javadocs](https://www.javadoc.io/badge/com.github.mjdev/libaums.svg)](https://www.javadoc.io/doc/com.github.mjdev/libaums)
[ ![Build Status](https://travis-ci.org/magnusja/libaums.svg?branch=develop)](https://travis-ci.org/magnusja/libaums)[ ![codecov](https://codecov.io/gh/magnusja/libaums/branch/develop/graph/badge.svg)](https://codecov.io/gh/magnusja/libaums)[ ![Download](https://api.bintray.com/packages/mjdev/maven/libaums/images/download.svg) ](https://bintray.com/mjdev/maven/libaums/_latestVersion)
[ ![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/libaums)

A library to access USB mass storage devices (pen drives, external HDDs, card readers) using the Android USB Host API. Currently it supports the SCSI command set and the FAT32 file system.

# User survey 2019

I am currently trying to understand who and how people use this library as well as where is most room for future improvement. To do so I created a really short survey (no more than 3 minutes) and I would be really happy if people would take part: https://forms.gle/WBkXWpPVRYxrPhrs9

Thank you!

## How to use

### Install

The library can be included into your project like this:

```ruby
compile 'com.github.mjdev:libaums:0.7.1'
```

If you need the HTTP or the storage provider module:

```ruby
compile 'com.github.mjdev:libaums-httpserver:0.5.3'
compile 'com.github.mjdev:libaums-storageprovider:0.5.1'
```

### Basics
#### Query available mass storage devices

#### Java

```java
UsbMassStorageDevice[] devices = UsbMassStorageDevice.getMassStorageDevices(this /* Context or Activity */);

for(UsbMassStorageDevice device: devices) {
    
    // before interacting with a device you need to call init()!
    device.init();
    
    // Only uses the first partition on the device
    FileSystem currentFs = device.getPartitions().get(0).getFileSystem();
    Log.d(TAG, "Capacity: " + currentFs.getCapacity());
    Log.d(TAG, "Occupied Space: " + currentFs.getOccupiedSpace());
    Log.d(TAG, "Free Space: " + currentFs.getFreeSpace());
    Log.d(TAG, "Chunk size: " + currentFs.getChunkSize());
}
```

#### Kotlin

```kotlin
val devices = UsbMassStorageDevice.getMassStorageDevices(this /* Context or Activity */)

for (device in devices) {

    // before interacting with a device you need to call init()!
    device.init()

    // Only uses the first partition on the device
    val currentFs = device.partitions[0].fileSystem
    Log.d(TAG, "Capacity: " + currentFs.capacity)
    Log.d(TAG, "Occupied Space: " + currentFs.occupiedSpace)
    Log.d(TAG, "Free Space: " + currentFs.freeSpace)
    Log.d(TAG, "Chunk size: " + currentFs.chunkSize)
}
```

#### Permissions

Your app needs to get permission from the user at run time to be able to communicate the device. From a `UsbMassStorageDevice` you can get the underlying `android.usb.UsbDevice` to do so.

#### Java
```java
PendingIntent permissionIntent = PendingIntent.getBroadcast(this, 0, new Intent(ACTION_USB_PERMISSION), 0);
usbManager.requestPermission(device.getUsbDevice(), permissionIntent);
````

#### Kotlin

```kotlin
val permissionIntent = PendingIntent.getBroadcast(this, 0, Intent(ACTION_USB_PERMISSION), 0);
usbManager.requestPermission(device.usDevice, permissionIntent);
```

For more information regarding permissions please check out the Android documentation: https://developer.android.com/guide/topics/connectivity/usb/host.html#permission-d

#### Working with files and folders

##### Java

```java
UsbFile root = currentFs.getRootDirectory();

UsbFile[] files = root.listFiles();
for(UsbFile file: files) {
    Log.d(TAG, file.getName());
    if(file.isDirectory()) {
        Log.d(TAG, file.getLength());
    }
}

UsbFile newDir = root.createDirectory("foo");
UsbFile file = newDir.createFile("bar.txt");

// write to a file
OutputStream os = new UsbFileOutputStream(file);

os.write("hello".getBytes());
os.close();

// read from a file
InputStream is = new UsbFileInputStream(file);
byte[] buffer = new byte[currentFs.getChunkSize()];
is.read(buffer);
```

##### Kotlin
```kotlin
val root = currentFs.rootDirectory

val files = root.listFiles()
for (file in files) {
    Log.d(TAG, file.name)
    if (file.isDirectory) {
        Log.d(TAG, file.length)
    }
}

val newDir = root.createDirectory("foo")
val file = newDir.createFile("bar.txt")

// write to a file
val os = UsbFileOutputStream(file)

os.write("hello".toByteArray())
os.close()

// read from a file
val ins = UsbFileInputStream(file)
val buffer = ByteArray(currentFs.chunkSize)
ins.read(buffer)
```

#### Using buffered streams for more efficency

```java
OutputStream os = UsbFileStreamFactory.createBufferedOutputStream(file, currentFs);
InputStream is = UsbFileStreamFactory.createBufferedInputStream(file, currentFs);
```

#### Cleaning up

```java
// Don't forget to call UsbMassStorageDevice.close() when you are finished

device.close();
```

#### Provide access to external apps

Usually third party apps do not have access to the files on a mass storage device if the Android system does mount (this is usually supported on newer devices, back in 2014 there was no support for that) the device or this app integrates this library itself. To solve this issue there are two additional modules to provide access to other app. One uses the Storage Access Framework feature of Android (API level >= 19) and the other one spins up an HTTP server to allow downloading or streaming of videos or images for instance.


### HTTP server
[![Javadocs](https://www.javadoc.io/badge/com.github.mjdev/libaums-httpserver.svg)](https://www.javadoc.io/doc/com.github.mjdev/libaums-httpserver)

libaums currently supports two different HTTP server libraries.

1. [NanoHTTPD](https://github.com/NanoHttpd/nanohttpd)
2. [AsyncHttpServer](https://github.com/koush/AndroidAsync/blob/master/AndroidAsync/src/com/koushikdutta/async/http/server/AsyncHttpServer.java)

You can spin up a server pretty easy, you just have to decide for a HTTP server implementation. If you do not have special requirements, you can just go for one, it should not make much of a difference.

#### Java

```java
UsbFile file = ... // can be directory or file

HttpServer server = AsyncHttpServer(8000); // port 8000
// or
HttpServer server = NanoHttpdServer(8000); // port 8000

UsbFileHttpServer fileServer = new UsbFileHttpServer(file, server);
fileServer.start();
```

#### Kotlin

```kotlin
val file: UsbFile
// can be directory or file

val server = AsyncHttpServer(8000) // port 8000
// or
val server = NanoHttpdServer(8000) // port 8000

val fileServer = UsbFileHttpServer(file, server)
fileServer.start()
```

The file you provide can either be an actual file or a directory:

1. File: Accessible either via "/" or "/FILE_NAME"
2. Directory: All files in this directory und sub directories are accessable via their names. Directory listing is not supported!

If you want to be able to access these files when your app is in background, you should implement a service for that. There is an example available in the `httpserver` module. You can use it, but should subclass it or create your own to adapt it to your needs.

#### Java

```java
private UsbFileHttpServerService serverService;

ServiceConnection serviceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        Log.d(TAG, "on service connected " + name);
        UsbFileHttpServerService.ServiceBinder binder = (UsbFileHttpServerService.ServiceBinder) service;
        serverService = binder.getService();
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        Log.d(TAG, "on service disconnected " + name);
        serverService = null;
    }
};

@Override
protected void onCreate(Bundle savedInstanceState) {
...
    serviceIntent = new Intent(this, UsbFileHttpServerService.class);
...
}

 @Override
protected void onStart() {
    super.onStart();

    startService(serviceIntent);
    bindService(serviceIntent, serviceConnection, Context.BIND_AUTO_CREATE);
}

private void startHttpServer(final UsbFile file) {
...
    serverService.startServer(file, new AsyncHttpServer(8000));
...
}
```

#### Kotlin

```kotlin
private var serverService: UsbFileHttpServerService? = null

internal var serviceConnection: ServiceConnection = object : ServiceConnection() {
    override fun onServiceConnected(name: ComponentName, service: IBinder) {
        Log.d(TAG, "on service connected $name")
        val binder = service as UsbFileHttpServerService.ServiceBinder
        serverService = binder.getService()
    }

    override fun onServiceDisconnected(name: ComponentName) {
        Log.d(TAG, "on service disconnected $name")
        serverService = null
    }
}

override protected fun onCreate(savedInstanceState: Bundle) {
    serviceIntent = Intent(this, UsbFileHttpServerService::class.java)
}

override protected fun onStart() {
    super.onStart()

    startService(serviceIntent)
    bindService(serviceIntent, serviceConnection, Context.BIND_AUTO_CREATE)
}
```

See the example app for additional details on that.


### Storage Access Framework
[![Javadocs](https://www.javadoc.io/badge/com.github.mjdev/libaums-storageprovider.svg)](https://www.javadoc.io/doc/com.github.mjdev/libaums-storageprovider)

To learn more about this visit: https://developer.android.com/guide/topics/providers/document-provider.html

To integrate this module in your app the only thing you have to do is add the definition in your AndroidManifest.xml.

```xml
<provider
    android:name="com.github.mjdev.libaums.storageprovider.UsbDocumentProvider"
    android:authorities="com.github.mjdev.libaums.storageprovider.documents"
    android:exported="true"
    android:grantUriPermissions="true"
    android:permission="android.permission.MANAGE_DOCUMENTS"
    android:enabled="@bool/isAtLeastKitKat">
    <intent-filter>
        <action android:name="android.content.action.DOCUMENTS_PROVIDER" />
    </intent-filter>
</provider>
```

After that apps using the Storage Access Framework will be able to access the files of the USB mass storage device.


#### Hints

1. In the `app/` directory you can find an example application using the library.
2. When copying a file always set the length via `UsbFile.setLength(long)` first. Otherwise the ClusterChain has to be increased for every call to write. This is very inefficent.
3. Always use `FileSystem.getChunkSize()` bytes as buffer size, because this alignes with the block sizes drives are using. Everything else is also most likeley a decrease in performance.
4. A good idea is to wrap the UsbFileInputStream/UsbFileOutputStream into BufferedInputStream/BufferedOutputStream. Also see `UsbFileStreamFactory`.

##### Thesis

Libaums - Library to access USB Mass Storage Devices  
License: Apache 2.0 (see license.txt for details)
Author: Magnus Jahnen, jahnen at in.tum.de  
Advisor: Nils Kannengießer, nils.kannengiesser at tum.de  
Supervisor: Prof. Uwe Baumgarten, baumgaru at in.tum.de  


Technische Universität München (TUM)  
Lehrstuhl/Fachgebiet für Betriebssysteme  
www.os.in.tum.de  

The library was developed by Mr. Jahnen as part of his bachelor's thesis in 2014. It's a sub-topic of the research topic "Secure Copy Protection for Mobile Apps" by Mr. Kannengießer. The full thesis document can be downloaded [here](https://www.os.in.tum.de/fileadmin/w00bdp/www/Lehre/Abschlussarbeiten/Jahnen-thesis.pdf).

We would appreciate an information email, when you plan to use the library in your projects.
