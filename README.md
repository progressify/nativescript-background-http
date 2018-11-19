# Background Upload NativeScript plugin [![Build Status](https://travis-ci.org/NativeScript/nativescript-background-http.svg?branch=master)](https://travis-ci.org/NativeScript/nativescript-background-http)

A cross platform plugin for [the NativeScript framework](http://www.nativescript.org), that provides background upload for iOS and Android.

[There is a stock NativeScript `http` module that can handle GET/POST requests that work with strings and JSONs](http://docs.nativescript.org/ApiReference/http/HOW-TO). It however comes short in features when it comes to really large files.

The plugin uses [NSURLSession with background session configuration for iOS](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSURLSessionConfiguration_class/index.html#//apple_ref/occ/clm/NSURLSessionConfiguration/backgroundSessionConfigurationWithIdentifier:); and the [alexbbb/android-upload-service](https://github.com/alexbbb/android-upload-service) library for Android.

## Installation

```bash
tns plugin add nativescript-background-http
```

## Usage

The below attached code snippets demonstrate how to use `nativescript-background-http` to upload single or multiple files.

### Uploading files

Sample code for configuring the upload session. Each session must have a unique `id`, but it can have multiple tasks running simultaneously. The `id` is passed as a parameter when creating the session (the `image-upload` string in the code bellow):

```JavaScript

// file path and url
var file =  "/some/local/file/path/and/file/name.jpg";
var url = "https://some.remote.service.com/path";
var name = file.substr(file.lastIndexOf("/") + 1);

// upload configuration
var bghttp = require("nativescript-background-http");
var session = bghttp.session("image-upload");
var request = {
        url: url,
        method: "POST",
        headers: {
            "Content-Type": "application/octet-stream"
        },
        description: "Uploading " + name
    };
```

For a single file upload, use the following code:

```JavaScript
var task = session.uploadFile(file, request);
```

For multiple files or to pass additional data, use the multipart upload method. All parameter values must be strings:

```JavaScript
var params = [
   { name: "test", value: "value" },
   { name: "fileToUpload", filename: file, mimeType: "image/jpeg" }
];
var task = session.multipartUpload(params, request);
```

In order to have a successful upload, the following must be taken into account:

- the file must be accessible from your app. This may require additional permissions (e.g. access documents and files on the device). Usually this is not a problem - e.g. if you use another plugin to select the file, which already adds the required permissions.
- the URL must not be blocked by the OS. Android Pie or later devices require TLS (HTTPS) connection by default and will not upload to an insecure (HTTP) URL.

### Upload task API

The task object has the following properties and methods, that can be used to get information about the upload:

Name | Type | Description
--- | --- | ---
upload | `number` | Bytes uploaded.
totalUpload | `number` | Total number of bytes to upload.
status | `string` | One of the following: `error`, `uploading`, `complete`, `pending`, `cancelled`.
description | `string` | The description set in the request used to create the upload task.
cancel()| `void` | Call this method to cancel an upload in progress.

### Handling upload events

After the upload task is created you can monitor its progress using the following events:

```JavaScript
task.on("progress", progressHandler);
task.on("error", errorHandler);
task.on("responded", respondedHandler);
task.on("complete", completeHandler);
task.on("cancelled", cancelledHandler); // Android only
```

Each event handler will receive a single parameter with event arguments:

```JavaScript
// event arguments:
// task: Task
// currentBytes: number
// totalBytes: number
function progressHandler(e) {
    alert("uploaded " + e.currentBytes + " / " + e.totalBytes);
}

// event arguments:
// task: Task
// responseCode: number
// error: java.lang.Exception (Android) / NSError (iOS)
// response: net.gotev.uploadservice.ServerResponse (Android) / NSHTTPURLResponse (iOS)
function errorHandler(e) {
    alert("received " + e.responseCode + " code.");
    var serverResponse = e.response;
}


// event arguments:
// task: Task
// responseCode: number
// data: string
function respondedHandler(e) {
    alert("received " + e.responseCode + " code. Server sent: " + e.data);
}

// event arguments:
// task: Task
// responseCode: number
// response: net.gotev.uploadservice.ServerResponse (Android) / NSHTTPURLResponse (iOS)
function completeHandler(e) {
    alert("received " + e.responseCode + " code");
    var serverResponse = e.response;
}

// event arguments:
// task: Task
function cancelledHandler(e) {
    alert("upload cancelled");
}
```

## Testing the plugin

In order to test the plugin, you must have a server instance to accept the uploads. There are online services that can be used for small file uploads - e.g. `http://httpbin.org/post` However, these cannot be used for large files. The plugin repository comes with a simple server you can run locally. Here is how to start it:

```bash
cd demo-server
npm i
node server 8080
```

The above commands will start a server listening on port 8080. Remember to update the URL in your app to match the address/port where the server is running.

>Note: If you are using the iOS simulator then `http://localhost:8080` should be used to upload to the demo server. If you are using an Android emulator, `http://10.0.2.2:8080` should be used instead.

## Contribute

We love PRs! Check out the [contributing guidelines](CONTRIBUTING.md). If you want to contribute, but you are not sure where to start - look for [issues labeled `help wanted`](https://github.com/NativeScript/nativescript-background-http/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22).

## Get Help

Please, use [github issues](https://github.com/NativeScript/nativescript-background-http/issues) strictly for [reporting bugs](CONTRIBUTING.md#reporting-bugs) or [requesting features](CONTRIBUTING.md#requesting-new-features). For general questions and support, check out [Stack Overflow](https://stackoverflow.com/questions/tagged/nativescript) or ask our experts in [NativeScript community Slack channel](http://developer.telerik.com/wp-login.php?action=slack-invitation).

## License

Apache License Version 2.0, January 2004
![](https://ga-beacon.appspot.com/UA-111455-24/nativescript/nativescript-background-http?pixel)
