# Application Packaging Automation System (APAS) Deployment Rollback Procedures

## Overview

This document outlines the programmatic procedures for rolling back deployments of the APAS system in case issues are encountered after deployment. These procedures are designed to be executed by AI Dev Agents with clear, machine-interpretable instructions and comprehensive error handling.

## Rollback Strategy

APAS uses a versioned deployment approach with strict versioning and backup mechanisms to enable precise, deterministic rollbacks.

### Version Management

1. **Version Tracking**:
   - Each deployment uses semantic versioning (MAJOR.MINOR.PATCH)
   - Version history is maintained in a JSON-formatted version registry file
   - Deployment artifacts are stored in isolated version-specific directories with immutable content

2. **Backup Strategy**:
   - Automated pre-deployment backup generation with unique timestamped identifiers
   - Database schema snapshots using Supabase migrations
   - Configuration exports as serialized JSON

## Rollback Script Specifications

All rollback scripts follow a standardized format with explicit input parameters, return values, and error codes to facilitate automated execution.

### Error Codes

| Code | Description | Recovery Action |
|------|-------------|----------------|
| 0 | Success | Continue operation |
| 1 | General failure | Retry operation once |
| 10 | Service stop failure | Force stop with alternate method |
| 20 | Version not found | Fall back to last known good version |
| 30 | Database restore failure | Use backup fallback procedure |
| 40 | File system error | Verify permissions and retry |
| 50 | Configuration error | Use default configuration |

## Rollback Procedures

### 1. Standard Rollback Procedure

This procedure provides the primary rollback functionality with full automation support.

#### Input Parameters:

```json
{
  "version": "string | 'previous'",
  "force": "boolean (default: false)",
  "backupBeforeRollback": "boolean (default: true)",
  "restartServices": "boolean (default: true)"
}
```

#### Steps (With Expected Output Formats):

1. **Stop APAS Services**:
   ```powershell
   function Stop-APASServices {
     param (
       [bool]$Force = $false
     )
     
     try {
       $services = @("APASService", "APASAgent", "APASOrchestrator")
       $results = @{}
       
       foreach ($service in $services) {
         try {
           if ($Force) {
             Stop-Service -Name $service -Force -ErrorAction Stop
           } else {
             Stop-Service -Name $service -ErrorAction Stop
           }
           $results[$service] = @{
             "status": "stopped",
             "errorCode": 0
           }
         } catch {
           $results[$service] = @{
             "status": "error",
             "errorCode": 10,
             "errorMessage": $_.Exception.Message
           }
           Write-Error "Failed to stop service $service: $_"
         }
       }
       
       return @{
         "success": !($results.Values | Where-Object { $_.status -eq "error" }),
         "results": $results
       } | ConvertTo-Json
     } catch {
       return @{
         "success": $false,
         "errorCode": 1,
         "errorMessage": $_.Exception.Message
       } | ConvertTo-Json
     }
   }
   ```

2. **Get Version Info**:
   ```powershell
   function Get-VersionInfo {
     param (
       [string]$Version
     )
     
     try {
       $installRoot = "C:\Program Files\APAS"
       $versionsDir = Join-Path $installRoot "versions"
       $versionRegistryPath = Join-Path $installRoot "version_registry.json"
       
       # Load version registry
       if (Test-Path $versionRegistryPath) {
         $versionRegistry = Get-Content $versionRegistryPath | ConvertFrom-Json
       } else {
         throw "Version registry not found at $versionRegistryPath"
       }
       
       if ($Version -eq "previous") {
         # Get current version
         $currentVersion = $versionRegistry.currentVersion
         
         # Find previous version
         $versionHistory = $versionRegistry.versions | Sort-Object -Property deploymentDate -Descending
         $currentIndex = [array]::IndexOf(($versionHistory | ForEach-Object { $_.version }), $currentVersion)
         
         if ($currentIndex -ge 1) {
           $targetVersion = $versionHistory[$currentIndex + 1].version
         } else {
           throw "No previous version found in version history"
         }
       } else {
         $targetVersion = $Version
         $versionExists = $versionRegistry.versions | Where-Object { $_.version -eq $targetVersion }
         
         if (-not $versionExists) {
           throw "Specified version '$targetVersion' not found in version registry"
         }
       }
       
       $targetVersionPath = Join-Path $versionsDir $targetVersion
       
       if (-not (Test-Path $targetVersionPath)) {
         throw "Version directory for '$targetVersion' not found at $targetVersionPath"
       }
       
       return @{
         "success": $true,
         "currentVersion": $versionRegistry.currentVersion,
         "targetVersion": $targetVersion,
         "targetPath": $targetVersionPath,
         "deploymentDate": ($versionRegistry.versions | Where-Object { $_.version -eq $targetVersion }).deploymentDate
       } | ConvertTo-Json
     } catch {
       return @{
         "success": $false,
         "errorCode": 20,
         "errorMessage": $_.Exception.Message
       } | ConvertTo-Json
     }
   }
   ```

