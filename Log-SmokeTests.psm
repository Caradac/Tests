#Requires -Version 7.0
#Requires -RunAsAdministrator

<#
.SYNOPSIS
    Create Global variables and initialise/write to log files for HCP Smoke Tests

.DESCRIPTION
    Detailed description of the module's functionality, features, and usage.

.NOTES

#>

# Module parameter(s)
param(
    [bool]$NoInit = $false
)

# Module variables
$script:ModuleRoot = Split-Path -Parent $MyInvocation.MyCommand.Path
$script:ModuleVersion = '1.0.0'

# Dot-source function files
$FunctionPath = Join-Path -Path $script:ModuleRoot -ChildPath 'Functions'
if (Test-Path -Path $FunctionPath) {
    Get-ChildItem -Path $FunctionPath -Filter '*.ps1' | ForEach-Object {
        . $_.FullName
    }
}



function Set-GlobalScriptInfo {
    <#
    .SYNOPSIS
        Sets global variables for the invoking script name and date, and constructs log and result file paths based on the script location.
    #>
    param()

    $callerScript = $null
    try {
        $stack = Get-PSCallStack
        if ($stack) {
            $modulePath = $MyInvocation.MyCommand.Path
            for ($i = $stack.Count - 1; $i -ge 0; $i--) {
                $frame = $stack[$i]
                if ($frame.ScriptName -and ($frame.ScriptName -ne '')) {
                    if ($modulePath) {
                        if ($frame.ScriptName -ne $modulePath) {
                            $callerScript = $frame.ScriptName
                            break
                        }
                    } else {
                        $callerScript = $frame.ScriptName
                        break
                    }
                }
            }
        }
    } catch {
        # Ignore and fall back to other methods
    }

    if (-not $callerScript) {
        if ($PSCommandPath) { $callerScript = $PSCommandPath }
        elseif ($MyInvocation.MyCommand.Path) { $callerScript = $MyInvocation.MyCommand.Path }
        elseif ($PSScriptRoot) { $callerScript = $PSScriptRoot }
    }

    $scriptName = if ($callerScript) { Split-Path -Leaf $callerScript } else { $null }
    $scriptPath = if ($callerScript) { $callerScript } else { $null }
    $scriptBasePath = if ($scriptPath) { Split-Path -Parent $scriptPath } else { $null }
    $timeStamp = (Get-Date).ToString('yyyyMMddHHmmss')
    $logPath = if ($scriptBasePath) { Join-Path -Path $scriptBasePath -ChildPath 'Logs' } else { $null }
    $logFileName = if ($logPath -and $scriptName) { Join-Path -Path $logPath -ChildPath ("{0}_{1}.log" -f $scriptName, $timeStamp) } else { $null }
    $resultPath = if ($scriptBasePath) { Join-Path -Path $scriptBasePath -ChildPath 'Results' } else { $null }
    $resultFileName = if ($resultPath -and $scriptName) { Join-Path -Path $resultPath -ChildPath ("{0}_{1}_Results.json" -f $scriptName, $timeStamp) } else { $null }
    
    Set-Variable -Name 'SmokeTestScriptName' -Value $scriptName -Scope Global -Force
    Set-Variable -Name 'SmokeTestScriptPath' -Value $scriptPath -Scope Global -Force
    Set-Variable -Name 'SmokeTestScriptBasePath' -Value $scriptBasePath -Scope Global -Force
    Set-Variable -Name 'SmokeTestTimeStamp' -Value $timeStamp -Scope Global -Force
    Set-Variable -Name 'SmokeTestLogPath' -Value $logPath -Scope Global -Force
    Set-Variable -Name 'SmokeTestLogFileName' -Value $logFileName -Scope Global -Force
    Set-Variable -Name 'SmokeTestResultPath' -Value $resultPath -Scope Global -Force
    Set-Variable -Name 'SmokeTestResultFileName' -Value $resultFileName -Scope Global -Force
    
}

function Initialize-LogFiles {
    <#
    .SYNOPSIS
        Initializes log and result directories and files.
    #>
    param()

    if (-not (Test-Path -Path $SmokeTestLogPath)) {
        New-Item -Path $SmokeTestLogPath -ItemType Directory -Force | Out-Null
    }

    if (-not (Test-Path -Path $SmokeTestResultPath)) {
       New-Item -Path $SmokeTestResultPath -ItemType Directory -Force | Out-Null
    }

        # Create empty log file
    if (-not (Test-Path -Path $SmokeTestLogFileName)) {
        New-Item -Path $SmokeTestLogFileName -ItemType File -Force | Out-Null
    }
            # Create empty result file
    if (-not (Test-Path -Path $SmokeTestResultFileName)) {
        New-Item -Path $SmokeTestResultFileName -ItemType File -Force | Out-Null
    }   
}

function Write-Log {
    <#
    .SYNOPSIS
        Appends a log entry to the log file with timestamp, source, status, detail, and notes.
    .PARAMETER Source
        The source or test name.
    .PARAMETER Status
        Boolean status ($true for pass, $false for fail).
    .PARAMETER Detail
        Detailed description of the log entry.
    .PARAMETER Notes
        Additional notes or comments.
    #>
    param(
        [Parameter(Mandatory=$true)]
        [string]$Source,
        
        [Parameter(Mandatory=$true)]
        [bool]$Status,
        
        [Parameter(Mandatory=$true)]
        [string]$Detail,
        
        [Parameter(Mandatory=$false)]
        [string]$Notes
    )

    # Create timestamp for this log entry
    $logTimeStamp = (Get-Date).ToString('yyyy-MM-dd HH:mm:ss')
    
    # Helper function to escape CSV fields (wrap in quotes and escape internal quotes)
    $escapeField = {
        param([string]$field)
        if ($field) {
            '"' + ($field -replace '"', '""') + '"'
        } else {
            '""'
        }
    }
    
    # Format the log line: logtimestamp, source, status, detail, notes
    $logLine = @(
        & $escapeField $logTimeStamp
        & $escapeField $Source
        & $escapeField ($Status.ToString())
        & $escapeField $Detail
        & $escapeField $Notes
    ) -join ','
    
    # Append to log file if it exists
    if ($SmokeTestLogFileName -and (Test-Path -Path $SmokeTestLogFileName)) {
        Add-Content -Path $SmokeTestLogFileName -Value $logLine -Encoding UTF8
    } else {
        Write-Warning "Log file not initialized: $SmokeTestLogFileName"
    }
}

# If NoInit is not requested, set up globals and initialise files on module import.
if (-not $NoInit) {
    # set global script info on module load to ensure variables are available for any script that imports the module, even if they don't call Set-GlobalScriptInfo directly
    Set-GlobalScriptInfo
    # Initialize log and result files on module load to ensure they are created and available for any script that imports the module, even if they don't call Initialize-LogFiles directly
    Initialize-LogFiles
}

# Module initialization

# Export functions for module users
Export-ModuleMember -Function @(
    'Set-GlobalScriptInfo',
    'Initialize-LogFiles',
    'Write-Log'
)

Write-Verbose "Module loaded: PSM Module v$script:ModuleVersion"

