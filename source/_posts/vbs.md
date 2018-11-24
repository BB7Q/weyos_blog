---
title: 使用vbs配置环境变量
date: 2018-11-24
categories:
- 开发工具
tags:
- 环境变量
- vbs
---

# 使用vbs自动化添加环境变量

设置用户变量，如需设置系统变量在`Environment`内填写`System`(需管理员权限)

```vbs
Set pSysEnv = CreateObject("WScript.Shell").Environment("User")

Function IsMatch(Str, Patrn)
  Set r = new RegExp
  r.Pattern = Patrn
  IsMatch = r.test(Str)
End Function

Sub SetEnv(pPath, pValue)
    Dim ExistValueOfPath
    IF pValue <> "" Then
     ExistValueOfPath = pSysEnv(pPath)
 IF Right(pValue, 1) = "\" Then pValue = Left(pValue, Len(pValue)-1)
 If IsMatch(ExistValueOfPath, "\*?" & Replace(pValue, "\", "\\") & "\\?(\b|;)") Then Exit Sub '已经存在该环境变量设置
 If ExistValueOfPath <> "" Then pValue = ";" & pValue
 pSysEnv(pPath) = ExistValueOfPath & pValue 
    Else
 pSysEnv.Remove(pPath)
    End IF
End Sub

Dim UserPath
UserPath = pSysEnv.Item("Path")
ZIP_HOME = "C:\Program Files\7-Zip"

IF Not IsMatch(UserPath, "%7Z_HOME%") Then
   SetEnv "Path", ZIP_HOME
End If

MsgBox "Set environment variable for 7z successfully."
```