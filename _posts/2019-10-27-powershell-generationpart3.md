---
layout: post
title: Learning to Automate the Creation of PowerShell Scripts Part 3 (WinForms GUI)
image: /img/ps.jpg
---

## Learning to Automate the Creation of PowerShell Scripts

After designing my GUI with WPF, I realized that it wasn't multi-threaded and it felt like an application created in Windows XP. I decided to look into other options for improving it, and decided to use PowerShell Studio. I re-created my entire GUI, but was able to remove a bunch of excess code. 

PowerShell Studio would then handle the implementation of the GUI components. If you've never heard of it before, you can check it out [here](https://www.sapien.com/software/powershell_studio). It's quite expensive to purchase a license for it, but there is a free trial for testing. It allowed me to generate installation files and source code after designing the GUI within the IDE. Compared to my previous method of editing GUIs, I don't need to worry about creating objects for the GUI elements. As far as handling button clicks and other actions, it's very similar to what I was working on in part 2. Here's my generation button handler:

```
$buttonGenerate_Click = {
	if ($checkboxCopyShortcutIntoPubl.Checked -eq $false) { $textBoxPublicDesktop.Text = "" }
	if ($checkboxCopyShortcutIntoStar.Checked -eq $false) { $textboxStartMenuShortcut.Text = "" }
	if ($checkboxCopyAFolderFile.Checked -eq $false)
	{
		$textboxCopyFileFolder.Text = ""
		$textboxDestinationPath.Text = ""
	}
	if ($checkboxAddExtraCode.Checked -eq $false) { $textboxExtraCode.Text = "" }
	if ($textboxFile.Text -match '.msi') { $productCode = getMSIData -Path $textboxFile.Text -Property ProductCode }
	if ($checkboxGenerateTestScript.Checked -eq $true) { $generateTest = "1" }
	generateScript -installationFileLocation $textboxFile.Text -switches $textboxSwitches.Text -installationFileName $textboxApplication.Text -generateTestScript $generateTest -productCode $productCode -desktopIconName $textBoxPublicDesktop.Text -startMenuShortcut $textboxStartMenuShortcut.Text -copyOverFile $textboxCopyFileFolder.Text -copyIntoDirectory $textboxDestinationPath.Text -extraCode $textboxExtraCode.Text
	
	
	
	#Output to console in GUI
	$currentTime = Get-Date -format g
	$appName = $textboxApplication.Text
	$installationLocation = $textboxFile.Text | Split-Path
	
	$richtextbox2.AppendText("$currentTime Created an installation script for $appName in $installationLocation")
	$richtextbox2.AppendText(".`n")
}
```

WinForms is an older GUI library compared to WPF, but is modern enough for my application. I used PowerShell Studio's built in GUI designer to design my application. After adding extra features and re-creating the GUI, this is what I ended up with:

![Finished GUI](https://i.imgur.com/TxWWfUI.png)

I ended up adding textboxes to display the output from script generation and installation file information. Instead of the application closing after a single use, it is now more like a utility or tool. The additional options to add pre-generated code or add-ons are now split into individual tabs:

![Additional Code Tab](https://i.imgur.com/HpeLWuw.png)

![Addons Tab](https://i.imgur.com/gEN3F52.png)

I then added a progress bar at the bottom of the GUI, which shows detailed progress during generation. I added additional features like a GUI converter to convert .PS1 files to .EXE for direct execution. Also, I'm in the process of implementing an installation switch tester. My thought is to allow users of the program to test their installation switches and the program would complete temporary generation. After the user verifies those are the installation switches that are working, the program would then delete the old temporary versions and generate a fresh copy.

Click on this image to view a demo of the current program:

[![PSDevelopmentTool Demo](http://img.youtube.com/vi/cDJq3X_VR58/0.jpg)](http://www.youtube.com/watch?v=cDJq3X_VR58)

As the video shows: you can generate your installation script, test the script, delete extra generated files, and then convert it to an exe. At this point, I'm now adding additional features that I've thought of and refactoring my code. You can find the source code, installer, and portable version on my GitHub [here](https://github.com/austindwebber/ps-development-tool).

Thank you for reading.