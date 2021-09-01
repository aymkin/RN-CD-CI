## Fastlane ([link](https://fastlane.tools/))

### Fastlane setup

The Fastlane itself - is a library of Ruby scripts, which automate routine tasks. In oreder to be able run Fastlane on the Mac - we need to install Bundler

- install bundler `sudo gem intall bundler`

All Ruby dependencies are kept in Gemfile - a brother of `package.json` is JS project. So we need to initialize two Gemfiles in iOS and Android folders. Go to root dirictory of the RN project and execute the command.

- `cd android && bundle init && cd ..`

This will populate two Gemfile. In both of them on line 7 add only on dependency Fastlane. Have a look on Gemfiles on corresponding ios and android folders.

`gem "fastlane"`

After specifying Faslane dependency - update bundle (install dependency). You may need to enter your users password.

- `cd android && bundle update && cd ..`

That will populate two Gemfile.lock files. Make sure to commit all of Gemfiles. From now on every Fastlane command may be executed via `bundle exec fastlane ...`

## Fastlane `produce`

> If you have not create your android app

The very first time you should build it locally and upload manually to Google Play console. The initials app setup in Google Play console is not part of this guide - so we gonna skip it.

Notes about it:

- use Google Play App Signing
- use tracks Internal test and Production

Build first app locally:

