### [powershell 变量](https://docs.microsoft.com/zh-cn/powershell/scripting/learn/ps101/using-variables-to-store-objects?view=powershell-7)

> set-executionpolicy 修改执行策略

> 变量以$开头 ,但是变量名不包括$,类似 PHP。\*-variable cmdlet 操作的变量不是环境变量。用\$+变量名引用变量或给变量赋值

> 驱动器类似命名空间。 Powershell 中所有不是我们自己的定义的变量都属于驱动器变量（比如环境变量），它的前缀只是提供给我们一个可以访问信息的虚拟驱动器.。例如 env:windir，象 env：驱动器上的一个”文件”，我们通过\$访问它，就会返回”文件”的内容。[参考](https://www.pstips.net/powershell-drive-variables.html)

```powershell
#查看操作变量的命令
Get-Command -Noun Variable | Format-Table -Property Name,Definition -AutoSize -Wrap

Set-Variable
Get-Variable
Remove-Variable
#显示使用变量驱动器的所有 PowerShell 变量
Get-ChildItem variable:
#等价于
Get-Variable

# PowerShell 可以使用任何 Windows 进程可用的相同环境变量,通过env: 的驱动器公开
Get-ChildItem env:

#引用 或设置环境变量
$env:abc

#设置变量并启动node debug
${env:NODE_ENV}='local'; & 'C:\Program Files\nodejs\npm.cmd' 'run' 'debug' '--' '--inspect-brk'
```
