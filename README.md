# Ciphix Framework: Performer with Queue and DirectReporter

# Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
    1. [Prerequisites](#prerequisites)
    2. [Default Project Settings](#default-project-settings)
3. [Framework Layout](#framework-layout)
    1. [Init](#init)
    2. [Get Transaction Data](#get-transaction-data)
    3. [Process Transaction](#process-transaction)
    4. [End Process](#end-process)
4. [Build and Test](#build-and-test)
    1. [(JSON) Config File](#json-config-file)
    2. [Workblock Templates](#workblock-templates)
    3. [Get Input Data](#get-input-data)
    4. [Developer Responsibilities](#developer-responsibilities)
    5. [Data Flow Explanation](#data-flow-explanation)
    6. [Transaction Processing](#transaction-processing)
    7. [DirectReporter](#directreporter)
    8. [Tips & Tricks](#tips-&-tricks)
5. [Contributing](#contributing)

* * *

# Introduction

Ciphix Frameworks are the core of our RPA development. The main reasons for creating this framework are cleanness, efficiency and robustness. The Ciphix framework makes development and the robot more efficient and more adaptable to our client's needs and wishes. 
It is important to understand how the framework works, which components are to be edited, and which are not. Correctly using the framework ensures the quality of our RPA delivery. This document explains the features and essential know-hows of the Ciphix Performer with Queue Framework.

The Ciphix Performer with Queue Framework is an extensive framework for a performer inspired by the REFramework. This framework template can only be used for Robotic Processes that make use of an Orchestrator Queue to store the transactions. 
This framework has a ‘Direct Reporter’ workflow embedded in the ‘End Process’. This allows to immediately create a report of the processed items in that performer run with the use of a single Orchestrator queue. The report is created with the data you deem necessary.

* * * 

# Getting Started

## Prerequisites

-   You have downloaded UiPath Studio (version depends on client).
-   You have downloaded Ciphix Performer with Queue Framework from Azure DevOps.

## Default Project Settings

The default project settings are updated to facilitate a smoother way of working and improve adherence to development standards.
Besides updated default project settings, the **Modern Design Experience** is enabled. 
<br> Changed project default settings are applied in the project, but not updated visibly in the Properties panel of classic activities. They are visible in modern activities.

**Changed Default Project Settings:**
<br> **Excel Business**
<br> *Excel Visibility*
<br> Show Excel window | *Run + Debug Value* -> False

**UI Automation Modern**
<br> *Targeting methods - Desktop*
<br> Input mode -> Simulate
<br> Ignore selectors with IDX / tableRow / tableCol -> False

*Targeting methods - Web*
<br> Strict selector -> True
<br> Ignore selectors with IDX / tableRow / tableCol -> False

*Targeting methods - Java*
<br> Input mode -> Simulate
<br> Ignore selectors with IDX / tableRow / tableCol -> False

*Targeting methods - SAP*
<br> Enable anchors -> True
<br> Ignore selectors with IDX / tableRow / tableCol -> False

*Verify execution*
<br> Always auto-verify typed text -> True

*Application/Browser*
<br> Resize window -> Maximize

*Keyboard Events*
<br> Empty Field -> Multi line

*Robot Logging*
<br> Log target & anchor search steps | *Debug value* -> True

**UI Automation Classic**
<br> *Browser*
<br> Browser | *Run + Debug Value* -> Chrome

*Robot Logging*
<br> Log target info | *Debug Value* -> True

*Keyboard Events*
<br> SimulateType | *Run + Debug Value* -> True
<br> ClickBeforeTyping | *Run + Debug Value* -> True
<br> EmptyField | *Run + Debug Value* -> True

*Mouse Events*
<br> SimulateClick | *Run + Debug Value* -> True

**System**
<br> *Invoke Workflow File*
<br> Log Entry | *Debug Value* -> With Arguments
<br> Log Exit | *Debug Value* -> With Arguments

* * *

# Framework Layout

This framework is constructed using the Windows Target Framework. Windows Target Framework is based on .NET 6 and brings compiled projects and 64-bit support.
The Framework consists of four states, each with their own function:
-   Init
-   Get Transaction Item
-   Process Transaction Item
-   End Process

## Init

The Init state is used as a starting point of the process. In the Init state all configuration is done relating to the necessary settings and applications.

There are several Framework Workflows in the Init state:
-   **GetInitiationSource** retrieves the initiation source of the process by checking the local logs that are created at the time of running. The log line that defines the execution start is read and the initiation party is extracted. This can be either Robot or Studio. _You don't have to change anything in this workflow._
-   **InitAllSettings** reads the Config file and makes sure all necessary settings, queues, assets and credentials are gathered. _You don't have to change anything in this workflow._
**Arguments:**
    - out_dictConfig: dictionary containing all settings and asset values from the config file.	All credentials (as listed in the credentials sheet in config.xlsx) are saved in the Windows Credential Manager with key [processname]_[key_in_config_file] (the key is also saved in dictConfig) and can easily be retrieved using the "Get Secure Credentials" activity.
-   **KillAllProcesses** kills all applications that are used in the process, in a hard close manner.  _In this workflow, you need to specify which applications need to be killed._
-   **GetMachineDiagnostics** retrieves the machine diagnostics at the moment of running the process. The required information is logged and can be accessed for debugging or running purposes. _You don't have to change anything in this workflow._
-   **Analyse Queue** checks the status of the latest Dispatcher status for Successful or Faulted. It then checks the assigned Orchestrator Queue whether it posssesses any QueueItems with the status New. If there are any, it retreives the creation time of the 'oldest' Newly added QueueItems and passes that on to the Direct Reporter inside the End State of the Framework. If there are no New QueueItems, the process will end before opening any applications and the time of the last processed item (failed or successful) will be passed on to the Direct Reporter to indicate the timespan since the last handled item.
-   **InitAllApplications** starts up all applications used in the process. _Invoke all Open and Login workflows for the applications in this workflow._

Two states can follow the Init state:
-   **Get Transaction Data:** After setting up the configuration of the process and opening all the necessary applications, the process continues to the Get Transaction Data state.
-   **End state:** When a systemError occurs, the Init state will standard be retried the number of times specified in the Config file. After that, the process will continue to the End Process state. The process will end without retrying in case a Business Exception Rule (BRE) is thrown or an empty queue is found.

## **Get Transaction Data**

This state retrieves the transaction items from the Orchestrator Queue. The queue name is specified in the Config file. Generally, nothing has to be changed to this state.

Two states can follow the Get Transaction Data state:
-   **End state:** When no transaction data is available in the queue, the process continues to the End Process state.
-   **Process state:** When transaction data is found, the process continues to the Process Transaction state.

## **Process Transaction**

The Process workflow is used to execute all the necessary processing of the transaction items. In this state, you invoke all workflows used to execute the process.

Generally, two states can follow the Process Transaction:
-   **Init:** If a systemError occurs, the robot will restart its operations, after which the transaction can be retried once. The queue is updated accordingly.
-   **Get Transaction Data:** If a transaction item is processed successfully, the robot will look for a new transaction in Get Transaction Data. If a BRE is thrown while processing the transaction item, the Process Transaction state catches this exception. A BRE is not retried. Instead, the next transaction item is retrieved.

To improve the exception handling, you can add extra activities to the catch parts of the TryCatch surrounding the process state.

## **End Process**

In the End Process state, all activities to close the operation of the robot are performed, such as closing the applications. Furthermore, you can add actions to the End Process state in the `End actions if Successful` part of the state. This could, for example, be creating and sending a report.
 -  **CloseAllApplications:** Closes all applications used in the process. _In this workflow, you need to specify which applications need to be closed._
-   **DirectReporter:** The Direct Reporter is an optional workflow, and can be implemented by setting the job argument for the Direct Reporter (in_isDirectReporterOn) to _True_. The Direct Reporter enables the creation of visual reports based on the same queue that the performer utilizes. All queue items that have been processed since the start of the job are collected using the Orchestrator API. **Sending a report by for example email is something the developer has to customize manually.**


* * *

# Build and Test

To use the Ciphix Performer Framework, several things must be taken into account:

-   How to use the Config file?
-   Workblock templates
-   Using the SmartDispatcher
-   Responsibilities of the Developer
-   Framework Data Flow

## (JSON) Config File

There are several sheets in the Config which need to be completed:

-   **Settings**: Configure the local settings here. These settings should be static and not be subject to change.
-   **Assets & Credentials**: These tabs are divided by the environment. Please specify the correct credentials and assets that will be used in the corresponding environment. Contrary to the settings sheet, the values of Assets & Credentials should be values that are more likely to change every once in a while as their values can be accessed more easily through Orchestrator.
-   **LogMessages**: Additional Log Messages that are added to the framework which provide more detailed diagnostics during the process.
-   **ProcessLogs**: These are process-specific logs that will be sent along with every action that the robot performs. These are used for reporting and dashboarding.
-   **ApplicationLogs**: These logs are used for reporting and maintenance purposes. For every application that is used in the performer, specify the application name and version. When applications are updated, it is easier to identify which processes automate those application.

All Config sheets are read into and cached as a **JSON file** in the project. When no changes are made to the Config.xlsx and Orchestrator Assets, the cached json Config settings are used. In case settings are updated, the json Config is composed from scratch.
The former Excel Config method remains available in the project.

## Workblock Templates

Developers should always use the workblock-templates when developing. They include the wbStartup and wbEnd structure, which is essential for operation, development, and support. Additionally, the template helps building new workflows quicker. There are four Workblock types:

-   **wfBrowserTemplate**: To be used for workflows using browser activities
-   **wfGeneralTemplate**: To be used for workflows using all types of activities
-   **wfModernTemplate**: To be used for workflows using modern activities
-   **wfWindowTemplate**: To be used for workflows using window activities

Make sure to connect the wbPath to the in_wbParentPath when invoking a new workflow.

## Get Input Data

The Performer Framework contains a SmartDispatcher. The SmartDispatcher is used to save time in the development and testing-phases, because it allows for an easy dispatch of data to the Queue, which you can immediately use.

The SmartDispatcher works as follows:

Place the data you would like to dispatch in Data>InputData>InputData.xlsx.

1.  The Column names are Specific content of the QueueItem. The rows represent the values in these QueueItems.
2.  Run Framework>GetDataAndDispatch.xaml. The data in the InputData.xlsx file will be dispatched to the Performer Queue defined in the main config.xlsx.

## Developer Responsibilities

The Framework needs only one thing in order to work correctly: **Complete the required Config-information.**

-   It is recommended that the sheets Config>Settings and Config>Constants are configured by the Solution Architect and the Developer together. This way both parties have a clear understanding of how the framework is set up.
-   The pages Config > Assets and Config > Credentials can be configured by the developer during development, since this may change somewhat during development.

Next to that, the developer is obviously responsible for placing the Workflows in the Framework, configuring the correct exception handling, etcetera.

## Data Flow Explanation

In the Ciphix Performer template, the data is passed through three steps:

-   Get the Transaction item
-   Processing the Transaction
-   Updating the Transaction status

**Get the Transaction item**

By default, data is collected from the Orchestrator Queue. This is done in the ‘Get Transaction Data’ state. In this state, a queue item is retrieved from the Orchestrator. If no queue item is found, out_TransactionItem is set to Nothing.

**Processing the Transaction**

The item retrieved in ‘Get Transaction Item’ is taken to the ‘Process Transaction Item’ state were it is used to execute the process. The Specific Content values are used to indicate specific attributes for the tansaction. Any systemErrors or BREs that come up during this stage are caught by the TryCatch around this workblock. The framework also allows for tracking of the progress of a queue item. In the situation that a queue item did not progress through the entire process, the framework will track at which process step the queue item has left off and should continue next time.

**Updating the Transaction status**

In the 'Finally' of the TryCatch, the transaction status (Successfull, Failed, BRE) is communicated to the orchestrator. In case of a failed item, the robot will check if a retry of that item is desired. If this is the case, a new item is created in the queue. If this is not the case, the process returns to 'Get Transaction Item'.

## DirectReporter

**Retrieve Queue Data**
-   Using an Orchestrator HTTP Request, all information concerning the queue items processed in the last run is gathered and stored in Data Tables.
-   A check is performed on whether there are any queue items processed or not. If there are queue items returned, the process continues to the Get statistics, filter and store data state where the data is processed to fit the report. If there are no queue items returned, it passes that to the Send Data state directly, where a mail is send with the corresponding information.

**Get statistics, filter and store data**
-   The data retrieved in the Retrieve Queue Data state is transformed and stored in an Excel sheet that is saved in the Data/Reports folder.
-   The data for each queue item can be filtered using the custom filter activities in the second step.
-   The Report Excel sheet and potential Exception Screenshots are added to a collection, which is included in the final mail.

**Send Data**
-   The data prepared in the previous states is used in the creation of the report and mail. A mail is created that is more visual and descriptive than before. Three types of HTML templates are created for this purpose. Based on the result of the performer, one of the three types is chosen by the robot.
-   By default, Outlook activities are added to the framework to send the mail. Change this according to your client's environment.

## Tips & Tricks

-   Don't change or remove activities/workflows already present in the framework.
-   Take a look at the state transitions to see what variable conditions the various transitions are depending upon.
-   It is possible to add a "Failure" workflow, which is only executed when the maximum number of Queue retries is reached, e.g. archiving or sending an e-mail.
-   Make sure all arguments in each invoke workflow are linked.
-   Make sure to log at the beginning and end of each workflow.
-   Enter the items in the Config file that are marked red. These are mandatory fields for the framework to work.
-   Make sure to check which mail protocol the client in question uses and change the mail activity in the Direct Reporter and the assets accordingly.
-   Make sure to refresh the Orchestrator Resources in Studio. This makes sure the robot knows in which Orchestrator Folder the queue/assets are stored.

* * *

# Contributing

Other users and developers can contribute to make the Ciphix frameworks better by submitting their ideas to the [RPA Service Desk](https://forms.monday.com/forms/0472ebf27cb464446472d917bb83ceb7?r=use1).

MIT License

Copyright 2019 UiPath

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.