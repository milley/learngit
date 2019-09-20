# adb基本操作

## 1. adb devices

列举当前连接的调试设备

## 2. adb logcat

打印log信息

| 指令 | 说明 | 备注 |
|---|-------|-------|
|adb logcat | 打印log | / |
| adb logcat | 打印log | / |
| adb logcat -c | 清除手机的log buffer | 有些手机权限控制, 不支持. |
| adb logcat -b <buffer> | 打印指定buffer的log信息 | buffer有: main(主log区,默认), events(事件相关的log), radio(射频, telephony相关的log) |
| adb logcat -v <format> | 格式化输出log | 常用的用adb logcat -v time显示时间 |
| adb logcat -f <filename> | 输出log到指定文件 |

## 3. adb install/uninstall

安装卸载apk

## 4. adb pull/push

调试设备和开发PC之间拷贝文件.

## 5. adb start/kill-server

启动/杀死adb简介中提到的Server端进程

## 6. adb shell

进入调试设备的shell界面, 此时可以使用调试设备中的很多指令

## 7. adb connect/disconnect

通过wifi进行远程连接手机进行调试的

## 8. adb shell am

am即activity manager.该命令用来执行一些系统动作, 例如启动指定activity, 结束进程, 发送广播, 更改屏幕属性等. 调试利器.

## 9. adb shell pm

pm即package manager.用来执行package相关的操作, 例如安装卸载, 查询系统的安装包等

## 10. adb shell screencap

截屏, 比截屏快捷键更加方便快捷

## 11. adb shell screenrecord

录屏, 做demo的话, 可以很方便的用这个命名录制视频, 然后借助工具将其转换成gif图

## 12. adb shell dumpsys

强大的dump工具, 可以输出很多系统信息. 例如window, activity, task/back stack信息, wifi信息等.

## 13. adb forward tcp:1322 tcp:1322

交互流程，pc端口重定向到手机端口
