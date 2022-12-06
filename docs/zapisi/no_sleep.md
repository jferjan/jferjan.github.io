---
tags:
  - IT
---

# Powershelll skripta za preprečevanje neaktivnosti/spanja mašine

Skripta vsake štiri minute simulira vklop/izklop scrool locka. Ob delovanju piše log na stdout, da vidimo da deluje. Delovanje se vidi tudi po kratkem utripu scrool-lock lučke vsake 4 minute.

## 1. Pripravi skripto

Naredi si datoteko (skripto) `nosleep.ps1` z vsebino:

``` powershell	
#
# -- NoSleep --
# Keep your computer awake by programmatically pressing the ScrollLock key every X seconds
#
param($sleep = 240) # seconds
$announcementInterval = 10 # loops

Clear-Host

$WShell = New-Object -com "Wscript.Shell"
$date = Get-Date -Format "dddd MM/dd HH:mm (K)"

$stopwatch
# Some environments don't support invocation of this method.
try {
  $stopwatch = [system.diagnostics.stopwatch]::StartNew()
} catch {
  Write-Host "Couldn't start the stopwatch."
}

Write-Host "Executing ScrollLock-toggle NoSleep routine."
Write-Host "Start time:" $(Get-Date -Format "dddd MM/dd HH:mm (K)")
Write-Host "<3" -fore red

$index = 0
while ( $true )
{
  Write-Host "< 3" -fore red      # heartbeat
  $WShell.sendkeys("{SCROLLLOCK}")
  Start-Sleep -Milliseconds 200
  $WShell.sendkeys("{SCROLLLOCK}")
  Write-Host "<3" -fore red       # heartbeat
  Start-Sleep -Seconds $sleep
  # Announce runtime on an interval
  if ( $stopwatch.IsRunning -and (++$index % $announcementInterval) -eq 0 )
  {
    Write-Host "Elapsed time: " $stopwatch.Elapsed.ToString('dd\.hh\:mm\:ss')
  }
}
```	
	
## 2. Nastavi pravice izvajanja

Odpri powershell in nastavi pravice izvajanja. 

Pravice nastavimo z ukazom:
	
    Set-ExecutionPolicy -Scope CurrentUser RemoteSigned 
	
## 3. Poženi skripto
	
Poženi skripto z ukazom:

    .\nosleep.ps1


## 4. Bližnjica za zagon skripte

Namesto korakov 2. in 3. si lahko narediš bližnjico.

Za cilj bljižnice navedi:

    C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -noexit -ExecutionPolicy Bypass -File C:\Users\username\NoSleep\nosleep.ps1

!!! note

    Nastavi ustrezno pot do datoteke `powershell.exe` in `nosleep.ps1`.