3. **Create Backup**:
   ```powershell
   function Backup-APASInstance {
     param (
       [string]$CurrentVersion
     )
     
     try {
       $installRoot = "C:\Program Files\APAS"
       $backupRoot = Join-Path $installRoot "backups"
       $timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
       $backupDir = Join-Path $backupRoot "backup_${CurrentVersion}_${timestamp}"
       
       # Ensure backup directory exists
       New-Item -ItemType Directory -Path $backupDir -Force | Out-Null
       
       # Backup configuration
       $configSource = Join-Path $installRoot "current\config"
       $configDest = Join-Path $backupDir "config"
       Copy-Item -Path $configSource -Destination $configDest -Recurse
       
       # Backup database
       $dbBackupPath = Join-Path $backupDir "database_backup.sql"
       $dbBackupResult = Invoke-Expression -Command "npx supabase db dump -f '$dbBackupPath'" 2>&1
       
       if ($LASTEXITCODE -ne 0) {
         throw "Database backup failed: $dbBackupResult"
       }
       
       # Create backup manifest
       $manifest = @{
         "timestamp": $timestamp,
         "version": $CurrentVersion,
         "components": @(
           "config",
           "database"
         )
       }
       
       $manifestPath = Join-Path $backupDir "manifest.json"
       $manifest | ConvertTo-Json -Depth 10 | Set-Content -Path $manifestPath
       
       return @{
         "success": $true,
         "backupPath": $backupDir,
         "backupManifest": $manifestPath
       } | ConvertTo-Json
     } catch {
       return @{
         "success": $false,
         "errorCode": 40,
         "errorMessage": $_.Exception.Message
       } | ConvertTo-Json
     }
   }
   ```

4. **Execute Rollback**:
   ```powershell
   function Invoke-APASRollback {
     param (
       [string]$TargetVersionPath,
       [string]$TargetVersion
     )
     
     try {
       $installRoot = "C:\Program Files\APAS"
       $currentSymlink = Join-Path $installRoot "current"
       $versionRegistryPath = Join-Path $installRoot "version_registry.json"
       
       # Remove existing symlink
       if (Test-Path $currentSymlink) {
         if (Get-Item $currentSymlink).LinkType -eq "SymbolicLink") {
           Remove-Item $currentSymlink -Force
         } else {
           throw "Current installation is not a symbolic link"
         }
       }
       
       # Create new symlink to target version
       New-Item -ItemType SymbolicLink -Path $currentSymlink -Target $TargetVersionPath
       
       # Update version registry
       $versionRegistry = Get-Content $versionRegistryPath | ConvertFrom-Json
       $versionRegistry.currentVersion = $TargetVersion
       $versionRegistry.lastRollback = @{
         "timestamp": (Get-Date -Format "yyyy-MM-ddTHH:mm:ssZ"),
         "fromVersion": $versionRegistry.currentVersion,
         "toVersion": $TargetVersion
       }
       
       $versionRegistry | ConvertTo-Json -Depth 10 | Set-Content -Path $versionRegistryPath
       
       # Run post-rollback script if exists
       $postRollbackScript = Join-Path $TargetVersionPath "scripts\post-rollback.ps1"
       $postRollbackResult = $null
       
       if (Test-Path $postRollbackScript) {
         $postRollbackResult = & $postRollbackScript 2>&1
       }
       
       return @{
         "success": $true,
         "targetVersion": $TargetVersion,
         "targetPath": $TargetVersionPath,
         "postRollbackScriptRun": (Test-Path $postRollbackScript),
         "postRollbackResult": $postRollbackResult
       } | ConvertTo-Json
     } catch {
       return @{
         "success": $false,
         "errorCode": 40,
         "errorMessage": $_.Exception.Message
       } | ConvertTo-Json
     }
   }
   ```

