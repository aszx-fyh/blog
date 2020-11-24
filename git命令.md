git push <远程主机名> <本地分支名>:<远程分支名>

## 如果 push 遇到在输入密码是输错后，就会报这个错误 fatal: Authentication failed for

> git config --system --unset credential.helper

## [存储凭证](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%87%AD%E8%AF%81%E5%AD%98%E5%82%A8)

> git config --global credential.helper store

-   修改分支名称
    > git branch -m 新分支名/git branch -m 旧分支名 新分支名
-   查看分支信息
    > git branch -v/-vv
    > git remote show origin
    > cat .git/config

*   [为什么.git/objects/pack 文件夹变得非常大，处理 git 仓库臃肿](https://www.jianshu.com/p/4f2ccb48da77)
    > git-repack - 在存储库中打包解包的对象
*   清理.git/objects/pack 文件
    > 首先找出 git 中前五大的文件:git verify-pack -v .git/objects/pack/pack-\*.idx | sort -k 3 -g | tail -5
    > 第一行的字母其实相当于文件的 id,用以下命令可以找出 id 对应的文件名：git rev-list --objects --all | grep 8f10eff91bb6aa2de1f5d096ee2e1687b0eab007
    > 删除:git filter-branch --force --index-filter "git rm --cached --ignore-unmatch -r 要删除的文件" --prune-empty --tag-name-filter cat -- --all
