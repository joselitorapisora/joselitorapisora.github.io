---
title: Screen Timer
date: 2025-05-03
layout: post
categories: [post]
---

When it comes to Windows automation, scripting, and quick GUI development, I have never encountered anything quite like AutoHotKey (AHK). It is a unique and powerful tool that can do really cool stuff in Windows, such as key bindings, simulating keyboard and mouse actions, controlling applications, generating complete GUIs, modifying the registry, manipulating files, and even playing sound! All of this is achieved through a simple and well-documented scripting language.  

It has served me numerous times in the past, particularly when I needed a GUI for a time/activity logger, and various production and engineering utility tools for work.  

Check out their website to get a feel for what this tool can do.  
<https://www.autohotkey.com/> 

![Image 1](/assets/images/2025-05-03/s1.png)  

<br>
So, the wife has been nagging me to do something about our son's screen time! He has Google parental control on his phone to limit playtime on certain apps, but nothing like that for our home PC. He plays Roblox exclusively... for hours!  

I explored various parental control tools and even considered a controllable smart plug (at least for the PC monitor). Then, the idea of a custom timer using AHK came to mind. Being an avid user of Grok, after dinner, I described what I wanted the tool would do, and before going to bed, I already had a working script that does exactly what I wanted! Impressive! This is how the future of programming will look like—just describe what you want, and the bot will program it for you. What an exciting time, or scary time, to be alive!  

So, anyway, here’s how it works: The script monitors whenever the Roblox application is launched, triggering a check on the remaining time allocated for the day (set to 2.5 hours) and counting down from there. The timer resets at midnight or if the PC is rebooted (hopefully, my son won’t notice this behavior!). Once the timer runs out, an always-on-top, unmovable window with a friendly message will pop up! This window remains at the center of the screen whenever Roblox is launched, persisting until midnight.  

I used the AHK++ plugin in VS Code, providing IntelliSense and syntax highlighting...  

![Image 2](/assets/images/2025-05-03/s2.png)  

<br>
This is what the countdown timer looks like...  

![Image 3](/assets/images/2025-05-03/s3.png)  

<br>
And here is the time’s-up notification window—not too annoying, but certainly renders the app unplayable, haha...  

![Image 4](/assets/images/2025-05-03/s4.png)  


<br>
I’ve included the full source code here, in case anyone is interested.  
 

```
#Requires AutoHotkey v2.0
#SingleInstance Force
; Roblox Countdown Timer
; Prompt by JRapisora, coded by Grok 3

; Set timer duration in hours
global TimerDuration := 2.5 ; in hours
global TimeUpTriggered := false ; Tracks if timer has expired
global TriggerDate := "" ; Stores date when timer expires
global DailyConsumedTime := 0 ; Tracks total Roblox time today (seconds)
global ProgressGui ; GUI instance

; Monitor for Roblox
Loop {
    WinWait "ahk_exe RobloxPlayerBeta.exe"
    
    ; Check date and reset if new day
    CurrentDate := FormatTime(A_Now, "YYYYMMDD")
    if (CurrentDate != TriggerDate) {
        TimeUpTriggered := false
        DailyConsumedTime := 0
        TriggerDate := CurrentDate
    }
    
    ; Calculate remaining time
    TotalTime := TimerDuration * 3600 - DailyConsumedTime
    if (TotalTime <= 0 || (TimeUpTriggered && CurrentDate = TriggerDate)) {
        ShowTimeUpGui()
        WinWaitClose "ahk_exe RobloxPlayerBeta.exe"
        CleanupGui()
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
    ;TrayTip "Roblox closed! Now go out an touch grass =) Love -Papa", "Done for today.", 1
}

; Show "Time is up!" GUI centered
ShowTimeUpGui() {
    global ProgressGui, TimerDuration
    ProgressGui := Gui("+AlwaysOnTop -Caption +ToolWindow", "Timer")
    ProgressGui.SetFont("s16 Bold")
    ProgressGui.Add("Text","y70 Center w600", "Done for today! Now go out and touch grass =)")
    ProgressGui.Add("Text","Center w600", "-Love, Papa")    
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
    
    if (TotalTime <= 0) {
        CleanupGui()
        ShowTimeUpGui()
        TimeUpTriggered := true
        TriggerDate := FormatTime(A_Now, "YYYYMMDD")
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

; Keep script running
Persistent

```