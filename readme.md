# BARTENDER
![logo](./icon.png)

> A Framework for making PowerShell Modules

[releasebadge]: https://img.shields.io/static/v1.svg?label=version&message=6.1.22&color=blue
[datebadge]: https://img.shields.io/static/v1.svg?label=Date&message=2019-03-14&color=yellow
[psbadge]: https://img.shields.io/static/v1.svg?label=PowerShell&message=5.0.0&color=5391FE&logo=powershell
[btbadge]: https://img.shields.io/static/v1.svg?label=bartender&message=6.1.21&color=0B2047


| Language | Release Version | Release Date | Bartender Version |
|:-------------------:|:-------------------:|:-------------------:|:-------------------:|
|![psbadge]|![releasebadge]|![datebadge]|![btbadge]|


Authors: Adrian.Andersson

Company: Domain Group

Latest Release Notes: [here](./documentation/6.1.22/release.md)

***

<!--Bartender Dynamic Header -- Code Below Here -->

## What is this

A module framework for PowerShell


## Why did I make this

1) To make my PowerShell modules follow the Continuous Integration -> Continuous Delivery lifecycle

2) To make collaborating on PowerShell projects as easy as possible

3) To try and encourage good practices around Pester Testing and Documentation


## Quick Start Guide

### Install the module

```powershell
install-module -name bartender
```


### Setup your bartender publish repository


```powershell
save-btRepository -repository myRepo -token myNugetToken -credential $(get-credential)
```
> assumes a repository is already in place

