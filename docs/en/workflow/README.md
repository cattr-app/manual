# Workflow  :id=intro :priority=7

Work with Cattr can be described in the following sections.

## Create project  :id=project

Projects contains and groups task inside them, which makes it easier to make the according reports and to control access to the related tasks.

!> Projects can only be created by users who have role `manager`

To create a new project, go to the `Projects` section and click `Create`.

![Create project](../../assets/en/workflow/project_create.png)

Once you add the project's name and description, click `Save`.

?> `Important` option lets you save the project's screenshots when cleaning up the storage if the server runs out of space

![Save project](../../assets/en/workflow/project_save.png)

Once the project is created, you can create tasks for it, and assign user for those.

?> You can lear how to assign managers for the projects, who are able to create and assign tasks in [Roles](en/roles/?id=promote) section

## Create task  :id=task

Projects contains the according tasks with the linked time intervals, displaying the work around the tasks.

!> Tasks can only be created by users who have role `manager` or `auditor`

To create a new task go to the `Tasks` section and click `Create`.

![Create task](../../assets/en/workflow/task_create.png)

Once you add the task's name, description and priority, you'll be able to select a task's project from the dropdown menu. The project's name field will dynamically suggest you the found projects. After that, you can assign the user to the task and save it, clicking `Save`.

?> `Important` option lets you save the task's screenshots when cleaning up the storage if the server runs out of space

![Save task](../../assets/en/workflow/task_save.png)

Once the task is added, the assigned user will be able to track the task's time.

?> Projects and tasks can be created automatically, if you add one of the [integrations](en/integrations/) to Cattr

## Client application  :id=tracker

In order to go through authorization process you'll need to use your Cattr instance's hostname, and user account's email and password.

Once you log in, you'll see the task list that are assigned to you, and how much time you've worked today.

If you click the task's name, you'll see its description. To start tracking time, you have to click the time button, located next to the task. In order to stop tracking time, you can click the time button next to the current task, or click the main time tracking button which is located in the client window's bottom part.

## Manual time addition  :id=manual-time

Users can add time spent both with the client app and control panel.

!> By default users can't manually add time via control panel. You can learn more about it on the [Access rules](en/roles/?id=manual-time) page

To add the time spent you'll need to go to the page `Dashboard` and select the `Add time` section.

![Add time](../../assets/en/workflow/time_create.png)

Once you add the user, task, and time interval borders, click `Save`.

![Remove time](../../assets/en/workflow/time_save.png)

?> Manually created time intervals will be shown in a different color on page `Dashboard` in sections `Personal` and `Team`.
 
![Intervals](../../assets/en/workflow/time_manual.png)

# Disable screenshots :id=screenshots

Global selector can now be set not only in settings, but also in environment variables. If it is set in environment variables, it is not available to change in the interface (the value is shown, but even the administrator cannot change it).

If it's disabled, then:
- the link to the ‘Screenshots’ section in the top menu has been removed,
- when clicking on working time in dashboard, the screenshot stops popping up over the interval,
- in Project report we see a normal ‘date-time’ list instead of an expanding list of screenshots against the date.

The settings to disable screenshots for the administrator role are displayed as follows:

- if the screenshot setting is set to ’Required’ at the global level, the administrator sees an inactive selector,
- if screenshots are set to ‘Optional’ at the global level, the administrator can make them mandatory for the user or disable them forcibly for the user.
- If the screenshots setting is set to ‘Forbidden’ at the global level, the administrator sees an inactive selector that says ‘disabled’ and screenshots are forbidden for the entire company.

![Interface to disable screenshots in company settings for the admin role](../../assets/en/workflow/screenshot_status_en.png)

The settings for disabling screenshots for the User role are displayed similarly, but the user does not have the ‘Optional’ option and inherits the administrator and global settings.
