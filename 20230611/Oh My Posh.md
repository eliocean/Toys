# [Oh My Posh](https://github.com/jandedobbeleer/oh-my-posh  ) shell 主题


[使用文档](https://ohmyposh.dev/)


## [Windows Terminal 下载，美化，完整配置](https://zhuanlan.zhihu.com/p/439437013)



## 1. 安装字体

[github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/FiraCode.zip](https://link.zhihu.com/?target=https%3A//github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/FiraCode.zip)

下载字体后，解压-全选-右键安装。

## 2. 安装 Powershell 插件 PSReadline

直接运行内部命令即可(需要管理员模式），运行时会让你授权，全部按A回车

```text
# 安装 PSReadline 包，该插件可以让命令行很好用，类似 zsh
Install-Module -Name PSReadLine  -Scope CurrentUser 
```

## 3. 安装 oh-my-posh 包，让你的命令行更酷炫、优雅
```text
winget install JanDeDobbeleer.OhMyPosh -s winget
```

## 4. 添加 Powershell 启动参数

powershell运行

```text
notepad.exe $Profile
或
code $Profile
```

上面的是系统自带的文档，下面的是vscode打开

```text
#-------------------------------  Set oh-my-posh theme BEGIN  -----------------------------
# 设置主题
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\kali.omp.json" | Invoke-Expression 

#-------------------------------  Set oh-my-posh theme END  -------------------------------

# 引入 ps-read-line
Import-Module PSReadLine

#------------------------------- Import Modules END   -------------------------------





#-------------------------------  Set Hot-keys BEGIN  -------------------------------
# 设置预测文本来源为历史记录
Set-PSReadLineOption -PredictionSource History

# 每次回溯输入历史，光标定位于输入内容末尾
Set-PSReadLineOption -HistorySearchCursorMovesToEnd

# 设置 Tab 为菜单补全和 Intellisense
Set-PSReadLineKeyHandler -Key "Tab" -Function MenuComplete

# 设置 Ctrl+d 为退出 PowerShell
Set-PSReadlineKeyHandler -Key "Ctrl+d" -Function ViExit

# 设置 Ctrl+z 为撤销
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo

# 设置向上键为后向搜索历史记录
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward

# 设置向下键为前向搜索历史纪录
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
#-------------------------------  Set Hot-keys END    -------------------------------





#-------------------------------    Functions BEGIN   -------------------------------
# Python 直接执行
$env:PATHEXT += ";.py"

#-------------------------------    Functions END     -------------------------------


```

将上述代码复制进去

## 5.修改主题

oh-my-posh提供了大量的主题，我们可以通过运行
```text
   Get-PoshThemes
```
也可以直接查看官方文档：https://ohmyposh.dev/docs/themes

查看都有哪些主题，然后打开配置文件

```text
notepad.exe $Profile
或
code $Profile
```

然后修改文件中的主题配置

```text
# 例如如果选定kali主题
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\kali.omp.json" | Invoke-Expression 
```

然后输入下面的命令，来让配置生效：
```text
. $PROFILE
```
