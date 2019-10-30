---
layout: post
title: Handling Adobe CC SDL Updates
image: /img/adobe.png
---

## Handling Adobe Creative Cloud App Updates

Since Adobe was changing their licensing structure to no longer allow serial based licensing, I needed to shift to shared device licensing. After installing the latest version of Adobe Creative Cloud 2019 in a shared device licensing environment, I had to come up with a solution to manage patches for individual products. Otherwise, each version of individual products would be different depending on when it was installed. This would cause compatibility issues with projects between different computers. In the past, an Adobe RUM server could be setup to handle and distribute updates to all devices. I decided to create my own solution instead to install on each computer to setup automatic updates, otherwise a user would need to initiate updates themselves. 

Make sure that you check the box for "Enable Remote Update Manager"  when creating your package if you're going to implement this solution:

![Adobe SDL RUM](https://i.imgur.com/WWZ5m6C.png)

Adobe's documentation states that you can invoke the update manager by calling the executable here:
```
C:\Program Files (x86)\Common Files\Adobe\OOBE_Enterprise\RemoteUpdateManager.exe
```

So now I just needed to write a PowerShell script to execute this. I ended up creating a task in Task Scheduler in order to execute this during a maintenance window. Instead of creating the task with PowerShell directly, I ended up creating a task through the GUI and importing it since it provided more options.

I created a new task, and under the "General" tab I specified the following settings:

![General_AdobeTask](https://i.imgur.com/fGkBUJN.png)

I specified the user account as SYSTEM, so connectivity doesn't necessarily matter. It also won't interrupt the user if it's running as SYSTEM. Note that when adding the user account as SYSTEM, you need to be running Task Scheduler as administrator otherwise it won't work. When specifying the account, you may need to enter NT AUTHORITY\SYSTEM rather than just SYSTEM as I had issues with that. You'll also need to run with highest priviledges since it requires administrator rights to execute the remote update manager. Next, for the trigger:

![Trigger_AdobeTask](https://i.imgur.com/bWUlesF.png)

I set it up to trigger daily around 4:30 AM since my devices were going to be in maintenance mode during that time. I also checked the box to stop if the task runs longer than two hours. This was just so it had enough time to finish depending on how many updates. Next, for the actual action:

![Actions_AdobeTask](https://i.imgur.com/7Tskp6O.png)

I needed to execute the Adobe Remote Update Manager, but also tell it to install all updates for every application. Here's the arguments that I specified to CMD.exe:
```
/C "C:\Program Files (x86)\Common Files\Adobe\OOBE_Enterprise\RemoteUpdateManager\RemoteUpdateManager.exe" --action=install
```

CMD.exe /C starts a new instance of Command Prompt and I specified --action=install to tell Adobe Remote Update Manager to download and install everything. Now for the Conditions and Settings tabs:

![Conditions_AdobeTask](https://i.imgur.com/FliYJmg.png)
![Settings_AdobeTask](https://i.imgur.com/EAUWgOF.png)

These customizations are really up to you and how it will be setup in your environment. Finally, I finished creating this task and right-clicked to export the task as an .XML file. I then found a command to import the task by calling schtasks.exe via PowerShell:

```
schtasks.exe /create /XML "$PSScriptRoot\AdobeUpdate.xml" /tn "AdobeUpdate"
```

I added it to my distribution point and used the following to detect if it was installed:
```
C:\Windows\System32\Tasks\AdobeUpdate
```

That is how I handled automatic updates for Adobe Creative Cloud, and I hope it was useful for you.

Thank you for reading!


