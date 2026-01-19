# FMI Weather -application's international version in a nutshell

V 1.0.1 - 16th january 2026

## Introduction

The FMI Weather application's international version is a fork of the FMI version. It uses the same codebase and a configuration file to enable or disable features and data interfaces. Code changes in the FMI version (security updates, etc.) are merged into the international versions a few times a year. Code changes are never merged from the international version back into the FMI version.

The FMI Weather application includes both the mobile app and widgets for iOS and Android.

## Technology stack and other related requirements

* React Native
* React Native Maps
* Node
* TypeScript
* Swift/SwiftUI (iOS widgets)
* Java (Android widgets)
* Redux
* Github Actions
* YAML
* Google Play
* iOS App Store
* SVG (weather and warning icons as well as some images in application)
* JSON
* General experience in mobile app development (the environment is quite different from web development)
* It is HIGHLY recommended that developer(s) have MacOS for development since iOS requires MacOS (Xcode). Android development can be done with non-Mac machine.

## Server requirements

The weather app requires Smartmet Server for forecast and observation data. Smartmet Server also provides WMS layers to the application, but GeoServer can also be used.

* Smartmet Server
  
  * The Timeseries interface is used for forecast and observation point data in tables/graphs (requires forecast and observation weather data in Smartmet Server).
  
  * The Autocomplete interface is used for searching locations (requires a location database).
  
  * The WMS interface is used for visualizing weather data layers on the map (requires WMS layers in Smartmet Server).
  
  * Smartmet Server's interfaces needs to be public so it is visible to the application
- Smartmet Alert
  
  - capfeed.php is used to fetch warnings
  
  - Warnings from MeteoAlarm are also supported

## How to get started

- [FMI Weather App in Github](https://github.com/fmidev/weather-app): the source code is open and is licensed under the MIT License

- Smartmet Server needs to be initialized, all necessary data integrated and all required interfaces need to be working publicly accessible

- Smartmet Alert's capfeed.php needs to be accessible to the app if warnings are being visualized

- Check the instructions in the README file.

- defaultConfig.ts needs to be edited with working URLs, etc. (same for widgetConfig.json)

- The FMI Weather app’s GitHub wiki describes defaultConfig's settings

## Afterwords

If all goes well, the app should start and visualize the provided data. However, in our experience most of the work is in setting up the servers, getting both app stores working, and ensuring the SVG images are correct rather than working on the application code itself. Once everything is running smoothly, both app stores have yearly update and other requirements, so you can’t simply leave the app untouched.

## Disclaimer

All possible fees from Play Store & App store accounts and annual fees needs to be taken care of target country.

If the target country makes changes to the application code, FMI is not responsible for merging new updates. Updates are still provided to the forked repository (as a separate branch from main), but the target country is responsible for merging the updates into their main branch.

There may be Smartmet Server performance issues depending on the number of application users. The FMI Weather application has around 500k installations and is supported by several Smartmet Servers, which also serve other FMI services. For that reason, we cannot promise that the same setup will work with millions of users and the target country’s server setup.

## Links

[GitHub - FMI Weather app](https://github.com/fmidev/weather-app)

[GitHub - Smartmet Server Timeseries plugin](https://github.com/fmidev/smartmet-plugin-timeseries)

[GitHub - Smartmet Server WMS plugin](https://github.com/fmidev/smartmet-plugin-wms)

[GitHub - Smartmet Server Autocomplete plugin](https://github.com/fmidev/smartmet-plugin-autocomplete)






