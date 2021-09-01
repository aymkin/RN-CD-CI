## `New` Code signing philosophy

Every iOS and React Native developer knows potential probles with code signing of iOS application.

When deploying an app to the App Store, a beta testing service or even installing it on a single device, most development teams have separate code signing identities for every member. This results in dozens of profiles including a lot of duplicates.

You have to manually renew and download the latest set of provisioning profiles every time you add a new device or a certificate expires. Additionally this requires spending a lot of time when setting up a new machine that will build your app.

### Solution

> Keep Your Keys In-Sync with Git

What if there was a central place where your code signing identity and profiles are kept, so anyone can access them during the build process? This way, your entire team can use the same account and have one code signing identity without any manual work or confusion.

Instead of registering for yet another service, you can use a separate private Git repo to sync your profiles across multiple machines.

We will use this approach (using Fastlane action `match`) to simplify and automate iOS apps deployment. You may want to read about philosofy and approach on web site https://codesigning.guide/

## Fastlane ([link](https://fastlane.tools/))

### Preconditions

- According to the CodeSigning Philosophy we well need to create Apple ID account with admins prevelegies. We reccomend to name it explicitly, for example `release-bot@yourcompany.com` Apple ID
- Invite `bot` as App Manager to your Team at AppstoreConnect https://appstoreconnect.apple.com/access/users and confirm the invitation
- Create new private repository, which belongs to your company. This repo will hold all certificates and provisioning profiles for iOS Code Signing. If your company ownes several project (apps) - you will need only one repo to hold all certificates and provisioning profiles
- Create separate shareble `release-bot@yourcomany.com` GitHub account (or any other VCS)
- Every team member who uses the Fastlane should generate SSH and add to the `bot` GH account. Beside that the Bitrise service should have its own SSH key pair to intearat with GitHub repo as well.

### Fastlane setup

The Fastlane itself - is a library of Ruby scripts, which automate routine tasks. In oreder to be able run Fastlane on the Mac - we need to install Bundler

- install bundler `sudo gem intall bundler`

All Ruby dependencies are kept in Gemfile - a brother of `package.json` is JS project. So we need to initialize two Gemfiles in iOS and Android folders. Go to root dirictory of the RN project and execute the command.

- `cd ios && bundle init && cd ..`

This will populate two Gemfiles. In both of them on line 7 add only on dependency Fastlane. Have a look on Gemfiles on corresponding ios and android folders.

`gem "fastlane"`

After specifying Faslane dependency - update bundle (install dependency). You may need to enter your users password.

- `cd android && bundle update && cd ../ios && bundle update && cd ..`

That will populate two Gemfile.lock files. Make sure to commit all of Gemfiles. From now on every Fastlane command may be executed via `bundle exec fastlane ...`

## Fastlane `produce`

If you have not create your iOS app in the AppStoreConnect - you may do it automatically by using Fastlane action `produce` https://docs.fastlane.tools/actions/produce/

- navigate to the ios dirictory and execute `bundle exec fastlane produce`
- fastlane will ask you credentials for Apple ID
- use `bot's` Apple ID, for example `bot@yourcompany.com`
- specify App Indetifier (bundle ID), for exapmle `com.yourcompany.amazingApplicationName`
- specify App Name, for example `Amazing App`
- go to developer portal, login with your developer credentials (not `bot's` Apple Id) and verify registerd identifier https://developer.apple.com/account/resources/identifiers/list
- go to appstore connect and select my apps https://appstoreconnect.apple.com/apps and verify that app is created

## Fastlane iOS `init`

> for published apps

https://docs.fastlane.tools/getting-started/ios/setup/

- go to `ios` folder by `cd ios`
- execute `bundle exec fastlane init`

That command will create the most basic fastlane configuration. So provide `bot's` Apple Id credentials when fastline ask. So, remember to use `bot's` Apple ID every time Fastline ask you about Apple ID.

👉 As far Bot's Apple ID has AppManeger's prevelegies - that will need 2FA. For local usage on developer's machine - the good option is Method 3: Application Specific password. For cloud Bitrise (we will use this one) - the best option is Method 1: App Store Connect API key.

🔶 To read more about Authenticating with Apple services - https://docs.fastlane.tools/getting-started/ios/authentication/

Executing `bundle exec fastlane init` will create Fastlane folder in ios directory and download existing metadata from AppStore

As far we would not like to keep autogenerated files - we should exlude those from Git index, so add lines to the `.gitingnore`

- ios/fastlane/report.xml
- ios/fastlane/Preview.html
- ios/fastlane/screenshots

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
