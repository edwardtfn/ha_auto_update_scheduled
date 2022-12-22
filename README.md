# Home Assistant - Auto update (scheduled)

## What is this?
A Home Assistant blueprint to assist keeping the system automatically updated with the latest version of it's multiple components.

I’ve created a new version of my previous [Auto-update Home Assistant blueprint](https://community.home-assistant.io/t/auto-update-home-assistant/429015) , but now based in the new [schedule helpers](https://www.home-assistant.io/blog/2022/09/07/release-20229/#new-helper-weekly-schedule) and other improvements. I’m using this blueprint myself.

I understand Home Assistant wasn't considered a stable system for years, 

I still think you should understand the benefits and risks of auto-updating, so I keep suggesting you to take a look on those discussions:

- [How to keep everything automatically up to date? - Configuration - Home Assistant Community (home-assistant.io)](https://community.home-assistant.io/t/how-to-keep-everything-automatically-up-to-date/288938)
- [Auto Update feature for Core/Supervisor - Feature Requests - Home Assistant Community (home-assistant.io)](https://community.home-assistant.io/t/auto-update-feature-for-core-supervisor/412783)
- If you don’t want an auto-update, you might like this blueprint for notifications of new updates: [Update notifications (Core, OS, Addons, HACS, etc) - Blueprints Exchange - Home Assistant Community (home-assistant.io)](https://community.home-assistant.io/t/update-notifications-core-os-addons-hacs-etc/409161)

Please provide feedback. This is based on my other blueprint, just changing the items related to the schedule, so I believe the update mechanism itself is quite well tested, but you know, there are always new ideas or new use cases that we never though about, so let me know your ideas, issues and concerns… :relaxed:

## Important notes
### Supervisor update
The update of HA Supervisor is always set to auto-update by the system, so it will be updated out of this blueprint anyways (and the time of that update is not controlled by this blueprint). Please take a look into the following discussions around this:
- [Feature request: block supervisor auto-updates](https://community.home-assistant.io/t/feature-request-block-supervisor-auto-updates/112743)
- [How to stop supervisor auto-update?](https://community.home-assistant.io/t/how-to-stop-supervisor-auto-update/132271)
- [How to stop supervisor's auto update?](https://community.home-assistant.io/t/how-to-stop-supervisors-auto-update/173017)
### Add-ons auto-update on the UI
All the standard add-ons of Home Assistant supports to set “Auto-update” thru the UI. If that is enabled, this blueprint cannot guarantee the time of running the updates for that specific add-on (as it is controlled outside this blueprint).
### HACS update
At this moment (2022-09-10), HACS updates are not released, but it is an experimental feature, so, if you enable HACS to use its experimental features, this blueprint will include HACS updates automatically.
### Schedule helper
You will need a [schedule helper](https://www.home-assistant.io/blog/2022/09/07/release-20229/#new-helper-weekly-schedule) indicating the time slots where the updates could start.
- Schedule documentation: [https://www.home-assistant.io/integrations/schedule](https://www.home-assistant.io/integrations/schedule)

The updates will be installed one by one, while inside the schedule windows. It’s not guaranteed that the update will finish inside the schedule windows.
If one of the updates forces HA to reboot, the automation will restart after the reboot and will resume the updates of all the remaining items while in the schedule windows.

## Installing
### Import blueprint tools in Home Assistant (easier)
Just click in the following button and follow it's steps:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fedwardtfn%2Fha_auto_update_scheduled%2Fmain%2Fauto_update_scheduled.yaml)

### Manual
1. Open Home Assistant interface
2. Using the ***File editor*** (or ***Studio Code Server***), navigate to the `/config/blueprints/automation`, right click and then click on ***New Folder...***:
![image](https://user-images.githubusercontent.com/94725493/193560317-750532bf-545e-4392-be94-9f6d2be79305.png)
3. Type a name for your new folder
4. Right click your new created folder and select ***New file...***, then name your file (make sure to end with `.yaml`).
![image](https://user-images.githubusercontent.com/94725493/193563810-d6dc01b2-1476-432c-9c6a-aa0cd07eac58.png)

5. Go to the edit window and past the content of this blueprint.
6. Go to ***Developer tools*** and select the tab ***YAML***.
7. Click on ***Automations*** and wait until it is reloaded.
![image](https://user-images.githubusercontent.com/94725493/193564392-19875a27-72bf-425d-8883-763c8752e4ea.png)

## Creating the new automation
1. Go to ***Settings*** and ***Automations & Scenes***
2. Click on ***+ Create automation*** button (bottom-right)
3. Click on the ***Use a blueprint*** box to open the list of blueprints: 
![image](https://user-images.githubusercontent.com/94725493/193566186-dcfa2325-18ed-43fc-962b-31c2474ebedf.png)
4. Select the new blueprint from the list

![image](https://user-images.githubusercontent.com/94725493/193566680-6668af94-ff6c-412b-8c93-a06c7268a1b5.png)
5. Enter the inputs and hit ***Save*** (See Setting up)

## Setting up
The following inputs are supported by this blueprint:
### Create a full backup before start the updates?
Select it if you want to trigger an additional backup before starting the updates.

### Force Home Assistant host to restart if required by any update? <sup>*Required</sup>
This won't affect updates where a restart is automatic, but for updates requiring a manual restart (quite common on HACS) this blueprint can automatically force a restart by the end of the updates.

### Schedule entity <sup>*Required</sup>
You can create an Schedule under Settings > Devices & Services > Helpers.
- The schedule windows will define when an update will start. It is possible that a backup, an update or a restart process finishes after the schedule window, but new updates won't stars outside the schedule windows.
- For more information about Helpers, please refer to the [Home Assistant docs](https://www.home-assistant.io/integrations/schedule).

### Earliest day in the month to update Home Assistant <sup>*Required</sup>
Usually a new major version of Home Assistant is available on the begining of every month. Some people consider those release as not stable enough and prefer to avoid those versions, not updating the system until the mid of the month (day 15).
- If you select a day higher than 28 the updates won't run every month.

 ### Exclusions <sup>*Optional</sup>
Select the items that should NOT be included on the automated updates.
* Use this if you want to keep some add-on on an specific version and avoid auto-updates to it.
      
### Pre-update actions <sup>*Optional</sup>
Actions to execute before the backup or any update starts.
You can use this to send notifications, turn on/off devices or activate scenes before starting the updates.
* Please be aware that all actions will run right before the update process, which can happens over-night. Take this in account when selecting your actions.
* The variable `{{ pending_updates }}` is available for your automations and contains the list of pending updates.

### Post-update actions <sup>*Optional</sup>
Actions to execute AFTER the update process finishes.
You can use this to send notifications, turn on/off devices or activate scenes after applying  the updates.
* Please be aware that all actions will run right before the update process, which can happens over-night. Take this in account when selecting your actions.
* The variable `{{ pending_updates }}` is available for your automations and contains the list of pending updates.
* **IMPORTANT!** Some updates will automatically restart Home Assistant, causing the automation to interrupt before finishing, preventing the pos-updates actions to be executed. If you have critical actions to run after an update, consider including also in another automation based on Home Assistant start.

### Pause update entities <sup>*Optional</sup>
You can select one or more entities to pause the updates. If any of the selected entities is "On" or "True" the system won't be updated on the schedule time.
You can use this to hold your updates when you have a party at home, or when you are on vacations and don't want to be concerned about updates on Home Assistant.


![image](https://user-images.githubusercontent.com/94725493/193578971-5981aa3b-06bc-4ceb-ae2a-f3149d8ad350.png)

### Changes log
Please take a look on the [log of commits](https://github.com/edwardtfn/ha_auto_update_scheduled/commits/main).
