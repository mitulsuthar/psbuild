psbuild
=======

This project aims to make using MSBuild easier from powershell. The project has two main purposes

1. To make interacting with MSBuild files in PowerShell much easier
1. To make it easier to manipulate MSBuild files from PowerShell/NuGet

Currently psbuild is still a ***preview*** but should be stable enough for regular usage.

## Getting Started


##### download and install psbulid

    (new-object Net.WebClient).DownloadString("https://raw.github.com/ligershark/psbuild/master/src/GetPSBuild.ps1") | iex

##### build an msbuild file

    Invoke-MSBuild C:\temp\msbuild\msbuild.proj

##### build the file provided with the given parameters

    Invoke-MSBuild C:\temp\msbuild\path.proj -properties (@{'Configuration'='Release';'visualstudioversion'='12.0'}) -extraArgs '/nologo'

##### build an msbuild file and execute a specific target

    Invoke-MSBuild C:\temp\msbuild\proj1.proj -targets Demo

##### build an msbuild file and execute multiple targets

    Invoke-MSBuild C:\temp\msbuild\proj1.proj -targets @('Demo';'Demo2')

##### how to get the log file for the last build

    Invoke-MSBuild C:\temp\msbuild\proj1.proj
    # returns the detailed log by default
    Get-PSBuildLog

	# you can also open the log in the default editor
	Open-PSBuildLog

    # returns the diagnostic
    Get-PSBuildLog -logIndex 1
	
	Open-PSBuildLog -logIndex 1

#### show common msbuild escape characters
	Get-MSBuildEscapeCharacters

##### You can also create a new MSBuild file with the following

    New-Project | Save-Project -filePath .\new.proj

##### to see what commands are available

    Get-Command -Module psbuild

Most functions have help defined so you can use ```get-help``` on most commands for more details.

We have not yet developed the NuGet package yet but will be working on it soon.

## Debug mode
In many cases after a build it would be helpful to be able to answer questions like the following.
 
 - What is the value of `x` property?
 - What is the value of `y` property?
 - What would the expression ```'@(Compile->'%(FullPath)')``` be?

But when you call msbuild.exe the project that is built is created in memory and trashed at the end of the process. ```Invoke-MSBuild``` now has a way that you can invoke your build and then have a _"handle"_ to your project that was built. This allows you to ask questions like the following. To enable this you just need to pass in the ```-debugMode``` switch to ```Invoke-MSBuild``` (_Note: this is actively under development so if you run into an problems please open an issue_). Here are some examples of what you can do.

```
PS> $bResult = Invoke-MSBuild .\temp.proj -debugMode

PS> $bResult.EvalProperty('someprop')
default

PS> $bResult.EvalItem('someitem')
temp.proj

PS> $bResult.ExpandString('$(someprop)')
default

PS> $bResult.ExpandString('@(someitem->''$(someprop)\%(Filename)%(Extension)'')')
default\temp.proj
```

You can get full access to the [ProjectInstance](http://msdn.microsoft.com/en-us/library/microsoft.build.execution.projectinstance(v=vs.121).aspx) object with the ProjectInstance property.

More functionality is available via the ProjectInstance object.
``` 
PS> $bResult.ProjectInstance.GetItems('someitem').EvaluatedInclude
temp.proj
```

You can get the [BuildResuilt](http://msdn.microsoft.com/en-us/library/microsoft.build.execution.buildresult(v=vs.121).aspx) via the BuildResult parameter.

```
PS> $bResult.BuildResult.OverallResult
Failure
```

# Reporting Issues
To report any issues please create an item on the [issues page](https://github.com/sayedihashimi/psbuild/issues/new).

# Contributing
Contributing is pretty simple. The project mostly consists of one .psm1 file located at ```/src/psbuild.psm1```. Just modify that file with the updates and send me a Pull Request. I'll review it and work with you from there.