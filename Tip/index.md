## Flutter Flavors Setup with multiple Firebase Environments using FlutterFire.

1. Install and run the FlutterFire CLI
```dart
dart pub global activate flutterfire_cli
```
2. If we look at the generated Flutter app, we'll find the following files inside ```lib```:
```dart
main_development.dart
```
```dart
main_production.dart
```
```dart
main_staging.dart
```
3. And we can run the generated app by passing the correct arguments on the command line:
#### Run development
```dart
flutter run --flavor development --target lib/main_development.dart
```
#### Run staging
```dart
flutter run --flavor staging --target lib/main_staging.dart
```
#### Run production
```dart
flutter run --flavor production --target lib/main_production.dart
```
In android/app/src/ folder create new folders as prod and dev as shown below. This folder name must match the flavor name specified in productFlavors. Put the downloaded google-services.json files into the correct folders considering the relevant environment. So, this will automatically connect to the correct firebase project according to the flavor we are running on.

```
android/
     app/
        src/
           development/ google-services.json
           staging/ google-services.json
           production/ google-services.json

```
Now we need to connect our 2 schemes to relevant Firebase projects. For that, we create a new folder named “config” separately and inside it, we create two new folders as prod and dev. Put the downloaded GoogleService-Info.plist files into the correct folder considering the environment.

Then drag and drop this config folder explicitly into XCode inside to “Runner”.
```
ios/
    config/
           development/ GoogleService-Info.plist
           production/ GoogleService-Info.plist
           staging / GoogleService-Info.plist
```
## we have to create one Firebase project for each Flutter flavor. For consistency, we could name the Firebase projects like so:

```
my-test-app-flavors-dev
my-test-app-flavors-stg
my-test-app-flavors-prod
```
And then, we can use the FlutterFire CLI generate the correct Dart initialization file for each Flutter flavor, with this command:

##### Dev environment (note: do the same for Stg and Prod)
```dart
flutterfire config \
  --project=my-test-app-flavors-dev \
  --out=lib/firebase_options_dev.dart \
  --ios-bundle-id=com.codewithandrea.my-test-app-flavors.dev \
  --macos-bundle-id=com.codewithandrea.my-test-app-flavors.dev \
  --android-app-id=com.codewithandrea.my_test_app_flavors.dev
```
When we do this, we have to use the correct bundle ID on iOS and macOS. This can be found by opening the project build settings in Xcode:

<img width="828" alt="Screenshot 2023-12-03 at 4 26 31 PM" src="https://github.com/sudeshnb/Flutter_usecase/assets/33403844/c22f3cfb-882f-4fb1-ac74-fa944578f7d2">

And on Android, the correct applicationId can be found inside android/app/build.gradle, along with the applicationIdSuffix used for each flavor:

android {
    ...    
    defaultConfig {
    ...
    }

```dart
  flavorDimensions "default"
    productFlavors { 
        production {
            dimension "default"
            applicationIdSuffix ""
            manifestPlaceholders = [appName: "My Test App Flavors"]
        }
        staging {
            dimension "default"
            applicationIdSuffix ".stg"
            manifestPlaceholders = [appName: "[STG] My Test App Flavors"]
        }        
        development {
            dimension "default"
            applicationIdSuffix ".dev"
            manifestPlaceholders = [appName: "[DEV] My Test App Flavors"]
        }
    }
```
....
}

## Dart initialization for each Flutter flavor

Once we have run flutterfire config for each Flutter flavor, the last step is to use the correct Firebase options for each target:

```dart
// main_development.dart
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'package:my_test_app_flavors/app/view/app.dart';
import 'package:my_test_app_flavors/firebase_options_dev.dart';
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(const App());
}
```
# Step 3- Set up iOS configuration

For iOs configuration, we need to use Xcode Schemes which is same as product flavors in Android. Open ios/Runner.xcworkspace from Xcode and click on the Runner scheme as shown below.

<img width="528" alt="Schemes" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*WrIBIoAva_6qMz6y3jGQyQ.png">

Here, go to “New Scheme” and create a new scheme as “dev”. Give Runner as the target.

<img width="528" alt="Schemes" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*lQSJ8-YnJLoxzGm8fokduQ.png">

Select Runner project ->Info -> Configurations. Click on the plus icon and give “Duplicate Debug Configuration”. Then, rename the duplicated configuration as Debug-dev. Do the same process for duplicated release and profile configurations too.

<img width="528" alt="Schemes" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dj4mjsya1MJZ4TNCH2oFDw.png">

Now, we need to connect each build configuration to the correct scheme. First, select dev scheme and go to “Edit Scheme”.

<img width="528" alt="Schemes" src="https://miro.medium.com/v2/resize:fit:1212/format:webp/1*XIv1SfCkEmoLMUi8CHOdbg.png">

In below window, Select “Run” and give “Debug-dev” as the build configuration. Give the correct build configurations in “Test”, “Profile”, Analyze” and “Archive” actions too as shown in the following image. Do this same process for prod scheme.

<img width="528" alt="Schemes" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*030Ey9Zr36_bR7H-K5_BsQ.png">


Next, we are going to instruct the build process to copy the correct GoogleServices-Info.plist file into the real location where Firebase init code expects to find GoogleServices-Info.plist file. For that, select “Runner” Target. Go to “Build Phases” , click on plus icon and give “New Run Script Phase”.

<img width="528" alt="Schemes" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Ot3J0XxuTqCQzESVEsHh7A.png">

<img width="528" alt="Schemes" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*nIM2WlQpvO4VkjEujGBvig.png">

```
# Get a reference to the destination location for the GoogleService-Info.plist
# This is the default location where Firebase init code expects to find GoogleServices-Info.plist file.
PLIST_DESTINATION=${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app
# We have named our Build Configurations as Debug-dev, Debug-prod etc.
# Here, dev and prod are the scheme names. This kind of naming is required by Flutter for flavors to work.
# We are using the $CONFIGURATION variable available in the XCode build environment to get the build configuration.
if [ "${CONFIGURATION}" == "Debug-prod" ] || [ "${CONFIGURATION}" == "Release-prod" ] || [ "${CONFIGURATION}" == "Profile-prod" ] || [ "${CONFIGURATION}" == "Release" ]; then
cp "${PROJECT_DIR}/config/prod/GoogleService-Info.plist" "${PLIST_DESTINATION}/GoogleService-Info.plist"
echo "Production plist copied"
elif [ "${CONFIGURATION}" == "Debug-dev" ] || [ "${CONFIGURATION}" == "Release-dev" ] || [ "${CONFIGURATION}" == "Profile-dev" ] || [ "${CONFIGURATION}" == "Debug" ]; then
cp "${PROJECT_DIR}/config/dev/GoogleService-Info.plist" "${PLIST_DESTINATION}/GoogleService-Info.plist"
echo "Development plist copied"
fi
```



