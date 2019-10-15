---
layout: post
title: Learning to Automate the Creation of PowerShell Scripts Part 2 (WPF GUI)
image: /img/ps.jpg
---

## Learning to Automate the Creation of PowerShell Scripts

After creating a CLI utility for generating scripts, I decided to research creating GUIs with PowerShell. It's not "built into" PowerShell, but rather just writing a PowerShell script that utilizes .NET graphical commands. I started to begin by creating a GUI with WPF. While working on this, I also refactored some of the code. 

This isn't meant to be a tutorial, but what I've created and learned throughout the process. If you'd like a detailed tutorial, visit [here](https://foxdeploy.com/2015/04/10/part-i-creating-powershell-guis-in-minutes-using-visual-studio-a-new-hope/).

I setup Visual Studio, created a project, and I began to add buttons/textboxes. I made a fairly simple GUI, as only needed a few inputs from the user. In my previous command line utility, I only gathered the exe/msi files from the current directory. With the ease of a GUI, I then added a button to open a file browser allowing the user to select a file from any location. The user was then able to enter in installation switches into a simple textbox.

![Visual Studio GUI](https://i.imgur.com/V1wyvfV.png)

If you recall my previous blogpost, I also had special add-ins to allow: adding of a start menu shortcut, adding of a public desktop shortcut, and copy a folder/file. I also added the ability to add extra code, since many times you need to add unique code to a script. Finally, I created a button to allow generation. I wanted to make sure the user didn't click the button before generation, as that would break the utility. I set the status of the button to disabled, and setup code to enable it after certain fields were filled out.

Now that we have a GUI created, let's dig into the code implementation. The problem about this type of GUI setup is you can easily design it in Visual Studio, but after implementing, you need to keep copying the code back into your PowerShell script.

After designing it, you need to export your XAML from the GUI. Here is the code to setup my GUI:

```
#XAML Input
$inputXML = @"
<Window x:Name="Script_Generator_Menu" x:Class="ScriptGeneratorGUI.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ScriptGeneratorGUI"
        mc:Ignorable="d"
        Title="Script Generator" Height="574.07" Width="585.034" Background="Black" FontFamily="Segoe UI Light" FontSize="16">
    <Grid Margin="0,0,-8,0" Background="#FF851E1E">
        <Grid.RowDefinitions>
            <RowDefinition Height="131*"/>
            <RowDefinition Height="50*"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="0*"/>
            <ColumnDefinition Width="0*"/>
            <ColumnDefinition Width="0*"/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <TextBlock x:Name="IntroText" HorizontalAlignment="Left" Margin="14,10,0,0" TextWrapping="Wrap" Text="Script Generator " VerticalAlignment="Top" Height="22" Width="507" Foreground="White" Grid.Column="3" FontFamily="Segoe UI Black"/>
        <Label x:Name="InstallationFile_Label" Content="Select your installation file:" HorizontalAlignment="Left" Margin="10,40,0,0" VerticalAlignment="Top" Foreground="White" Grid.Column="3" Height="31" Width="211" FontFamily="Segoe UI Semibold"/>
        <Label x:Name="Switch_Label" Content="Enter your installation switches:" HorizontalAlignment="Left" Margin="10,70,0,0" VerticalAlignment="Top" Foreground="White" Grid.Column="3" Height="31" Width="247" FontFamily="Segoe UI Semibold"/>
        <TextBox x:Name="InstallationFile_TextBox" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="27" Margin="285,45,0,0" VerticalAlignment="Top" Width="164" Background="#FFE5E5E5" IsReadOnly="True"/>
        <TextBox x:Name="Switches" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="27" Margin="285,75,0,0" TextWrapping="Wrap" Text="" VerticalAlignment="Top" Width="250" Background="#FFE5E5E5"/>

        <CheckBox x:Name="StartMenu_CheckBox" Grid.ColumnSpan="4" Content="Add Start Menu shortcut?" HorizontalAlignment="Left" Margin="14,120,0,0" VerticalAlignment="Top" FontWeight="Bold" FontFamily="Segoe UI Semibold" Background="White" Foreground="White"/>
        <CheckBox x:Name="Desktop_CheckBox" Grid.ColumnSpan="4" Content="Add Public Desktop shortcut?" HorizontalAlignment="Left" Margin="12,175,0,0" VerticalAlignment="Top" Foreground="White" FontWeight="Bold" FontFamily="Segoe UI Semibold"/>
        <CheckBox x:Name="Copy_CheckBox" Grid.ColumnSpan="4" Content="Copy a Folder/File?" HorizontalAlignment="Left" Margin="12,230,0,0" VerticalAlignment="Top" Foreground="White" FontWeight="Bold" FontFamily="Segoe UI Semibold"/>
        <CheckBox x:Name="ExtraCode_CheckBox" Grid.ColumnSpan="4" Content="Add extra code?" HorizontalAlignment="Left" Margin="10,309,0,0" VerticalAlignment="Top" Foreground="White" FontWeight="Bold" FontFamily="Segoe UI Semibold"/>

        <Label x:Name="StartMenu_Label" Grid.ColumnSpan="4" Content="Shortcut Filename (including extension):" HorizontalAlignment="Left" Margin="32,140,0,0" VerticalAlignment="Top" Width="253" Foreground="White" FontSize="14" Visibility="Hidden"/>
        <Label x:Name="Desktop_Label" Grid.ColumnSpan="4" Content="Shortcut Filename (including extension):" HorizontalAlignment="Left" Margin="32,195,0,0" VerticalAlignment="Top" Width="253" Foreground="White" FontSize="14" Visibility="Hidden"/>
        <Label x:Name="Copy_Label" Grid.ColumnSpan="4" Content="File/Folder Path:" HorizontalAlignment="Left" Margin="32,250,0,0" VerticalAlignment="Top" Width="253" Foreground="White" FontSize="14" Visibility="Hidden"/>
        <Label x:Name="CopyDestination_Label" Grid.ColumnSpan="4" Content="Destination Path:" HorizontalAlignment="Left" Margin="32,280,0,0" VerticalAlignment="Top" Width="253" Foreground="White" FontSize="14" Visibility="Hidden"/>
        <Label x:Name="ExtraCode_Label" Grid.ColumnSpan="4" Content="Insert other code (use Enter to disguish different lines):" HorizontalAlignment="Left" Margin="120,329,0,0" VerticalAlignment="Top" Width="329" Foreground="White" FontSize="14" Visibility="Hidden"/>

        <TextBox x:Name="StartMenuShortcut_TextBox" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="27" Margin="285,143,0,0" TextWrapping="Wrap" Text="" VerticalAlignment="Top" Width="250" Background="#FFE5E5E5" Visibility="Hidden"/>
        <TextBox x:Name="DesktopShortcut_TextBox" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="27" Margin="285,198,0,0" TextWrapping="Wrap" Text="" VerticalAlignment="Top" Width="250" Background="#FFE5E5E5" Visibility="Hidden"/>
        <TextBox x:Name="Copy_TextBox" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="27" Margin="185,250,0,0" TextWrapping="Wrap" Text="" VerticalAlignment="Top" Width="250" Background="#FFE5E5E5" Visibility="Hidden"/>
        <TextBox x:Name="CopyDestination_TextBox" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="27" Margin="185,280,0,0" TextWrapping="Wrap" Text="" VerticalAlignment="Top" Width="250" Background="#FFE5E5E5" Visibility="Hidden"/>
        <TextBox x:Name="ExtraCode_TextBox" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="118" Margin="32,358,0,0" TextWrapping="Wrap" Text="" VerticalAlignment="Top" Width="503" Background="#FFE5E5E5" AcceptsReturn="True" Visibility="Hidden" Grid.RowSpan="2"/>
        <Label x:Name="Author" Grid.ColumnSpan="4" Content="Austin Webber" HorizontalAlignment="Left" Margin="451,9,0,0" VerticalAlignment="Top" Width="125" Foreground="White" FontSize="12"/>
        <Button x:Name="Generate_Button" Content="Generate" Grid.Column="3" HorizontalAlignment="Left" Margin="1,109.009,0,0" VerticalAlignment="Top" Width="568" Height="38.5" Background="White" FontWeight="Bold" IsEnabled="False" Grid.Row="1"/>
        <Border BorderBrush="Black" BorderThickness="3" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="115" VerticalAlignment="Top" Width="568" Margin="1,0,0,0"/>
        <Border BorderBrush="Black" BorderThickness="3,0,3,3" Grid.ColumnSpan="4" HorizontalAlignment="Left" Height="387" VerticalAlignment="Top" Width="568" Margin="1,110,0,0" Grid.RowSpan="2"/>
        <Button x:Name="Browse_Button" Content="Browse" Grid.ColumnSpan="4" HorizontalAlignment="Left" VerticalAlignment="Top" Height="27" Width="85" Margin="450,45,0,0" BorderBrush="#FFABADB3" Background="#FFE5E5E5"/>
    </Grid>
</Window>

"@

$inputXML = $inputXML -replace 'mc:Ignorable="d"', '' -replace "x:N", 'N' -replace '^<Win.*', '<Window'
[void][System.Reflection.Assembly]::LoadWithPartialName('presentationframework')
[xml]$XAML = $inputXML
#Read XAML

$reader = (New-Object System.Xml.XmlNodeReader $xaml)
try
{
	$Form = [Windows.Markup.XamlReader]::Load($reader)
}
catch
{
	Write-Warning "Unable to parse XML, with error: $($Error[0])`n Ensure that there are NO SelectionChanged or TextChanged properties in your textboxes (PowerShell cannot process them)"
	throw
}


