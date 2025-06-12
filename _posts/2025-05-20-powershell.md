# PowerShell

Learning Spring is quite hard, there is a lot in it. To keep myself motivated I'm gonna learn the things I hate to be bad at, one of them having control over my pc via PowerShell. In this blog post I will make a list of commands and techniques that help me to do less with a mouse and get closer to the OS. In the near future I will make similar posts for curl, jq and Maven. Yes, I want to do all Maven things via the command line.

## Navigating Files and folders

#### Change directory

I was wondering whether it is possible to go up and down the directory in one command. It is possible, a bit similar to Java .relativize() method from Path:

```
>cd ../../otherFolder
```

You can use Tab to scroll through the folder list after you typed ```cd ../../```, PowerShell will know what you try to do and it works.

#### Directory listing, file listing

You can get a list of files and directories using ```Get-ChildItem```. Using ```Get-ChildItem -Recurse``` gives a list in which all nested files and directories, no matter how deep, are included as well. ```Get-ChildItem -File``` gives only files, ```Get-ChildItem -Directory``` gives only folders. To get certain type of files, do ```Get-ChildItem *.java``` or ```Get-ChildItem -Recurse -Filter *.java``` if you want all nested files of that type as well.

You can give the output a table format the following way:

```
Get-ChildItem -File -Recurse | Format-Table name, length
```

The 'name' and 'length' are the properties that will be in the table. They are case-insensitive, 'LEngth' works as well. You can add 2 more properties, namely LastWriteTime and CreationTime. You can also sort the table, then the sorting must be done before formatting:

```
Get-ChildItem -File -Recurse | Sort-Object Length -Descending | Format-Table namE, LeNgth
```

In this example -Descending is used to sort in inverse order. There is no -Ascending counterpart (term is not recognized) but ascending is the default. You can add ```-AutoSize``` to the 'Format-Table' part, it will do adjustments of column width and results in a neater look.

#### Shorthands

There are shorthands for many commands. Get-ChildItem has gci, Sort-Object has sort and Format-Table has ft.

#### Using Tab

If you have a directory listing and want to navigate to one of those directories, you do not have to type the full name. After you have typed the first letter, you can use Tab and you get autocomplete. If multiple folders fit, use Tab repeatedly until you have the right one.

## Creating files and folders

#### Create a file with Notepad (fast method)

ChatGPT provided this method to quickly create a file and open it in Notepad. Notepad will open and complain that it cannot find the file and will ask if it has to create it. Confirm and you will have the file created and open for editing in Notepad. It works for .txt but also for other extensions.

```
notepad "example.txt"  // creates in current dir

notepad "C:/example.properties"  // creates it somewhere else, with different extension
```

#### Using Out-File to create

This is relatively easy too:

```
"" | Out-File "example.txt"
```

Instead of an empty string you can use the contents of a variable to put in the Out-File. If you have stored output in a variable, this allows you to write that output to a text file:

```
$res | Out-File "http-headers.txt"
```

#### Using Set-Content and Add-Content

The Set-Content will overwrite a file if it already exists. The Add-Content method is useful if you want to add to a textfile in a loop or so. You can use other file extensions, the file will be treated as a textfile anyhow.

```
"This is some text" | Set-Content "example.txt"  // create file with content

"Another line" | Add-Content "example.txt"   //  adds a new line
```

If you add content to a textfile and want to have line breaks, use \`n. 

#### Using New-Item for file or directory

The more verbose way, whereby any filetype can be created, is this:

```
New-Item -Path "example.txt" -ItemType File

or, with shorthands

ni -p "example.txt" -ItemType File
```

It will give a warning if the file already exists. You can also create a directory this way by replacing File with Directory:

```
ni -p "MyDir" -ItemType Directory

ni MyDir -ItemType Directory  // even shorter, works as well

```

#### Using classic mkdir

This is a sort of universal classic notation and it works in PowerShell. ```mkdir MyDirectory```.


## Deleting files and folders

#### Deleting files

Removing can be done with _Remove-Item_, which can be shortened to _rm_, or with _del_. The three methods below all do the same:

```
Remove-Item "index.html"

rm "index.html"

del "index.html"
```

#### Deleting folders

With folders you need *-Recurse* to indicate that a folder with content can be deleted, including the content. If you do not add -Recurse, PowerShell will ask you to confirm that you want to delete the folder with all content.

```
Remove-Item "MyFolder"  // will ask for confirmation

