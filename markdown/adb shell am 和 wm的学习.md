###adb shell am 和 wm的学习 

使用此命令可以从cmd控制台启动 activity, services；发送 broadcast等等：
```
C:\Users\Administrator>adb shell am
usage: am [subcommand] [options]

    start an Activity: am start [-D] [-W] <INTENT>
        -D: enable debugging
        -W: wait for launch to complete

    start a Service: am startservice <INTENT>

    send a broadcast Intent: am broadcast <INTENT>

    start an Instrumentation: am instrument [flags] <COMPONENT>
        -r: print raw results (otherwise decode REPORT_KEY_STREAMRESULT)
        -e <NAME> <VALUE>: set argument <NAME> to <VALUE>
        -p <FILE>: write profiling data to <FILE>
        -w: wait for instrumentation to finish before returning

    start profiling: am profile <PROCESS> start <FILE>
    stop profiling: am profile <PROCESS> stop

    start monitoring: am monitor [--gdb <port>]
        --gdb: start gdbserv on the given port at crash/ANR

    <INTENT> specifications include these flags:
        [-a <ACTION>] [-d <DATA_URI>] [-t <MIME_TYPE>]
        [-c <CATEGORY> [-c <CATEGORY>] ...]
        [-e|--es <EXTRA_KEY> <EXTRA_STRING_VALUE> ...]
        [--esn <EXTRA_KEY> ...]
        [--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE> ...]
        [-e|--ei <EXTRA_KEY> <EXTRA_INT_VALUE> ...]
        [-n <COMPONENT>] [-f <FLAGS>]
        [--grant-read-uri-permission] [--grant-write-uri-permission]
        [--debug-log-resolution]
        [--activity-brought-to-front] [--activity-clear-top]
        [--activity-clear-when-task-reset] [--activity-exclude-from-recents]
        [--activity-launched-from-history] [--activity-multiple-task]
        [--activity-no-animation] [--activity-no-history]
        [--activity-no-user-action] [--activity-previous-is-top]
        [--activity-reorder-to-front] [--activity-reset-task-if-needed]
        [--activity-single-top]
        [--receiver-registered-only] [--receiver-replace-pending]
        [<URI>]

```
使用实例：
**如启动一个 Activity：**
格式：
`adb shell am start -n 包名/包名＋类名（-n 类名,-a action,-d date,-m MIME-TYPE,-c category,-e扩展数据)`
 
- 实例1：
```
C:\Users\Administrator>adb shell am start -n com.android.camera/.Camera
Starting: Intent { cmp=com.android.camera/.Camera }
```
- 实例2：（带extra 的 intent）
```
C:\Users\Administrator>adb shell am start -n com.android.camera/.Camera -e abc hello
Starting: Intent { cmp=com.android.camera/.Camera (has extras) }
其中 extra 的 key 为 abc ,value 为字串 "hello"
```
**还可以发送命令模拟手机低电环境：**
- 实例：
```
adb shell am broadcast -a android.intent.action.BATTERY_CHANGED --ei "level" 3 --ei "scale" 100
```


####wm
adb shell wm size 1200x720

adb shell wm size 800x480
adb shell wm density 240
adb shell wm overscan reset

djh S2 模拟S1 机器
```
root@H2000:/ # wm density 160                                                  
root@H2000:/ # wm size 768x1024
```