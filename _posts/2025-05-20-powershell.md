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