Remove-Item "MyFolder" -Recurse  // won't ask
```

There is a -Force argument as well to suppress warnings but I don't think I'll need it.

## File properties and preview

#### File properties

There is the simple properties you can get via Get-Item or Get-ChildItem. For this Get-ChildItem is faster, and you can abbreviate it to gci:

```
Get-Item myFile.yml

Get-ChildItem   // or just gci
```

If you want one specific property, either a hidden or a non-hidden one:

```
(Get-Item "myFile.yml").Length

(Get-Item "myFile.yml").Exists   // Exists is a hidden property
```

For a more extensive overview of all properties, including the hidden ones:

```
Get-Item "myFile.yml" | Format-List *
or 
Get-Item "myFile.yml" | fl *
```

Some properties are not retrieved by Get-Item but by Get-Acl. Acl stands for Access Control List.

```
(Get-Acl "myFile.yml").Owner  // gives specific Acl property

Get-Acl "myFile.yml" | fl *
```

The Format-List * is important, otherwise it is unreadable. It can be abbreviated to fl.

By the way everything you can do with files can be done with folders as well. They have properties, retrievable via Get-Item and Get-Acl.

#### File preview

The straight way to preview the first n lines of a file containing text:

```
Get-Content "example.txt" -TotalCount 10
or 
gc example.txt -TotalCount 10
```

You can also be a more sophisticated with *Select-Object*. This allows you to do the last n lines if you want:

```
gc example.txt | Select-Object -First 10

gc example.txt | Select-Object -Last 10
```

## Filtering and searching file content

#### Select-String

An easy way to get only the lines that contain some specific word is this. You start with the string you search for and you end with -Path + file:

```
Select-String "error" -Path "log.txt"
```

#### Where-Object

A more sophisticated method is this, _using Where-Object_. You can add an extra element to limit the output:

```
Get-Content "example.txt" | Where-Object { $_ -like "*error*" }

Get-Content "example.txt" | Where-Object { $_ -like "*error*" } | Select-Object -First 10
```

Here some superpower of PowerShell appears. The Get-Content creates a sort of iteration over all lines in the file, and on each line the second statement ```Where-Object { $_ -like "*error*" }``` is applied. $_ represents an element in the iteration (a line). ```-like "*error*"``` means that you search for the word error. If it would be ```*error``` it would mean 'ends with error'. There are other operators than like, such as 'match', 'contains' and 'lt' (less than). The {} thing is called a script block, it is in here where you can set the conditions.

## Writing custom functions

This is a great topic. There is a file ```Microsoft.PowerShell_profile.ps1``` in ```C:\Users\User\Documents\WindowsPowerShell``` in which you can write your own custom functions. This reminds me of my days with 3D Studio Max and maxscript. It is rather simple but effective if using the scripts is part of your daily routine. To get the ps1 file, type ```notepad $PROFILE```. It opens the specific file for you.

Within the file you can create functions> Naming convention is that it must start with a letter or underscore but numbers are allowed.

Important: by default PowerShell didn't allow me to run the scripts. Upon restarting PowerShell (as administrator) it gave a warning that I had no permission to use scripts. This policy could be changed by typing: 

```
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned  // you get asked to confirm, type Y
```

After this, to see what you have changed, type ```Get-ExecutionPolicy -List```. 

#### My first function

I want to be able to get to the right directory faster. Having a method setting the Java projects folder as cuurent seems handy:

```
function Kuip2 {
    cd "C:/Kuips files/Java projects"
}
```

Very straightforward indeed. You can make a parameterized function as well:

```
function Kuip1 {
    param (
        [string]$d
    )

    if ($d -eq "java") {
        cd "C:/Kuips files/Java projects"
    } elseif ($d -eq "kuip") {
        cd "C:/Kuips files"
    } else {}
}
```

Microsoft has documented it well and ChatGPT helps as well. I'm gonna think what sort of functions I need.

## Environment and system variables

#### Environment variables

PowerShell has ```$env```, which according to ChatGPT: _$env in PowerShell is not a reserved variable, but rather a special drive provider—specifically, it's a shortcut to the Environment Provider, which exposes environment variables as if they were part of a file system._

If you want to have a list of all environment variables, simply do:

```
gci env:
or
Get-ChildItem env:
```

If you want a specific one, or use a wildcard, do:

```
gci env:OS // returns the OS environment variable

gci env:*java*  // returns all environment variables containig java (case insensitive)
```

#### Path variables

Here you need ```-split``` because the path variables are some sort of string.

```
$env:path -split ";"  // all path variables

