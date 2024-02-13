# Changelog

All notable changes to this project will be documented in this file.

## [23.10.1] (2023-11-09)

### Bug Fixes

* **InitAllSettings.xaml:** Fixed an issue regarding the retrieval of credentials. Credentials are retrieved based on the Value instead of the Name column. 

### Removed
* **assets.json:** Removed due auto-creation
* **config.json:** Removed due auto-creation

## [23.10.0] (2023-10-30)

### Features

* **analysequeue.yml:** Ignore postponed transactions, so that the job will be stopped without initializing applications if there are only postponed queue items
* **GetInitiationSource:** Improved determination of UiPath Version by trying to approach all running instances.
* **InitAllSettings.xaml:** Improved the storage of credentials in the Windows Credential Manager: added an existence test and added the process name as a prefix of the credential (in case of duplicate keys for different UiPath processes. Moved away from dictPassword. All credentials (as listed in the credentials sheet in config.xlsx) are saved in the Windows Credential Manager with key [processname]_[key_in_config_file] (the key is also saved in dictConfig) and can easily be retrieved using the "Get Secure Credentials" activity.
* **project.json:** Upgraded dependencies to the latest stable versions: UiPath.Excel.Activities, UiPath.Mail.Activities, UiPath.System.Activities, UiPath.UIAutomation.Activities and UiPath.WebAPI.Activities

### Removed
* **GetAppCredentials.xaml:** Removed due to redundancy

## [23.9.1] (2023-09-28)

### Bug Fixes

* **InitAllSettings.xaml:** Corrected a bug regarding the SecurePassword attribute in NetworkCredentials in out_dictPassword

## [23.9.0] (2023-08-30)

### Features

* **azure-pipelines.yml:** Introduced the azure-pipelines.yml file to provide support for pipelines.
* **project.json:** Excluded assets.json and config.json from being published.
* **GetInitiationSource:** Improved determination of UiPath Version.

### Bug Fixes

* **Framework:** Addressed an issue within the Framework by populating empty JSON Payloads of HTTP requests with an empty string. This resolves a UiPath bug that triggered errors in newer studio versions.
* **InitAllSettings.xaml:** Fixed a problem where an Orchestrator HTTP request, responsible for retrieving the robot id, caused an exception when the process was executed in an unattended mode.
* **DirectReporter.xaml:** Rectified a limitation in the reporter, removing the previous constraint of a maximum limit of one thousand.
* **InitAllSettings.xaml:** Eliminated the requirement for the Syncfusion.XlsIO.Net.Core dependency during the conversion of config.xlsx to JSON.
* **InitAllSettings.xaml:** Corrected a UiPath bug where passwords were also being saved in .Password instead of solely in .SecurePassword.

## [22.12.0] (2022-12-27)

### Features

* **.gitignore:** Exclude .entities from gitignore to improve use of Data Services.
* **Config.xlsx:** Added Orchestrator Folder Name to Config file.
* **Framework:** Added Orchestrator Folder Name to Orchestrator API Requests.
* **.gitignore:** Exclude .settings from gitignore to allow for custom project activity settings.
* **.settings:** Updated custom project activity settings according to Ciphix standards. See README.md for settings.
* **.settings:** Enable Modern Design Experience by default in Framework.
* **InitAllSettings.xaml:** Added new InitAllSettings file with JSON Config and Orchestrator API Asset retrieval.
* **InitAllSettings.xaml:** Updated InitAllSettings with dictPassword for credentials from Config. 
* **Framework:** Changed dictConfig from Dictionary(Of String, Object) to Dictionary(Of String, String) and removed .ToString expressions in framework. 
* **Framework:** Updated framework to Windows Target Framework because Legacy compatibility will be deprecated in 2023.

### Bug Fixes

* **Config.xlsx:** Fixed a typo in Config file.
* **InitAllSettings.xaml:** Fixed read config file exception message variables.
* **Framework:** Replaced Deserialize JSON activities with Assign activities for deserialization without dependency conflicts after update.
* **Framework:** Removal of 3rd party packages during update to Windows Target Framework.
* **DirectReporter:** Updated header in report email to English when ENG report language is selected.