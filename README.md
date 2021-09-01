# CD / CI for React Native app with Fastlane && Bitrise

## Automate

- Code Signing
- Provisioning profiles and Certificates management for iOS
- Beta deployment
- App Store and Google Play deployment

## Environment variables management

This tool is optional and may any other to manage your Env variable, so please have a look on [React Native Ultimate Config](https://github.com/maxkomarychev/react-native-ultimate-config).
During development, most probably, our application will need to use different environment variables. The minimal info you can put in there:

- A variable to generate BASE_URL for API calls.
- App Name to easily detect the app on your hardware device

Chapter 1: [setup Fastlane for local iOS bulds](ios/fastlane.md)
Chapter 2: [setup Fastlane for local Android builds](ios/fastlane.md)
Chapter 3: setup Bitrise for cloud iOS builds
Chapter 4: setup Bitrise for cloud Android builds
Chapter 5: RNUC and app verisoning
