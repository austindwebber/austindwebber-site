---
layout: post
title: Mass Java Removal & OpenJDK Installation
image: /img/java.png
---

In this blog post, I'll discuss my strategy created to remove existing versions of Java from workstations and reinstalling OpenJDK (Amazon Corretto). Oracle changed their license agreement, and some of their users were no longer licensed for Java. The first element of this script was closing open applications that were using Java. Since Internet Explorer uses Java for web plugins, I first start by killing the Internet Explorer process:

```
Get-Process -Name *iexplore* | Stop-Process -Force -ErrorAction SilentlyContinue
```

Next, I'll kill any process that is using the primary Java process, which contains "Java" in the name:

```
Get-WmiObject win32_process | Where-Object {$_.ExecutablePath -like "*Java*"} | Select-Object @{n='Name';e={$_.Name.Split('.')[0]}} | Stop-Process -Force -ErrorAction SilentlyContinue
```

I then ended up defining a filter containing all applications matching Java and not matching Auto. This is so it skips over the Java auto updater since it should be removed by uninstalling Java anyway. I also created a variable to store the locations I wanted to search in the registry as I needed to remove x86 and x64 Java:

```
#Locations to search
$UninstallLocations = @('HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall','HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall')

#Select files you want to uninstall
$UninstallFilter = { (($_.GetValue('DisplayName') -match 'Java') -and ($_.GetValue('DisplayName') -notmatch 'Auto')) }
```

After defining those variables, I needed to use a loop to search through each location to find applications that meet the filter's criteria. During this loop, I would grab the display name of the software to display it before uninstalling, and the GUID of the application. I created a string to apply the standard msiexec silent uninstallation strings to that GUID. I was then able to execute msiexec and uninstall the software. After uninstallation, I removed that GUID's folder from the registry in case it failed uninstallation. This was mostly just to make sure everything was removed.

```
ForEach ($Path in $UninstallLocations) {
if (Test-Path $Path) {
    Get-ChildItem $Path | Where-Object $UninstallFilter | ForEach-Object { #Uninstallation
        $name = ($_.GetValue('DisplayName'))
        Write-Output "Uninstalling $name"
        $switches = $("/qn /x " + $_.PSChildName)
        Start-Process "$env:SystemDrive\Windows\System32\msiexec.exe" -Wait -ArgumentList "$switches"
        Remove-Item $_.PSPath -Force -Recurse -ErrorAction SilentlyContinue
    } 
}
}
```

Great, so now I've removed all versions of Java and their installation properties from the registry. However, if a few of them failed to uninstall, it may leave behind unnecessary files. I created another filter and location variable to remove anything with Java in its name from the installed products:

```
#Removal Filter
$RemovalFilter = { ($_.GetValue('ProductName') -like "*Java*") }

#Path to look through
$Path = "Registry::HKEY_CLASSES_ROOT\Installer\Products"

#Remove Java from Registry
Get-ChildItem $Path | Where $RemovalFilter | ForEach {
    Remove-Item $_.PSPath -Force -Recurse
}
```

So now I've removed any chance of it being installed. I added a few extra lines to this script to remove any JavaSoft registry keys and all remaining folders/files from "C:\Program Files (x86)\Java" & "C:\Program Files\Java":

```
#Remove extra registry values
if (Test-Path "HKLM:\Software\JavaSoft") {
    Remove-Item "HKLM:\Software\JavaSoft" -Force -Recurse -ErrorAction SilentlyContinue
}

#Remove extra files
if (Test-Path "$env:ProgramFiles\Java\") {
    Remove-Item "$env:ProgramFiles\Java\" -Force -Recurse -ErrorAction SilentlyContinue
    Remove-Item "${env:ProgramFiles(x86)}\Java\" -Force -Recurse -ErrorAction SilentlyContinue  
}
```

Now, my removal script is complete. I'll briefly cover installing Amazon Corretto (OpenJDK) as a workaround. OpenJDK doesn't support web plugins, so it only needs with cooperate with other installed applications. Generally, applications using Java will search environment variables and for registry keys. Amazon provides MSIs for the JDK, but not for the JRE. You can download the JRE/JDK [here](https://docs.aws.amazon.com/corretto/latest/corretto-8-ug/downloads-list.html). I ended up creating the following script to install the JRE:


```
#Create required folder
if (!(Test-Path "C:\Program Files\Java")) {
    New-Item -ItemType Directory "C:\Program Files\Java"
}

#Move over files
Copy-Item "$PSScriptRoot\jre1.8.0_212\" "C:\Program Files\Java" -Recurse -Force

#Set variables
[Environment]::SetEnvironmentVariable("Path","$env:PATH;C:\Program Files\Java\jre1.8.0_212\bin",[System.EnvironmentVariableTarget]::Machine) | Out-Null
[Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Java\jre1.8.0_212", "Machine") | Out-Null

#Create Registry Keys
reg import "$PSScriptRoot\values.reg"
```

As you can see, I copy over the files, add it to the PATH variable, set the JAVA_HOME variable, and import registry keys. These are the registry keys I found the most useful to import:
```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\JavaSoft\Java Runtime Environment]
"CurrentVersion"="1.8"

[HKEY_LOCAL_MACHINE\SOFTWARE\JavaSoft\Java Runtime Environment\1.8]
"RuntimeLib"="C:\\Program Files\\Java\\jre1.8.0_212\\bin\\server\\jvm.dll"
"JavaHome"="C:\\Program Files\\Java\\jre1.8.0_212"
```

If you're unfamilar with registry imports, you can paste this text into a file and save it with a .reg extension to be imported with PowerShell.

Now that it is installed, it is ready for use in your environment. You'll need to update these registry keys and overwrite existing variables/files if you are going to patch it.

I hope you found this useful. Thank you for reading.
