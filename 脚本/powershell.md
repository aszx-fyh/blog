### [powershell变量](https://docs.microsoft.com/zh-cn/powershell/scripting/learn/ps101/using-variables-to-store-objects?view=powershell-7)
> set-executionpolicy 修改执行策略

>变量以$开头 ,但是变量名不包括$,类似PHP
>变量驱动器

```powershell
#查看操作变量的命令
Get-Command -Noun Variable | Format-Table -Property Name,Definition -AutoSize -Wrap

#显示使用变量驱动器的所有 PowerShell 变量
Get-ChildItem variable:

# PowerShell 可以使用任何 Windows 进程可用的相同环境变量,通过env: 的驱动器公开
Get-ChildItem env:
```