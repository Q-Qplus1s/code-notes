# PhP Cheat Sheet
I don't have a better repo to store this in, so it will live here

* [SWHLabPHP / abf.php](https://github.com/swharden/SWHLabPHP/blob/master/recode/src/functions/abf.php) has lots of copy/pastable functions

## Get Querystring Variables

#### See if a variable is defined, regardless of value
```php
isset($_REQUEST[$key]
```

#### If it's defined use it, if not use a default
```php
$param = (isset($_REQUEST[$key]) ? $_REQUEST[$key] : $default);
```

## PhP HTML If/Else Blocks
```html
<? if ($condition): ?>
  <p>Content</p>
<? elseif ($other_condition): ?>
  <p>Other Content</p>
<? else: ?>
  <p>Default Content</p>
<? endif; ?>
```

## Read a CSV into a 2d array
```php
$origin_project_file="origin-projects.txt";
$f = fopen($origin_project_file, "r");
$raw=fread($f,filesize($origin_project_file));
fclose($f);
foreach (explode("\n",$raw) as $line){
    $line=trim($line);
    if (strlen($line)<3) continue; // ignore short lines
    if ($line[0]=='#') continue; // ignore comments
    $parts = explode(",",$line,2);
}
```

## Barebones HTML file
```html
<html>
<head>
<title>Awesome Webpage</title>
<link rel="stylesheet" href="styles.css">
<style>
body {font-family: sans-serif;}
a {color: blue; text-decoration: none;}
a:hover {color: blue;text-decoration: underline;}
</style>
</head>
<body>
Awesome Content
</body>
</html>
```

## Underline links only on hover
```html
a {
    color: blue;
    text-decoration: none;
}
a:hover {
    color: blue;
    text-decoration: underline;
}
```

## Page Render Server and Timestamp
```php
$date_stamp = (new DateTime('', new DateTimeZone('US/Eastern')))->format('Y-m-d H:i:s'); 
$server_name = $_SERVER['SERVER_ADDR'];
echo "Page generated by $server_name on $date_stamp";
```

## Search an indexed filesystem

This is really just a fancy SQL query call to the system's server.
Example query (`%` is a wildcard):
```sql
SELECT System.ItemPathDisplay FROM SYSTEMINDEX WHERE System.FileName LIKE '10909%.abf' 
```

Sample code:
```php

// Use windows indexing to rapidly search for a file on the hard drive
// you can use * or % as a wildcard (i.e., "17n%.abf")
// BE CAREFUL! this function  may be sensitive to SQL injection.
function com($path, $string, $maximum_records=500){
    $string = str_replace("*","%",$string);
    $time_start = microtime(true);
    
    $sql="SELECT System.ItemPathDisplay FROM SYSTEMINDEX WHERE System.FileName LIKE '$string'";
    echo "<!-- \n\n\n SQL QUERY: $sql \n\n\n -->";

    // requires COM extension to be installed (edit php.ini)
    $files=[];
    $conn = new COM("ADODB.Connection") or die("Cannot start ADO");
    $recordset = new COM("ADODB.Recordset");
    $recordset -> MaxRecords = $maximum_records;
    $conn -> Open("Provider=Search.CollatorDSO;Extended Properties='Application=Windows';");
    $recordset -> Open($sql, $conn);
    $recordset -> MoveFirst();   
    while (!$recordset -> EOF) {
        $found_file_path=$recordset -> Fields -> Item("System.ItemPathDisplay") -> Value;
        //echo "<li>$found_file_path";
        $files[]=$found_file_path;
        $recordset -> MoveNext();
    }

    // display information about how long it took
    $time_elapsed = round((microtime(true) - $time_start)*1000,2);
    $count = count($files);
    $action = "found";
    if ($count==$maximum_records) $action = "maxed-out at";
    echo "<div><i>search for \"$string\" in \"$path\" $action $count results in $time_elapsed</i> ms</div>";
    
    return $files;
}

com('D:/X_Drive/','10909004.abf'); // search for "10909004.abf" in "D:/X_Drive/" found 1 results in 13.04 ms
com('D:/X_Drive/','*.opj'); // search for "%.opj" in "D:/X_Drive/" maxed-out at 500 results in 142.9 ms
com('D:/X_Drive/','FAKE*.jpg'); // search for "FAKE%.jpg" in "D:/X_Drive/" found 2 results in 1.15 ms

```


## Render markdown as HTML
Download [parsedown.php](https://github.com/erusev/parsedown)
```php
include_once('Parsedown.php');
$Parsedown = new Parsedown();
$f = fopen($markdown_filename, "r");
$raw = fread($f,filesize($markdown_filename));
fclose($f);
echo $Parsedown->text($raw);
```

## List Files and Folders in a Directory
```php
$rootFolder='X:\Lab Documents';
$folderNames=[]; // will get filled
$fileNames=[]; // will get filled
foreach (scandir($rootFolder) as $name){
    if ($name=='.' || $name=='..') continue;
    if (is_dir($rootFolder."/".$name)) $folderNames[]=$name;
    else $fileNames[]=$name;
}
sort($folderNames);
sort($fileNames);
echo "<br><b>FOLDERS</b><br>";
foreach ($folderNames as $name) echo "$name<br>";
echo "<br><b>FILES</b><br>";
foreach ($fileNames as $name) echo "$name<br>";
```

## Send a partially-rendered HTML file
Running a slow script that you want to start printing-out text as it runs? Use this to force sending text:
```php
flush();ob_flush(); // update the browser
```

## Get file extension
```php
$extension=pathinfo($fname, PATHINFO_EXTENSION);
```

## Shadows on images
```css
.micrograph{
    margin: 5px;
    border: 1px solid black;
    background-color: black;
    box-shadow: 2px 2px 7px  rgba(0, 0, 0, 0.5); 
}
```

```html
<img style="margin: 5px; border: 1px solid black; background-color: black;
            box-shadow: 2px 2px 7px  rgba(0, 0, 0, 0.5); " src="test.jpg" />
```

## Display File Modified Time
```php
function file_age_string($fname){
    
    // determine file age
    $ageSec=time()-filemtime($fname);
    $ageMin=$ageSec/60;
    $ageHr=$ageMin/60;
    $ageDy=$ageHr/24;
    $ageYr=$ageDy/365.25;
    $ageString=date("F d Y H:i:s.", filemtime($fname));
    
    // determine string formatting
    if ($ageHr<1) $ageString=sprintf("%.02f min", $ageMin);
    else if ($ageDy<1) $ageString=sprintf("%.02f hr", $ageHr); 
    else if ($ageDy<60) $ageString=sprintf("%.02f d", $ageDy);
    else $ageString=sprintf("%.02f yr", $ageYr);

    return $ageString;
}
```

## String Contains Substring
You there isn't a `contains()` that I know of, so strpos works... but be careful because if the string starts with the substring then strpos will be zero and false will be returned. Notice the difference between `!=` and `!==`
```php
if (strpos($fname,$matching)!==FALSE){
  echo $fname."<br>";
}
```

## HTML Redirect
Put this in the head
```HTML
<meta http-equiv="refresh" content="0; url=http://example.com/" />
```

## Convert TIF Images to PNG or JPG
```php

function tiff_convert_folder($folder, $putInSubFolder="/swhlab/"){
    // given a folder with a bunch of TIF files, use python to make them JPGs.

    if (!file_exists($folder)) return;

    $folder_output=$folder.$putInSubFolder;
    if (!file_exists($folder_output)) mkdir($folder_output);

    $files = scandir($folder);
    $files2 = scandir($folder_output);
    $tifs_to_convert=[];
    foreach ($files as $fname){
        $extension=strtolower(pathinfo($fname, PATHINFO_EXTENSION));
        if ($extension == "tif" || $extension == "tiff") {
            if (!in_array($fname.".jpg",$files2)){
                $tifs_to_convert[]=$fname;
            }
        }
        
    }

    foreach ($tifs_to_convert as $tifFile){
        $fileIn=$folder."/".$tifFile;
        $fileOut=$folder_output.$tifFile.".png";
        if (file_exists($fileOut)) continue;
        $cmd="convert \"$fileIn\" -contrast-stretch 0.15x0.05% \"$fileOut\"";
        echo "<div style='font-family: monospace;'>$cmd</div>";
        exec($cmd);       
        flush();ob_flush();
    }
    
}
```
