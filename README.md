# Picker plugin for Flutter

A Flutter plugin for iOS and Android for picking images and videos from gallery or camera,
and save files on gallery

## Installation

First, add `picker` as a [dependency in your pubspec.yaml file](https://flutter.io/platform-plugins/).

### iOS

Add the following keys to your _Info.plist_ file, located in `<project root>/ios/Runner/Info.plist`:

* `NSPhotoLibraryUsageDescription` - describe why your app needs permission for the photo library. This is called _Privacy - Photo Library Usage Description_ in the visual editor.
* `NSCameraUsageDescription` - describe why your app needs access to the camera. This is called _Privacy - Camera Usage Description_ in the visual editor.
* `NSMicrophoneUsageDescription` - describe why your app needs access to the microphone, if you intend to record videos. This is called _Privacy - Microphone Usage Description_ in the visual editor.

### Android

No configuration required - the plugin should work out of the box.

### Example

``` dart
import 'package:picker/picker.dart';

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  File _image;

  Future getImage() async {
    var image = await Picker.pickImage(source: ImageSource.camera,  maxHeight: 480, maxWidth: 640, quality: 75);

    // if you want save a file on gallery:
    var filePath = await Picker.saveFile(fileData: image);

    setState(() {
      _image = image;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Image Picker Example'),
      ),
      body: Center(
        child: _image == null
            ? Text('No image selected.')
            : Image.file(_image),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: getImage,
        tooltip: 'Pick Image',
        child: Icon(Icons.add_a_photo),
      ),
    );
  }
}
```

### Handling MainActivity destruction on Android

Android system -- although very rarely -- sometimes kills the MainActivity after the picker finishes. When this happens, we lost the data selected from the picker. You can use `retrieveLostData` to retrieve the lost data in this situation. For example:

```dart
Future<void> retrieveLostData() async {
  final LostDataResponse response =
      await ImagePicker.retrieveLostData();
  if (response == null) {
    return;
  }
  if (response.file != null) {
    setState(() {
      if (response.type == RetrieveType.video) {
        _handleVideo(response.file);
      } else {
        _handleImage(response.file);
      }
    });
  } else {
    _handleError(response.exception);
  }
}
```

There's no way to detect when this happens, so calling this method at the right place is essential. We recommend to wire this into some kind of start up check. Please refer to the example app to see how we used it.
