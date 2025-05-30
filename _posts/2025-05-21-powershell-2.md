To reload the Microsoft.PowerShell_profile.ps1 file where you can store funtions. The use of dot gives it the name "dot sourcing"

```. $PROFILE```

You can create new .ps1 files and load them (again dot-sourcing):

```. "$HOME\Documents\PowerShell\MyFunctions.ps1"```

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

Iterating:

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

Collection types (there are more, these seem useful for a start):

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

^(?!.*[.;]).+?(\w+)\(    // deze regex extraheert method names
