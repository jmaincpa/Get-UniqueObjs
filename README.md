# Get-UniqueObjs
Filter a list of Powershell Objects by one or more properties

```Powershell
Function Get-UniqueObjs {
    <#
    .SYNOPSIS
        Return a list of Powershell Objects, unique by one or more properties
    .DESCRIPTION
        Accepts a list of objects, groups them by one or more properties, and returns the first object from each group
    .EXAMPLE
        Example using PowerCLI objects
        PS C:\> (Get-VMHost).ExtensionData.config.storageDevice.ScsiLun | Get-UniqueObjs -property Key

        Example returns a unique list of SCSI LUN objects for the entire Virtual Center
    .INPUTS
        List of Objects
            If a string or non-PSObject type is submitted, all objects are converted to strings and the property param is ignored
        List of Properties
            if $null, will use the object Name property
    .OUTPUTS
        Unique list from Inputs
    .NOTES
        John Main 2020-07-25
    #>
    [CmdletBinding()]
    param (
        [parameter(Mandatory = $false, ValueFromPipeline = $true)]
        [object[]]$inputObjects,
        [string[]]$property,
        [switch]$passThru
    )
    Begin {
        $list = @()
        if (-not $property) {
            $property = "Name"
        }
        $simpleItemSubmitted = $false
    }
    Process {
        foreach ($inputObject in $inputObjects | where { $_ }) {
            if ($inputObject -isnot [psobject]) { $simpleItemSubmitted = $true }
            $list += $inputObject
        }
    }
    End {
        if ($list -and $list.count -gt 0) {
            if ($simpleItemSubmitted) {
                $list = $list | % { "$_" } | Get-Unique -AsString
            } else {
                $list | Group-Object -Property $property -ErrorAction Ignore | % { $_.group | select -First 1 }
            }
        }
    }
}

```