$env:path -split ";" | where-object {$_ -like "*Users*"} // only those containing "users" (case-insensitive)
```

The above samples indicate that you sometimes need to use ```env:``` and sometimes ```$env:```. This has to do with the fact that certain methods ('cmdlets') understand that when you write ```env:```, you mean ```$env:```. The ```$``` indicates it is a variable, which it is, but somehow you can sometimes refer to it as ```env:```. 

It is important not to assign a new value to ```$env``` as you will loose access to this specific use. If you have overwritten it anyway, you can revert it by:

```
Remove-Variable env
```

#### Setting environment variables

When setting environment variables via PowerShell you need to be aware that by default they will only last as long as the PowerShell session. This is the basic command:

```
$env:BLA_BLA = "Hello guys"

gci env:BLA_BLA  // returns "Hello guys"
```

To make it persistent, for all users with administration rights:

```
[System.Environment]::SetEnvironmentVariable("MY_VARIABLE", "MyValue", "Machine")
```

The "Machine" parameter indicates that this will be the same for every user. If you replace "Machine" with "User", the variable will only be part of the profile of "User".

Note: changes in persistent variables won't be reflected in the current session, you need to restart PowerShell.

To persistently change variables:

```
# For the current user
[Environment]::SetEnvironmentVariable("MY_VARIABLE", "NewValue", "User")

# For all users (requires admin)
[Environment]::SetEnvironmentVariable("MY_VARIABLE", "NewValue", "Machine")
```

And to remove a variable (again, restarting PS is needed to see the changes):

```
# Remove from current user scope
[Environment]::SetEnvironmentVariable("MY_VARIABLE", $null, "User")

# Remove from system scope (requires admin)
[Environment]::SetEnvironmentVariable("MY_VARIABLE", $null, "Machine")
```

#### Setting path variables

ChatGPT warns that this is a risky operation and advises to make a backup of path variables before attempting to add or remove paths. The advice:

- Always backup the current PATH before making changes.
- Use absolute paths.
- Avoid adding duplicate entries.
- You may need to restart your system or log out/in for system-wide changes to take effect.

The code to save a backup:

```
# Save User PATH
[Environment]::GetEnvironmentVariable("Path", "User") | Out-File -FilePath "$HOME\user_path_backup.txt"

# Save System PATH (requires elevated permissions to view fully)
[Environment]::GetEnvironmentVariable("Path", "Machine") | Out-File -FilePath "$HOME\system_path_backup.txt"
```

To restore the backup:

```
# Restore User PATH from backup
$originalPath = Get-Content "$HOME\user_path_backup.txt" -Raw
[Environment]::SetEnvironmentVariable("Path", $originalPath, "User")
```

The code to add a Path:

```
-- add to user path (no admin required)
$oldPath = [Environment]::GetEnvironmentVariable("Path", "User")
$newPath = "$oldPath;C:\My\New\Folder"
[Environment]::SetEnvironmentVariable("Path", $newPath, "User")

-- add to system path (requires admin)
$oldPath = [Environment]::GetEnvironmentVariable("Path", "Machine")
$newPath = "$oldPath;C:\My\New\Folder"
[Environment]::SetEnvironmentVariable("Path", $newPath, "Machine")
```

Removing goes like this (change "User" to "Machine" to do it for System paths):

```
$path = [Environment]::GetEnvironmentVariable("Path", "User") -split ';'
$newPath = ($path | Where-Object { $_ -ne "C:\Old\Path" }) -join ';'
[Environment]::SetEnvironmentVariable("Path", $newPath, "User")
```

To check the result (_Always check the result_):

```
[Environment]::GetEnvironmentVariable("Path", "User") -split ';'

[Environment]::GetEnvironmentVariable("Path", "Machine") -split ';'
```

## Inquiring the hardware

Here some handy commands:

```
-- returns a large object with many specs
Get-ComputerInfo

-- legacy command
systeminfo

-- OS details
Get-CimInstance Win32_OperatingSystem

-- processor
Get-CimInstance Win32_Processor
Get-CimInstance Win32_Processor | fl *  -- looks better, more info

-- BIOS
Get-CimInstance Win32_BIOS

-- RAM
Get-CimInstance Win32_PhysicalMemory

-- Disk info
Get-PSDrive -PSProvider FileSystem

-- GPU info
Get-CimInstance Win32_VideoController | Select-Object Name, DriverVersion

-- Network info
Get-NetIPAddress

```


## Not the end

This is not the end of my PowerShell research, I actually like the whole idea of doing things from a terminal. I'm almost ready for touch typing as well, as I practice it on [keybr](https://www.keybr.com/) and only have to learn the Q, X and Y plus the non-letters.

## Additions

```
-- Show directory tree starting in current folder
tree

-- Same, but with files as well
tree /f
```




