---
title: Coding Vibes
date: 2025-07-14
layout: post
categories: [post]
---

And I'm back! A few days after posting about the Screen Timer AHK code (May 2025), I had to update it. My son found a loophole where rebooting the PC would reset the timer, making for an unlimited play time. So, I consulted Grok once more to help me with my code. Like a coding genie to whom I would simply describe what I wanted and out goes the code. I would try the code, reported any errors, clarify some more if he gets my requests wrong, and so on... This goes on for a few more rounds until finally, a version that I am satisfied with. The timer value is being saved to a local ini file every few seconds preserving the countdown between reboots.

This, I realized, is what they call vibe coding, and dang it works.   

<br>
```
; AutoHotkey script to limit Roblox playtime with persistent timer data
; Prompt by jrapisora, coded by Grok3
#Requires AutoHotkey v2.0
#SingleInstance Force

; Set timer duration in hours
global TimerDuration := 1 ; in hours
global TimeUpTriggered := false ; Tracks if timer has expired
global TriggerDate := GetCurrentDate() ; Initialize with valid date
global DailyConsumedTime := 0 ; Tracks total Roblox time today (seconds)
global ProgressGui ; GUI instance
global DataFile := A_ScriptDir . "\RobloxTimerData.ini" ; File to store timer data

; Load saved timer data on startup
LoadTimerData()

; Check and reset timer if it's a new day at script startup
ResetIfNewDay()

; Monitor for Roblox
Loop {
    WinWait "ahk_exe RobloxPlayerBeta.exe"
    
    ; Check date and reset if new day
    ResetIfNewDay()
    
    ; Calculate remaining time
    TotalTime := TimerDuration * 3600 - DailyConsumedTime
    if (TotalTime <= 0 || TimeUpTriggered) {
        ShowTimeUpGui()
        WinWaitClose "ahk_exe RobloxPlayerBeta.exe"
        CleanupGui()
        SaveTimerData() ; Save state after Roblox closes
        continue
    }
    
    ; Create progress GUI in lower left
    ProgressGui := Gui("+AlwaysOnTop -Caption +ToolWindow", "Timer")
    ProgressGui.Add("Progress", "w200 h20 vMyProgress", 0)
    ; Initial time-left text in HH:MM:SS
    Hours := Floor(TotalTime / 3600)
    Minutes := Floor(Mod(TotalTime, 3600) / 60)
    Seconds := Mod(TotalTime, 60)
    ProgressGui.Add("Text", "w200 vTimeLeft", Format("{:d}:{:02d}:{:02d} left", Hours, Minutes, Seconds))
    ScreenHeight := SysGet(1)
    ProgressGui.Show("x0 y" . (ScreenHeight - 60))
    
    ; Start timer
    global SessionStartTime := DailyConsumedTime
    SetTimer(UpdateProgress, 1000)
    
    ; Wait for Roblox to close
    WinWaitClose "ahk_exe RobloxPlayerBeta.exe"
    CleanupGui()
    SaveTimerData() ; Save state after Roblox closes
}

; Show "Time is up!" GUI centered
ShowTimeUpGui() {
    global ProgressGui, TimerDuration
    ProgressGui := Gui("+AlwaysOnTop -Caption +ToolWindow", "Timer")
    ProgressGui.SetFont("s16 Bold")
    ProgressGui.Add("Text", "y70 Center w600", "Done for today! Now go out and touch grass =)")
    ProgressGui.Add("Text", "Center w600", "-Love, Papa")    
    ProgressGui.Show("w600 h200 Center")
}

; Clean up GUI and stop timer
CleanupGui() {
    global ProgressGui
    SetTimer(UpdateProgress, 0)
    if (IsSet(ProgressGui) && ProgressGui)
        ProgressGui.Destroy()
}

; Timer function to update progress and time left
UpdateProgress() {
    global ProgressGui, TimerDuration, TimeUpTriggered, TriggerDate, DailyConsumedTime, SessionStartTime
    DailyConsumedTime++ ; Increment total time used
    TotalTime := TimerDuration * 3600 - DailyConsumedTime
    Percent := (DailyConsumedTime / (TimerDuration * 3600)) * 100
    
    SaveTimerData() ; Save state every second
    
    if (TotalTime <= 0) {
        CleanupGui()
        ShowTimeUpGui()
        TimeUpTriggered := true
        TriggerDate := GetCurrentDate()
        SaveTimerData() ; Save state after timer expires
    } else {
        if (IsSet(ProgressGui) && ProgressGui) {
            ProgressGui["MyProgress"].Value := Percent
            Hours := Floor(TotalTime / 3600)
            Minutes := Floor(Mod(TotalTime, 3600) / 60)
            Seconds := Mod(TotalTime, 60)
            ProgressGui["TimeLeft"].Value := Format("{:d}:{:02d}:{:02d} left", Hours, Minutes, Seconds)
        }
    }
}

; Get current date in YYYYMMDD format with fallback
GetCurrentDate() {
    local date := FormatTime(A_Now, "YYYYMMDD")
    ; Validate: must be 8 digits
    if RegExMatch(date, "^\d{8}$") {
        return date
    }
    ; Fallback: Manually construct date
    local year := SubStr(A_Now, 1, 4)
    local month := SubStr(A_Now, 5, 2)
    local day := SubStr(A_Now, 7, 2)
    date := year . month . day
    if RegExMatch(date, "^\d{8}$") {
        return date
    }
    ; Last resort: Hardcode a safe date
    date := "20250512"
    return date
}

; Load timer data from INI file with TriggerDate validation
LoadTimerData() {
    global DailyConsumedTime, TriggerDate, TimeUpTriggered, DataFile
    if FileExist(DataFile) {
        DailyConsumedTime := IniRead(DataFile, "Timer", "DailyConsumedTime", 0)
        TriggerDate := IniRead(DataFile, "Timer", "TriggerDate", GetCurrentDate())
        if !RegExMatch(TriggerDate, "^\d{8}$") {
            TriggerDate := GetCurrentDate()
        }
        TimeUpTriggered := IniRead(DataFile, "Timer", "TimeUpTriggered", "false") = "true" ? true : false
    }
}

; Save timer data to INI file with TriggerDate validation
SaveTimerData() {
    global DailyConsumedTime, TriggerDate, TimeUpTriggered, DataFile
    if !RegExMatch(TriggerDate, "^\d{8}$") {
        TriggerDate := GetCurrentDate()
    }
    IniWrite(DailyConsumedTime, DataFile, "Timer", "DailyConsumedTime")
    IniWrite(TriggerDate, DataFile, "Timer", "TriggerDate")
    IniWrite(TimeUpTriggered ? "true" : "false", DataFile, "Timer", "TimeUpTriggered")
}

; Reset timer if it's a new day
ResetIfNewDay() {
    global DailyConsumedTime, TriggerDate, TimeUpTriggered
    CurrentDate := GetCurrentDate()
    if (CurrentDate != TriggerDate) {
        DailyConsumedTime := 0
        TimeUpTriggered := false
        TriggerDate := CurrentDate
        SaveTimerData() ; Save reset state
    }
}

; Keep script running
Persistent

```