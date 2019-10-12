---
layout: post
title: Learning to Automate the Creation of PowerShell Scripts Part I (CLI)
image: /img/ps.jpg
---

## Learning to Automate the Creation of PowerShell Scripts Part I (CLI)

When I was first introduced to packaging applications, I was creating Batch files and already knew most of the basic commands. I decided to begin learning PowerShell since it was increasing in popularity. I watched a few tutorials on Lynda (LinkedIn Learning) and learned the basic syntax.

In an environment where you need to manage a signficant amount of scripts, you'll begin to notice that a significant amount of time is taken creating basic PowerShell scripts. While creating scripts from scratch, I tended to use the same basic formatting for commands:

```
Start-Process "$PSScriptRoot\your_applicaton.exe" -Wait -ArgumentList "/your-installation-switches"
```

I decided to create a program to generate scripts with dynamic comments, standard installation file execution format, and the ability to add-in other common commands. Here's where I started.

I originally started with a basic command line interface. It grabbed a list of the current installation files detected in the directory (**.exe, and .msi**) and allowed the user to select their installation file they wanted to package from the list.

![File Information](https://i.imgur.com/WYcyu0r.png)

The following allowed me to provide a list to the user with indexes:
	
```
Get-ChildItem .\* -Include *.exe, *.msi -Exclude "ScriptGenerator.exe" | Select-Object -Property Name | ForEach-Object {"Index "+ "$itemList" + ": " + $_.Name; $itemList+=1} | Format-List
```
    
I then needed to provide a way to allow users to select a file from the list. I ended up creating a short function to open a built-in GUI for selecting files in the current working directory. Because remember, this gathers its list of installation files from the current working directory. 

![Select File](https://i.imgur.com/flPeOUQ.png)

Here's a short function I created that utilizes Out-GridView:


```
#Function that allows user to select installation file
function getCWDInstallationFile {
    param(
    [Parameter(Mandatory=$true)]$Include,
    [Parameter(Mandatory=$true)]$Exclude
    )
    
#Data fields
$installationFile = ""
$installationFileLocation = ""

#Prompt user to select a file
Write-Host "`nPlease select the installation file." -ForegroundColor Yellow $installationFile = @(Get-ChildItem .\* -Include $Include -Exclude $Exclude | Out-GridView -Title "Choose your installation file" -OutputMode Single)
        
#Return the location of the installation file
Return $installationFile
}
```

As you can see, it's similar to the command I showed earlier, but in a nicer GUI format.

In this example, I selected an .MSI as my installation file. It's useful to have more information about your installation file like the product code, which is a unique identifier to determine if a specific product is installed:

![Installation File Info](https://i.imgur.com/dC9uXtU.png)

I created this function to allow ease of making sure you're packaging the right program, and being able to assist with auto-generating uninstall scripts for MSIs. Since you can directly uninstall MSIs based off of product code using MSIEXEC; it's a fairly simple generation process. You can find an example of how to extract data from MSI files [here](https://www.scconfigmgr.com/2014/08/22/how-to-get-msi-file-information-with-powershell/).

I also gathered information from EXE files using Select-Object:
```
    $productDescription = (Get-ChildItem $installationFile).VersionInfo | Select-Object -ExpandProperty FileDescription
    $productVersion = (Get-ChildItem $installationFile).VersionInfo | Select-Object -ExpandProperty FileVersion
    $productManu = (Get-ChildItem $installationFile).VersionInfo | Select-Object -ExpandProperty CompanyName
```
Less useful than product codes for MSIs, but still useful nonetheless:

![Installation File Info](https://i.imgur.com/5vHN5Fy.png)

At this point in the process, I prompted the user to input installation switches to complete script generation. If nothing
was entered for input, whether it was for an MSI or EXE, I applied some default switches for each type respectively: "/qn /norestart" or "/s". After a user verifies their installation switches are correct, I then ask what the application should be named (In order to create a unique name for the script).

The script is now ready for generation. I added in some additional functions to generate extra code.

###### Add a shortcut to the public desktop:

```
Write-Host -ForegroundColor Yellow "`nWould you like your script to add a shortcut onto the Desktop? (You must have it in the current folder.)"
$desktopIcon = Read-Host "(Y/N)"

if ($desktopIcon -eq "y") {
    Write-Host -ForegroundColor Yellow "What is the name of the Desktop icon? (including the extension)"
    $desktopIconName = Read-Host "Name"      
}
```

###### Add a shortcut to the start menu:

```
Write-Host -ForegroundColor Yellow "`nWould you like your script to add a shortcut into the Start Menu? (You must have it in the current folder.)"
$startMenu = Read-Host "(Y/N)"

if ($startMenu -eq "y") {
            
    #Ask for the name of the shortcut
    Write-Host -ForegroundColor Yellow "What is the name of the shortcut? (including the extension)"
    $startMenuShortcut = Read-Host "Name" 

}
```

###### Copy a specific file/folder:
```
Write-Host -ForegroundColor Yellow "`nWould you like your script to copy a file/folder into a directory? (You must have it in the current folder.)"
$copyFile = Read-Host "(Y/N)"

if ($copyFile -eq "y") {
            
    #Ask for the name of the file/folder
    Write-Host -ForegroundColor Yellow "What is the name of the file/folder you would like to copy over? (including the extension if it is a file)"
    $copyOverFile = Read-Host "File/Folder Name" 

    #Ask for the name of the directory
    Write-Host -ForegroundColor Yellow "What is the location of the directory you want to copy into? (Ex: C:\Program Files\Firefox)"
    $copyIntoDirectory = Read-Host "Directory Location" 

}
```

After the user decides to add any of these, it begins to generate the script and closes out of the utility. In the current working directory, it generates a .PS1 file named after the application name. In this example, it created a file called "install_GIT.ps1". Here are the contents of the file generated:
```
#Silently installs GIT
#Script Auto-Generated by Admin
#Date Created 10/11/2019 19:23:04
            
#Silently installs GIT
Start-Process "$PSScriptRoot\Git-2.23.0-64-bit.exe" -Wait -ArgumentList "/s"
```

As you can see, this doesn't have any add-ins, but includes auto-generated comments with username and time generated. It will run the Git installer when placed in the same directory. It automatically supplied an installation switch, since I didn't provide any in this example. 

I've adapted code over a significant period of time, and transitioned into different GUI models. To be discussed in my next blog posts. Thank you for reading.