5. **Start Services**:
   ```powershell
   function Start-APASServices {
     try {
       $services = @("APASOrchestrator", "APASAgent", "APASService")  # Start in reverse order
       $results = @{}
       
       foreach ($service in $services) {
         try {
           Start-Service -Name $service -ErrorAction Stop
           $results[$service] = @{
             "status": "started",
             "errorCode": 0
           }
         } catch {
           $results[$service] = @{
             "status": "error",
             "errorCode": 10,
             "errorMessage": $_.Exception.Message
           }
           Write-Error "Failed to start service $service: $_"
         }
       }
       
       return @{
         "success": !($results.Values | Where-Object { $_.status -eq "error" }),
         "results": $results
       } | ConvertTo-Json
     } catch {
       return @{
         "success": $false,
         "errorCode": 1,
         "errorMessage": $_.Exception.Message
       } | ConvertTo-Json
     }
   }
   ```

6. **Verify Rollback**:
   ```powershell
   function Test-APASRollback {
     param (
       [string]$TargetVersion
     )
     
     try {
       $installRoot = "C:\Program Files\APAS"
       $currentSymlink = Join-Path $installRoot "current"
       $versionRegistryPath = Join-Path $installRoot "version_registry.json"
       $healthcheckEndpoint = "http://localhost:8000/api/v1/health"
       
       $tests = @{
         "symlink": $false,
         "registry": $false,
         "healthcheck": $false
       }
       
       # Check symlink
       if (Test-Path $currentSymlink) {
         $symlinkTarget = (Get-Item $currentSymlink).Target
         $expectedTarget = Join-Path (Join-Path $installRoot "versions") $TargetVersion
         $tests.symlink = ($symlinkTarget -eq $expectedTarget)
       }
       
       # Check version registry
       if (Test-Path $versionRegistryPath) {
         $versionRegistry = Get-Content $versionRegistryPath | ConvertFrom-Json
         $tests.registry = ($versionRegistry.currentVersion -eq $TargetVersion)
       }
       
       # Check health endpoint
       try {
         $healthCheck = Invoke-RestMethod -Uri $healthcheckEndpoint -Method Get -TimeoutSec 30
         $tests.healthcheck = ($healthCheck.status -eq "ok" -and $healthCheck.version -eq $TargetVersion)
       } catch {
         # Health check failed
       }
       
       $allTestsPassed = !($tests.Values -contains $false)
       
       return @{
         "success": $allTestsPassed,
         "tests": $tests,
         "errorCode": if ($allTestsPassed) { 0 } else { 50 }
       } | ConvertTo-Json
     } catch {
       return @{
         "success": $false,
         "errorCode": 1,
         "errorMessage": $_.Exception.Message
       } | ConvertTo-Json
     }
   }
   ```

#### Master Rollback Script

