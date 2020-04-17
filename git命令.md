git push <远程主机名> <本地分支名>:<远程分支名>


## 如果push遇到在输入密码是输错后，就会报这个错误fatal: Authentication failed for
>git config --system --unset credential.helper
## [存储凭证](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%87%AD%E8%AF%81%E5%AD%98%E5%82%A8)
>git config --global credential.helper store