- `cd android && mkdir secure && cd secure`
- generate the keystore (upload key): use [guide](https://developer.android.com/studio/publish/app-signing#sign-apk) or use keytool

```
keytool -genkeypair
  -v -keystore your-upload-key.keystore
  -alias upload-alias
  -keyalg RSA
  -keysize 2048
  -validity 10000
```

- enter keystore password, make sure you save it if a safe place, re-enter the password
- answer the questions about your name - use [fictioal bot's identety](../ios/fastlane/Fastfile)
- answer the questions about organization, city, country
- in the end explicitly type `yes`
- specify password for alias, make sure you save it if a safe place, re-enter the password

Open `android/app/build.gradle` and below the **defaultConfig {}** insert

```java
defaultConfig {
        applicationId project.config.get("BUNDLE_ID")
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 48
        versionName project.config.get("APP_VERSION")
        // the actual version name will be extracted from APP_VERSION rnuc yaml configs
    }

    signingConfigs {
      release {
        if (System.getenv("ANDROID_KEYSTORE_FILE")) {
          storeFile file(System.getenv("ANDROID_KEYSTORE_FILE"))
          storePassword System.getenv("ANDROID_KEYSTORE_PASSWORD")
          keyAlias System.getenv("ANDROID_KEY_ALIAS")
          keyPassword System.getenv("ANDROID_KEY_PASSWORD")
        }
      }
    }
```

- make sure to specify mentioned env variables, otherwise build will not succeed.
- in your env varibles should be specified following variables
  - ANDROID_KEYSTORE_FILE=path/to/generated/your-upload-key.keystore
  - ANDROID_KEYSTORE_PASSWORD=\***\*saved_kestore_password\*\***
  - ANDROID_KEY_ALIAS=upload-alias
  - ANDROID_KEY_PASSWORD=\***\*saved_alias_password\*\***
- insted of using envs you may explicitly set path, alias and passwords, **BUT I DO NOT RECCOMEND TO KEEP IT IN GIT**
- if you do not have ENV set, you may use temporary session env variables, enter in termibal before build for example

```
    export ANDROID_KEYSTORE_FILE="../../../release-key.keystore"
    export ANDROID_KEYSTORE_PASSWORD="very_strong_password"
    export ANDROID_KEY_ALIAS="alias_name"
    export ANDROID_KEY_PASSWORD="very_strong_password"
```

- in the same `android/app/build.gradle` in build types specify signing config

  ```
  buildTypes {

        release {
            // Caution! In production, you need to generate your own keystore file.
            // see https://reactnative.dev/docs/signed-apk-android.
            // add line with signing config below
            signingConfig signingConfigs.release // <<<<<-------
            minifyEnabled enableProguardInReleaseBuilds
            proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
        }
  ```

  So, actual build:

- `cd android`
- `./gradlew assembleRelease` that will create file at `android/app/build/outputs/bundle/release/app-release.aab`

By runnin this command Gradle will prepare build signed by upload key.

Go to Play Console -> select you app -> select internal testin -> create new release -> upload `app-relese.abb`

## Fastlane android `init`

> for published apps

https://docs.fastlane.tools/getting-started/android/setup/

- go to `android` folder by `cd android`
- execute `bundle exec fastlane init`

* That command will create the most basic fastlane configuration.
* Provide Package Name (example: com.mycompany.awesomeapp).
* Then asked to provide JSON secret file - just skip it by hitting the button enter.
* For question "Do you plan to on uploading metadata ..." - reply "no"

Executing `bundle exec fastlane init` will create Fastlane folder in android directory and create Appfile and Fastfile. In Appfile path to JSON will define later. In Fastfile defined lanes which are just collection of scripts to automate the task of building and deployin android apps.

## Fastlane android JSON secret

In order to let Fastlane to upload android apps directly

reference https://developers.google.com/android/management/service-account

- go to the Google Play Console -> Setup -> API access -> find button `create servise account
- In modal window in very first point "Navigate to the Google API console" - click it
- on a separete tab click the button "create service account"
- Service account name - type somethins memorizible, for expamle name of your release bot.
- select a role: Project - Service Account -> Service acount actor

---

## Faslane iOS `match` || `sync_code_signing`

> to manage code signing
>
> Easily sync your certificates and profiles across your team (via match)

https://docs.fastlane.tools/actions/sync_code_signing/
https://docs.fastlane.tools/actions/match/

run `bundle exec fastlane macth init`

- You will need to provide the URL for Git Repo wich holds the code signing certificates. Remember - one repo for all your Apple cetificates and provisioning profiles
- Make sure specify the SSH url
- This information will be kept in ios/fastlane/Matchfile
- So this command will generate and autgenerate if needed signing identities

run `bundle exec fastlane macth appstore`

- 🈲 running this command first time on a new machine the fastline will ask you for passphrase, make sure to save it, we will need it to run `match` on a different machine
- you can execute match with the different profile type, can be appstore, adhoc, development, enterprise, developer_id, mac_installer_distribution. About distribution methods (profile types) you can read here https://help.apple.com/xcode/mac/current/#/dev31de635e5

to create development signing identities run `bundle exec fastlane macth development`

### Xcode

- open your project in xcode
- select your main target
- in Signing && Capabilities turn off `automatically manage codesigning`
- on Signing (Debug) select `match Development ...`
- on Signing (Release) select `match Appstore ...`

On test target ...

- do not forget to do the same for Test Target too turn off `automatically manage codesigning`
- signing (Debug) select your Team
- signing (Release) select your Team

Go to Build settings and type `code sign`

- for 🌟**both**🌟 main and test targets
- select iOS Developer for Debug
- select iOS Distribution for Release

### Time to test

from ios directory run

`xcodebuild clean build -workspace yourApplicationName.xcworkspace -configuration Release -scheme yourApplicationName`

The end message should be ** BUILD SUCCEEDED **

Also you may try to build it from xcode UI

- `⇧ + ⌘ + K` to clean
- `⌘ + B` to build

## Before first iOS release (if so)

The app must be with icons.

- go to [online tool](https://appiconmaker.co/) and paste icon of size 1024 \* 1024 px (or use any other you like better)
- download the icons
- in xcode set Images.xcasswts -> AppIcon -> to all available sizes
- or you can click tiny arrow in General tab, section "App Icons ..."
  ![App Icons](https://prnt.sc/1qettl6)

  https://prnt.sc/1qettl6

## iOS first Test Flight release

- open `ios/fastlane/Fastfile`
- find `lane :beta do`
- next line should be match(type: "appstore") - uncomment it
- before match insert line with action `increment_build_number`
- in **gym** action

```ruby
    cocoapods
    gym(
        shceme: "MyProjectName",
        workspace: "./MyProjectName.xcworkspace"
    )
```

- make sure -> before **gym** action **cocoapods** action is called
- make sore -> in Gemfile after `gem fastlane` listed `gem cocoapods`
- after action **pilot** insert `clean_buld_artefacts`
- next line should be commiting the version bump

```ruby
    commit_version_bump(
        message: "Fastlane iOS: Released new build #{lane_context[SharedValues::BUILD_NUMBER]} [ci skip]",
        xcodeproj: "./MyProjectName.xcodeproj", # relatively to the ios folder
        force: true
    )
```

- for **pilot** action use this argument

```ruby
    pilot(
        skip_waiting_for_build_processing: true
    )
```

- now from ios folder run `bundle exec fastlane beta`

That commant will sign the code (match), build the app (gym) and upload (pilot) to the Test Flight.

## iOS Missing Compliance

For the very first updload the Test Flight - you will see the worning in the Test Flight portal :warn Missing Compliance.

- On TestFlight tab next to the lowest build nuber click on the yellow triangle with exclamation sign -> Provide Export Compliance -> Select NO -> Start Internal Testing -> will see green sign with `Ready to test`
- Open Info.plist and provide new key (just below CFBundleVersion), the result will look like

```xml
<key>CFBunldeVersion</key>
<string>2</string>
<key>ITSEncryptionExportComplianceCode</key>
<false/>
```

- run again `bundle exec fastlane beta`
- check this was accepted in TestFligh automatically (should be Ready to Test)

## iOS: invite testers (up to 25 users)

You may google it by your own or use this link https://help.muvi.com/help/mobile-apps/how-to-add-testers-to-test-the-ios-app-in-testflight.html