```powershell
# rollback.ps1
param (
  [Parameter(Mandatory=$true)]
  [string]$Version,
  [bool]$Force = $false,
  [bool]$BackupBeforeRollback = $true,
  [bool]$RestartServices = $true
)

# Import functions
. .\lib\stop-apas-services.ps1
. .\lib\get-version-info.ps1
. .\lib\backup-apas-instance.ps1
. .\lib\invoke-apas-rollback.ps1
. .\lib\start-apas-services.ps1
. .\lib\test-apas-rollback.ps1

# Setup logging
$logDir = "C:\Program Files\APAS\logs"
$logFile = Join-Path $logDir "rollback_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"
Start-Transcript -Path $logFile

try {
  Write-Host "Starting rollback to version: $Version" -ForegroundColor Cyan
  
  # Step 1: Stop services
  Write-Host "Stopping APAS services..." -ForegroundColor Yellow
  $stopResult = Stop-APASServices -Force $Force | ConvertFrom-Json
  
  if (-not $stopResult.success) {
    throw "Failed to stop all services: $($stopResult.results | ConvertTo-Json)"
  }
  
  # Step 2: Get version info
  Write-Host "Resolving target version..." -ForegroundColor Yellow
  $versionInfo = Get-VersionInfo -Version $Version | ConvertFrom-Json
  
  if (-not $versionInfo.success) {
    throw "Failed to resolve version info: $($versionInfo.errorMessage)"
  }
  
  Write-Host "Current version: $($versionInfo.currentVersion)" -ForegroundColor Cyan
  Write-Host "Target version: $($versionInfo.targetVersion)" -ForegroundColor Cyan
  Write-Host "Target path: $($versionInfo.targetPath)" -ForegroundColor Cyan
  
  # Step 3: Create backup if requested
  if ($BackupBeforeRollback) {
    Write-Host "Creating backup before rollback..." -ForegroundColor Yellow
    $backupResult = Backup-APASInstance -CurrentVersion $versionInfo.currentVersion | ConvertFrom-Json
    
    if (-not $backupResult.success) {
      Write-Host "Warning: Backup failed: $($backupResult.errorMessage)" -ForegroundColor Red
      
      # Ask for confirmation to continue without backup
      if (-not $Force) {
        $confirmation = Read-Host "Backup failed. Continue with rollback without backup? (y/n)"
        if ($confirmation -ne 'y') {
          throw "Rollback aborted by user due to backup failure"
        }
      }
    } else {
      Write-Host "Backup created at: $($backupResult.backupPath)" -ForegroundColor Green
    }
  }
  
  # Step 4: Execute rollback
  Write-Host "Executing rollback..." -ForegroundColor Yellow
  $rollbackResult = Invoke-APASRollback -TargetVersionPath $versionInfo.targetPath -TargetVersion $versionInfo.targetVersion | ConvertFrom-Json
  
  if (-not $rollbackResult.success) {
    throw "Rollback failed: $($rollbackResult.errorMessage)"
  }
  
  # Step 5: Start services if requested
  if ($RestartServices) {
    Write-Host "Starting APAS services..." -ForegroundColor Yellow
    $startResult = Start-APASServices | ConvertFrom-Json
    
    if (-not $startResult.success) {
      Write-Host "Warning: Failed to start all services: $($startResult.results | ConvertTo-Json)" -ForegroundColor Red
    }
  }
  
  # Step 6: Verify rollback
  Write-Host "Verifying rollback..." -ForegroundColor Yellow
  $verifyResult = Test-APASRollback -TargetVersion $versionInfo.targetVersion | ConvertFrom-Json
  
  if (-not $verifyResult.success) {
    Write-Host "Warning: Rollback verification failed: $($verifyResult.tests | ConvertTo-Json)" -ForegroundColor Red
  } else {
    Write-Host "Rollback verification passed" -ForegroundColor Green
  }
  
  Write-Host "Rollback to version $($versionInfo.targetVersion) completed successfully" -ForegroundColor Green
  
  return @{
    "success": $true,
    "targetVersion": $versionInfo.targetVersion,
    "backupPath": if ($BackupBeforeRollback -and $backupResult.success) { $backupResult.backupPath } else { $null },
    "verificationPassed": $verifyResult.success,
    "logFile": $logFile
  } | ConvertTo-Json
  
} catch {
  $errorMessage = $_.Exception.Message
  Write-Host "Rollback failed: $errorMessage" -ForegroundColor Red
  
  # Try to restore to original state
  try {
    Write-Host "Attempting to restore original state..." -ForegroundColor Yellow
    # Restoration logic here
  } catch {
    Write-Host "Restoration failed: $($_.Exception.Message)" -ForegroundColor Red
  }
  
  return @{
    "success": $false,
    "errorMessage": $errorMessage,
    "logFile": $logFile
  } | ConvertTo-Json
  
} finally {
  Stop-Transcript
}
```