> Check [here](https://powershellexplained.com/2018-03-03-Powershell-Using-a-NuGet-server-for-a-PSRepository/?utm_source=blog&utm_medium=blog&utm_content=tags) for an execelent blog post from Kevin Marquette for more details

> Token is used for publishing to nuget repositories and is not needed for fileshare repositories

> Credentials are used for checking module dependancies and are only required if the repository is private

### Setup your bartender defaults


```powershell
$myDefaults = @{
   author = 'myname'
   repository = 'myRepo'
   company = 'myCompany'
   tags = @('mydepartment')
   publishOnBuild = $true
   autoDocument = $true
   includeGitDetails = $true
   trimSpaces = $false
   removeEmptyLines = $true
   removeSingleLineQuotes = $false
   runPesterTests = $true
}
save-btDefaultSettings @myDefaults
```

### Create a new btProject

```powershell
md $newProjectFolder
cd $newProjectFolder

new-btProject -moduleName myBtProject -moduleDescription 'my bartender module'
```
![newproject](./gifs/newproject.gif)

## Start Coding

### Create a function

![function](./gifs/function.gif)

### Create Create a pester test
![pester](./gifs/pester1.gif)

### Build your first revision and test it out
![build1](./gifs/build1.gif)

### Increment your build/minor/major version to initiate a release build
![build1](./gifs/build2.gif)

### Check out your fancy automated markdown
![build1](./gifs/markdown.gif)



## What's it doing:

### New Projects

#### Folder Structure
Creating a new project with `new-btproject` will create the following folder structure:

```
├───documentation
├───postBuildScripts
├───rev
└───source
    ├───bin
    ├───classes
    ├───dscClasses
    ├───enums
    ├───filters
    ├───functions
    ├───lib
    ├───pester
    ├───private
    ├───resource
    └───validationClasses
```

#### Files

A few files will also be created, including:

 > A readme.md if one is not present

 > Some basice pester tests to ensure we are running a clean powershell window, with no previous commands or modules loaded
 
 > a .btIgnore and the .btOrderEnd + .btOrderStart files in each of the source directories


### Revisions

#### Compile

When running a standard revision build (simply by running the `start-btbuild`), all relevant ps1 files from the source folder (enums,classes,functions etc) will have their contents extracted and added to a single module-manifest in a new incremented revision folder.

> Subfolders can be used within the relevant folder (i.e. \functions\sub1, \functions\sub2) and they will still be compiled correctly

Files listed in the relevant .btIgnore will not be included in the compiled module

Files listed in .btOrderStart will have their contents added prior to other items in the folder, likewise files listed in .btOrderEnd will have their contents added post the other items in the folder.

> If required, you can add a file to both .btOrderStart and .btOrderEnd and it will bookend the other file contents.

Enums and Classes will be added to an appropriate ScriptsToProcess file within the module to handle the scoping issues of these things within the module manifest (i.e. they will be available to the command-line when importing the module)

#### Pester Test

This module will be loaded in a clean PowerShell job, and all scripts in the pester source folder will be executed against the revision module

#### Clean-Up

By default, only a limited number (5) of previous releases will be kept in the revision folder

### Release

#### Compile and Test

If incrementing a major, minor or build version with the `start-btbuild` and one of the following switch arguments `-incrementBuildVersion`,`-incrementMinorVersion`,`-incrementMajorVersion`, the revision process will take place, and if all tests are passed, it will create a new release version with the appropriate version.

The Module Manifest will be recreated using the `configuration` module

#### PostBuildScripts

At this point, scripts in the `postBuildScripts` will execute.

> This is the ideal place to put in things like poshpot permission insertion into your manifest, script signing etc.

#### PlatyPs

The `platyps` module will extract all the comment-based help from public functions and creating an appropriate version-documentation folder with the relevant markdown items.

A `release.md` file will also be created with some overall change stats, details on what source-files have changed, as well as links to any Markdown files created in the previous step

If you have the appropriate comment in your readme.md, bartender will update the header section with the latest release version and release date.

#### Publish

If your project is set to automatically publish, a `publish-module` command will be executed against the set repositories. It will use any saved credentials and tokens used with the `save-btrepository` command

### Misc

#### This isn't working and my repository requires credentials

PowerShellGet seems to be a bit finicky with the version of nuget especially if you have credential requirements on your repository.

PowerShellGet default version that comes with windows doesn't seem to support credentials at all, so you have two options

##### Option 1 (Preferred)
1. Update Package Management to the latest version

```
install-module packagemanagement
```
2. Force install powershellget to pull it from psGallery

```
Install-module powershellget
```

3. Install the nuget packageprovider (I'm using 2.8.5.208)

```
Install-PackageProvider -requiredversion 2.8.5.208
```

4. Restart your machine (Restarting PowerShell alone/removing and reloading all modules does not consistently work)



> Package Management comes with its own nuget provider which is > v3 , which also causes problems, to override it you must install the nuget packageprovider

##### Option 2

Add the credentials to your nuget profile

```
nuget sources add -name $repositoryName -source $uri -username $UserName -password $Password
```

#### Ok I've done all this and its still giving me an access denied when publishing

If your repository requires a username/password, your token may also need to include a username. (i.e. token = username:token)

Check your repository provider documentation


#### Thanks:

Joel Bennett - For the Configuration module

Pester Team - for Pester

PowerShell team - for the PlatyPs module

#### Example Bartender Projects

This module was built using itself, and therefore provides a good example

Additionally, for some projects built with Bartender see below:

|Project|Description|Link|
|-|-|-|
|psScreenRecorder|A PowerShell screen-capture tool using ffmpeg to make desktop videos|https://github.com/adrian-andersson/psScreenRecorder|
|psrapid|A PowerShell API Server|https://github.com/DomainGroupOSS/psrapid|
|domainAwsDSCResources|A couple of PowerShell Class DSC resources to help manage AWS instances|https://github.com/DomainGroupOSS/domainAwsDSCResources|
|domainAwsPowershelLTools|A collection of PowerShell functions for AWS|https://github.com/DomainGroupOSS/domainAwsPowershellTools|
|Poshbot.Democracy|A voting plugin for Poshbot|https://github.com/adrian-andersson/PoshBot.Democracy|


> If you have a project built with BarTender let us know by raising an issue. If we like it we will link it back in here

## Feedback? 

If you have any feedback (Good or Bad?) on this module, please let us know by raising an issue

***

Owned and maintained by Domain Group

[powershellbadge]: https://img.shields.io/static/v1.svg?label=language&message=PowerShell%20%3E_&color=blue&logo=powershell&style=plastic

