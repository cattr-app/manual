# Cattr Integration with Access Control Systems

These instructions demonstrate how to use Cattr as a library for integration with access control systems in an enterprise, enabling automatic tracking of employee work time.

## Use Case Scenario

A company uses an access control system that registers employee entry and exit times. The goal is to integrate this system with Cattr to automatically track work time, verifying employee presence with photos from surveillance cameras.

## Solution Architecture

The solution consists of three main components:

1. **Access Control System:** Registers employee entry/exit events and provides time data and photos from cameras.

2. **Integrator (Middleware Application):** Handles the interaction between the access control system and the Cattr API.

3. **Cattr (Used as a Library):** Provides the API for creating tasks, time intervals, and uploading screenshots used for time tracking.

## Integrator Functions

* **Cattr Authentication:** Obtains an access token for the Cattr API.
* **Project and Task Data Retrieval:** Identifies the project and task in Cattr for time tracking.
* **User Data Retrieval:** Maps employees in the access control system to Cattr users.
* **Entry/Exit Event Handling:** Reacts to employee entry and exit events.
* **Task Creation:** Creates a task in Cattr when an employee enters.
* **Time Interval Creation:** Creates time intervals (e.g., every 5-10 minutes) based on presence data.
* **Screenshot Upload:** Uploads photos from surveillance cameras to Cattr, associating them with time intervals.


## Cattr Setup

1. **User Creation with Permissions:** Create a user in Cattr with permissions to create time intervals for other users.
2. **Project Creation:** Create a project for work time tracking.
3. **User Creation:** Create users in Cattr corresponding to company employees.

## Cattr as a Library

In this scenario, Cattr acts as a library, providing an API for external interaction. The integrator uses this API to implement the necessary functionality without delving into Cattr's internal logic.

Key characteristics of using Cattr as a library:

* **API Provision:** Interaction occurs exclusively through the API.
* **Modularity and Abstraction:** The time tracking functionality is used as a separate module.
* **Reusability:** The Cattr API can be used in other applications.
* **Supporting Role:** Cattr provides services used by the integrator, rather than being a standalone solution.


## Cattr API Endpoints

* Authorization: https://api.docs.cattr.app/#api-Auth-Login
* Project List: https://api.docs.cattr.app/#api-Project-GetProjectList
* User List: https://api.docs.cattr.app/#api-User-GetUserList
* Task Creation: https://api.docs.cattr.app/#api-Task-Create
* Time Interval Creation: https://api.docs.cattr.app/#api-Time_Interval-Create
* Screenshot Upload: https://api.docs.cattr.app/#api-Screenshot-Create