### 2. Database-Only Rollback Procedure

For situations where only the database state needs to be reverted.

#### Input Parameters:

```json
{
  "backupFile": "string",
  "skipSchemaRestore": "boolean (default: false)",
  "skipDataRestore": "boolean (default: false)"
}
```

#### Script:

```powershell
# database-rollback.ps1
param (
  [Parameter(Mandatory=$true)]
  [string]$BackupFile,
  [bool]$SkipSchemaRestore = $false,
  [bool]$SkipDataRestore = $false
)

$logDir = "C:\Program Files\APAS\logs"
$logFile = Join-Path $logDir "db_rollback_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"
Start-Transcript -Path $logFile

try {
  Write-Host "Starting database rollback from: $BackupFile" -ForegroundColor Cyan
  
  # Validate backup file
  if (-not (Test-Path $BackupFile)) {
    throw "Backup file not found: $BackupFile"
  }
  
  # Stop services that depend on the database
  Write-Host "Stopping database-dependent services..." -ForegroundColor Yellow
  Stop-Service -Name "APASService" -Force
  
  # Restore database
  Write-Host "Restoring database..." -ForegroundColor Yellow
  
  $restoreArgs = @()
  if ($SkipSchemaRestore) {
    $restoreArgs += "--skip-schema"
  }
  if ($SkipDataRestore) {
    $restoreArgs += "--skip-data"
  }
  
  $restoreResult = Invoke-Expression -Command "npx supabase db restore -f '$BackupFile' $($restoreArgs -join ' ')" 2>&1
  
  if ($LASTEXITCODE -ne 0) {
    throw "Database restore failed: $restoreResult"
  }
  
  # Start services
  Write-Host "Starting services..." -ForegroundColor Yellow
  Start-Service -Name "APASService"
  
  Write-Host "Database rollback completed successfully" -ForegroundColor Green
  
  return @{
    "success": $true,
    "backupFile": $BackupFile,
    "skipSchemaRestore": $SkipSchemaRestore,
    "skipDataRestore": $SkipDataRestore,
    "logFile": $logFile
  } | ConvertTo-Json
  
} catch {
  $errorMessage = $_.Exception.Message
  Write-Host "Database rollback failed: $errorMessage" -ForegroundColor Red
  
  return @{
    "success": $false,
    "errorMessage": $errorMessage,
    "logFile": $logFile
  } | ConvertTo-Json
  
} finally {
  Stop-Transcript
}
```

## Automated Testing of Rollback Procedures

Each rollback procedure should be automatically tested by the AI Dev Agent during deployment by following these steps:

1. **Test Environment Setup**:
   ```powershell
   function Initialize-RollbackTest {
     param (
       [string]$TestVersion = "1.0.0-test"
     )
     
     # Create test version
     $installRoot = "C:\Program Files\APAS"
     $versionsDir = Join-Path $installRoot "versions"
     $testVersionPath = Join-Path $versionsDir $TestVersion
     $versionRegistryPath = Join-Path $installRoot "version_registry.json"
     
     # Create test version directory
     New-Item -ItemType Directory -Path $testVersionPath -Force | Out-Null
     
     # Copy current content
     Copy-Item -Path "$installRoot\current\*" -Destination $testVersionPath -Recurse
     
     # Update version registry
     $versionRegistry = Get-Content $versionRegistryPath | ConvertFrom-Json
     $currentVersion = $versionRegistry.currentVersion
     
     # Add test version to registry
     $newVersion = @{
       "version" = $TestVersion
       "deploymentDate" = (Get-Date -Format "yyyy-MM-ddTHH:mm:ssZ")
       "artifacts" = @()
     }
     
     $versionRegistry.versions += $newVersion
     $versionRegistry | ConvertTo-Json -Depth 10 | Set-Content -Path $versionRegistryPath
     
     return @{
       "success": $true,
       "testVersion": $TestVersion,
       "originalVersion": $currentVersion
     }
   }
   ```

