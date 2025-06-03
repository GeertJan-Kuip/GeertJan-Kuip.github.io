## PowerShell 2

### Creating scripts, modules, and loading them

#### Microsoft.PowerShell_profile.ps1

To make PowerShell more potent you can start writing scripts. The script file that is automatically loaded is this one: 

```
C:\Users\User\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

To open it you can type ```notepad $profile```. The file will be opened in Notepad, $profile is the variable that represents this file.

#### Creating modules

In the same folder you can create extra ps1 files or even modules. To create a module, make a new folder and put a file with a .psm1 extension in it. In this psm1 .file you can name .ps1 files that will then become part of the module. A .psm1 file might look like this:

```
. "$PSScriptRoot\PSShowMenu.ps1"
. "$PSScriptRoot\PSFormatOutput.ps1"
```

The dot at the start is a way of saying that this file needs to be loaded, it is called 'dot sourcing'. This dot sourcing is always used when you want to force the (re)load of a script file.

When you have a module, it will only be loaded automatically if it is imported by the $profile (Microsoft.PowerShell_profile.ps1). To achieve this, use import statements at the start of $profile like this:

```
Import-Module "$HOME\Documents\WindowsPowerShell\JavaTools"
Import-Module "$HOME\Documents\WindowsPowerShell\Utilities"
```

You point to the directory in which the .psm1 file resides. I'm not sure if the .psm1 file needs to have the same name as the directory. 

#### Forced reload

When you alter scripts you want them to reload. If they're not reloaded the program won't use the new script. The difficult way is to restart close and restart PowerShell, the easier way is to dot source $profile:

```. $PROFILE```

You can create and/or adjust other .ps1 files and load them as well (again dot-sourcing):

```. "$HOME\Documents\PowerShell\MyFunctions.ps1"```

What I noticed was that when you make changes in module files you need to do dot sourcing as well, and simply doing . $profile won't get you the module reload you want. I circumvented this problem by adding the following lines in $profile, just beneath the module declarations:

```
. "$HOME\Documents\WindowsPowerShell\JavaTools\PSJavaTools.ps1"
. "$HOME\Documents\WindowsPowerShell\Utilities\PSFormatOutput.ps1"
```

Now if you type ```. $profile```, the individual .ps1 file from the module folders get a forced reload. It is not pretty but assuming that the scripts are to be used instead of to be altered most of the time it is okay. 

### Some specifics about PowerShell script

#### Object collections

Get-ChildItem returns a collection, but specifically, it returns a collection of .NET objects depending on what it finds — usually System.IO.FileInfo and System.IO.DirectoryInfo objects.

If you want to know the types in collections, objects:

```
Get-ChildItem | Get-Member

(Get-ChildItem)[0].GetType().FullName   -- type of the first element in collection

$items = Get-ChildItem

foreach ($item in $items) {
    $item.GetType().FullName
}
```

#### How to iterate

There are several ways to do loops:

```
# Method 1: foreach loop
foreach ($item in $items) {
    Write-Output "Fruit: $item"
}

# Method 2: ForEach-Object (pipeline-friendly)
$items | ForEach-Object {
    Write-Output "Fruit: $_"
}

# Loop through key-value pairs
foreach ($key in $person.Keys) {
    $value = $person[$key]
    Write-Output "$key: $value"
}

# OR: using GetEnumerator()
foreach ($entry in $person.GetEnumerator()) {
    Write-Output "$($entry.Key): $($entry.Value)"
}

# Using index
$colors = @("red", "green", "blue")

for ($i = 0; $i -lt $colors.Count; $i++) {
    Write-Output "Color #$i: $($colors[$i])"
}

```

#### Collection types

Collection types (there are more, these seem useful for a start). The simple way of declaring/initializing is by either @() for an array or @{} for a hashtable. There are other ways to declare collections as well. List might sometimes be preferable above array because it is not fixed size. Btw comments are preceeded by #. 

```
-- Array (fixed size, mixed types)
$array = @(1, 2, 3)
$array[0]  # => 1
$array.Count  # => 3

--HashTable
$map = @{
    "name" = "Alice"
    "age" = 30
}
$map["name"]  # => Alice


--List (size adjustable)
--Strongly typed variants available (e.g., List[string])
$list = New-Object System.Collections.Generic.List[object]
$list.Add("apple")
$list.Add("banana")
$list[1]  # => banana

--Set
$set = [System.Collections.Generic.HashSet[string]]::new()
$set.Add("a")
$set.Add("b")
$set.Add("a")  # duplicates not allowed

```