# Load XAML Objects In PowerShell 
$xaml.SelectNodes("//*[@Name]") | %{
	"trying item $($_.Name)";
	try { Set-Variable -Name "WPF$($_.Name)" -Value $Form.FindName($_.Name) -ErrorAction Stop }
	catch { throw }
} | Out-Null
```

$inputXAML contains your actual XAML extracted from Visual Studio, and the remaining code inputs the code in a PowerShell script as variables.

Next, how do you handle actions from each of the buttons/textboxes? Well, most of the commands are fairly straightforward and easy to manipulate. One of the first things I did when setting up a GUI was to allow a user to click "Browse" and select a specific file from any directory. Here's a simple snippet of code that allows it to work:

```
$WPFBrowse_Button.Add_Click({ 
$WPFInstallationFile_TextBox.Text = Get-FileName -initialDirectory $env:USERNAME\Desktop
$WPFGenerate_Button.IsEnabled = $true

})
```

When the browse button is clicked, it calls a simple function I created called Get-FileName, which takes an initial directory as an input. From this file selection, it then sets the text of the installation file textbox to the file selected. At this point, I've also enabled the generate button to allow the user to generate anytime after this selection. If you notice, all of these actions take place within a single script block denoted with braces {}.

![After File Selection](https://i.imgur.com/HQJeaBh.png)

Here's the function I used to allow a file selection of a particular type:

```
Function Get-FileName($initialDirectory)
{
	[System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms") |
	Out-Null
	
	$OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog
	$OpenFileDialog.initialDirectory = $initialDirectory
	$OpenFileDialog.filter = "Executable Files (*.exe, *.msi)| *.exe;*.msi"
	$OpenFileDialog.ShowDialog() | Out-Null
	$OpenFileDialog.filename
}
```

This allows me to gather data from the file selected, and input it into my textbox for use later. I sourced this function [here](https://devblogs.microsoft.com/scripting/hey-scripting-guy-can-i-open-a-file-dialog-box-with-windows-powershell/). 

Now that I learned how to manipulate buttons, I begin to add basic functionality to my checkboxes. The textboxes and labels are hidden by default for each checkbox. I simply added a Add_Checked and Add_UnChecked action for each checkbox:


```
$WPFStartMenu_CheckBox.Add_Checked({ if ($WPFStartMenu_Label.Visibility -ne 'Visible') { $WPFStartMenu_Label.Visibility, $WPFStartMenuShortcut_TextBox.Visibility = 'Visible', 'Visible' } })
$WPFStartMenu_CheckBox.Add_UnChecked({ if ($WPFStartMenu_Label.Visibility -eq 'Visible') { $WPFStartMenu_Label.Visibility, $WPFStartMenuShortcut_TextBox.Visibility = 'Hidden', 'Hidden' } })
```

All this code is doing is flipping the visibility based on current status. If it's visible, hide it. If it's hidden, make it visible. There's probably an easier way to do this, but I used this as my starting point. I added this for each text box and label that is default hidden related to each checkbox respectively.

![Checkbox Display](https://i.imgur.com/iSmCbMa.png)

Next, we need to gather all user input from the GUI. When a user is done inputting information into each field, they need to click the generate button at the bottom. We'll need to gather the following input: installation file name, installation switches, and input from all other textboxes:

```
$WPFGenerate_Button.Add_Click({
		if ($WPFStartMenu_CheckBox.IsChecked -eq $false) { $WPFStartMenuShortcut_TextBox.Text = "UserInputNull" }
		if ($WPFDesktop_CheckBox.IsChecked -eq $false) { $WPFDesktopShortcut_TextBox.Text = "UserInputNull" }
		if ($WPFCopy_CheckBox.IsChecked -eq $false)
		{
			$WPFCopyDestination_TextBox.Text = "UserInputNull"
			$WPFCopy_TextBox.Text = "UserInputNull"
		}
		if ($WPFExtraCode_CheckBox.IsChecked -eq $false) { $WPFExtraCode_TextBox.Text = "UserInputNull" }
		
		#Gather MSI productCode
		if ($WPFInstallationFile_TextBox.Text -match '.msi') {$productCode = getMSIData -Path $WPFInstallationFile_TextBox.Text -Property ProductCode}
		
		#Generate Script
		generateScript -Switches $WPFSwitches.Text -installationFileLocation $WPFInstallationFile_TextBox.Text -productCode $productCode -desktopIconName $WPFDesktopShortcut_TextBox.Text -startMenuShortcut $WPFStartMenuShortcut_TextBox.Text -copyOverFile $WPFCopy_TextBox.Text -copyIntoDirectory $WPFCopyDestination_TextBox.Text -extraCode $WPFExtraCode_TextBox.Text
		
		#Close form
		$Form.Close()
	})
```

The first part of this code will set the input from the textboxes to null if a checkbox isn't checked. I just implemented this in case a checkbox was checked, but then unchecked before generation.

The second part of this code simply checks if their installation file is an .MSI. If it is, it will gather the product code using the function I created in the previous blog. Again, this is only needed to provide an easy way to generate uninstallation scripts

We'll send all the gathered information to a function I created called "generateScript", which just assigns all the data to temporary variables. I then close the form by calling $Form.Close().

At this point, I'm reusing the code from the previous blog and sending the input into that infrastructure to be parsed. Here's the parameters of the function:

```
function generateScript
{
	param (
		[Parameter(Mandatory = $true)]
		$switches,
		[Parameter(Mandatory = $true)]
		$installationFileLocation,
		[Parameter(Mandatory = $false)]
		$productCode,
		[Parameter(Mandatory = $false)]
		$desktopIconName,
		[Parameter(Mandatory = $false)]
		$startMenuShortcut,
		[Parameter(Mandatory = $false)]
		$copyOverFile,
		[Parameter(Mandatory = $false)]
		$copyIntoDirectory,
		[Parameter(Mandatory = $false)]
		$extraCode
	)
```

Finally, for the most important part. Displaying the form:

```
$Form.ShowDialog() | Out-Null
```

Piping the display into Out-Null is optional, but I used this to hide input from the console.

Done. That's how I setup my first GUI in PowerShell! 

In my next blog, I'll discuss some more changes and additions to my program. I also ended up changing my entire GUI from WPF to WinForms.

Thank you for reading.