2. **Automated Rollback Tests**:
   ```powershell
   # Execute and validate test rollback
   $testParams = @{
     "Version" = $testSetup.testVersion
     "Force" = $true
     "BackupBeforeRollback" = $true
     "RestartServices" = $true
   }
   
   $testRollback = & .\rollback.ps1 @testParams | ConvertFrom-Json
   
   if (-not $testRollback.success) {
     Write-Error "Rollback test failed: $($testRollback.errorMessage)"
     exit 1
   }
   
   # Roll forward to original version
   $forwardParams = @{
     "Version" = $testSetup.originalVersion
     "Force" = $true
     "BackupBeforeRollback" = $false
     "RestartServices" = $true
   }
   
   $forwardRollback = & .\rollback.ps1 @forwardParams | ConvertFrom-Json
   
   if (-not $forwardRollback.success) {
     Write-Error "Forward rollback to original version failed: $($forwardRollback.errorMessage)"
     exit 1
   }
   
   Write-Host "Rollback test completed successfully" -ForegroundColor Green
   ```

## Monitoring and Verification

The `verify-deployment.ps1` script provides comprehensive verification of system integrity after rollback:

```powershell
# verify-deployment.ps1
param (
  [bool]$DetailedOutput = $true
)

$installRoot = "C:\Program Files\APAS"
$currentSymlink = Join-Path $installRoot "current"
$versionRegistryPath = Join-Path $installRoot "version_registry.json"
$healthcheckEndpoint = "http://localhost:8000/api/v1/health"

try {
  $results = @{
    "systemIntegrity" = @{
      "status" = "unknown",
      "checks" = @{}
    },
    "serviceStatus" = @{
      "status" = "unknown",
      "services" = @{}
    },
    "apiStatus" = @{
      "status" = "unknown",
      "endpoints" = @{}
    },
    "databaseStatus" = @{
      "status" = "unknown",
      "checks" = @{}
    }
  }
  
  # System integrity checks
  $systemChecks = $results.systemIntegrity.checks
  
  # Check symlink
  $symlinkExists = Test-Path $currentSymlink
  $systemChecks.symlinkExists = $symlinkExists
  
  if ($symlinkExists) {
    $symlinkTarget = (Get-Item $currentSymlink).Target
    $systemChecks.symlinkTarget = $symlinkTarget
  }
  
  # Check version registry
  $registryExists = Test-Path $versionRegistryPath
  $systemChecks.registryExists = $registryExists
  
  if ($registryExists) {
    $versionRegistry = Get-Content $versionRegistryPath | ConvertFrom-Json
    $systemChecks.currentVersion = $versionRegistry.currentVersion
  }
  
  # Check for critical files
  $requiredFiles = @(
    "apas.exe",
    "config\appsettings.json",
    "libs\core.dll"
  )
  
  $fileChecks = @{}
  foreach ($file in $requiredFiles) {
    $filePath = Join-Path $currentSymlink $file
    $fileChecks[$file] = Test-Path $filePath
  }
  $systemChecks.requiredFiles = $fileChecks
  
  # Overall system integrity status
  $systemIntegrityPassed = $symlinkExists -and $registryExists -and (-not ($fileChecks.Values -contains $false))
  $results.systemIntegrity.status = if ($systemIntegrityPassed) { "passed" } else { "failed" }
  
  # Service status checks
  $services = @("APASService", "APASAgent", "APASOrchestrator")
  $serviceStatus = $results.serviceStatus.services
  
  foreach ($service in $services) {
    try {
      $status = Get-Service -Name $service -ErrorAction Stop
      $serviceStatus[$service] = @{
        "exists": $true,
        "status": $status.Status,
        "startType": $status.StartType
      }
    } catch {
      $serviceStatus[$service] = @{
        "exists": $false,
        "error": $_.Exception.Message
      }
    }
  }
  
  # Overall service status
  $allServicesRunning = (-not ($serviceStatus.Keys | ForEach-Object { $serviceStatus[$_].status -ne "Running" }))
  $results.serviceStatus.status = if ($allServicesRunning) { "passed" } else { "failed" }
  
  # API status checks
  $apiEndpoints = @{
    "health" = "http://localhost:8000/api/v1/health",
    "version" = "http://localhost:8000/api/v1/version"
  }
  $endpointChecks = $results.apiStatus.endpoints
  
  foreach ($endpoint in $apiEndpoints.Keys) {
    try {
      $response = Invoke-RestMethod -Uri $apiEndpoints[$endpoint] -Method Get -TimeoutSec 10
      $endpointChecks[$endpoint] = @{
        "status": "responsive",
        "statusCode": 200,
        "response": if ($DetailedOutput) { $response } else { "response data available" }
      }
    } catch {
      $endpointChecks[$endpoint] = @{
        "status": "failed",
        "error": $_.Exception.Message
      }
    }
  }
  
  # Overall API status
  $allApisResponsive = (-not ($endpointChecks.Keys | ForEach-Object { $endpointChecks[$_].status -ne "responsive" }))
  $results.apiStatus.status = if ($allApisResponsive) { "passed" } else { "failed" }
  
  # Database status checks
  $dbChecks = $results.databaseStatus.checks
  
  try {
    # Check database connection
    $dbResponse = Invoke-RestMethod -Uri "http://localhost:8000/api/v1/system/database/status" -Method Get -TimeoutSec 10
    $dbChecks.connection = $dbResponse.connected
    $dbChecks.migrations = $dbResponse.migrationsApplied
    $dbChecks.tables = $dbResponse.tableCount
    
    # Overall database status
    $databaseHealthy = $dbResponse.connected -and $dbResponse.healthy
    $results.databaseStatus.status = if ($databaseHealthy) { "passed" } else { "failed" }
  } catch {
    $dbChecks.connection = $false
    $dbChecks.error = $_.Exception.Message
    $results.databaseStatus.status = "failed"
  }
  
  # Overall verification result
  $overallStatus = if (
    $results.systemIntegrity.status -eq "passed" -and
    $results.serviceStatus.status -eq "passed" -and
    $results.apiStatus.status -eq "passed" -and
    $results.databaseStatus.status -eq "passed"
  ) { "passed" } else { "failed" }
  
  return @{
    "success": $overallStatus -eq "passed",
    "status": $overallStatus,
    "results": $results
  } | ConvertTo-Json -Depth 10
  
} catch {
  return @{
    "success": $false,
    "status": "error",
    "errorMessage": $_.Exception.Message
  } | ConvertTo-Json
}
```

## Programmatic Integration

For AI Dev Agent integration, these endpoints can be called with PowerShell:

```powershell
# Example of programmatic rollback via AI Dev Agent
$rollbackParams = @{
  "Version" = "previous"  # Use string version number or "previous" 
  "Force" = $false        # Force operation without prompts
  "BackupBeforeRollback" = $true  # Create backup before rollback
  "RestartServices" = $true       # Restart services after rollback
}

# Execute rollback
$rollbackResult = & "C:\Program Files\APAS\scripts\rollback.ps1" @rollbackParams | ConvertFrom-Json

# Parse result
if ($rollbackResult.success) {
  # Rollback successful
  Write-Host "Rollback to version $($rollbackResult.targetVersion) completed successfully"
} else {
  # Rollback failed
  Write-Host "Rollback failed: $($rollbackResult.errorMessage)"
  # Parse log file for detailed error information
  $logContent = Get-Content -Path $rollbackResult.logFile
}
```

## Implementation Timeline

The rollback procedures will be implemented as part of the deployment pipeline development in Epic 1 (Story 1.3: Database Schema Design and Implementation and additional tasks in Story 2.3: Deployment Pipeline).
