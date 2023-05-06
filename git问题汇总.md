# git遇到的各种问题汇总
## error: failed to push some refsto
    先git pull --rebase origin master
    然后上传 git push -u origin master
## 添加remote地址
    在本地运行 git remote -v，没有显示任何地址，需要添加，而不是修改。运行
    git remote add origin 远程url
## Please move or remove them before you switch branches
    git clean -dfx 干掉缓存区的文件
    强调一下,git clean是把你当前工作的分支所有东西保持和远程的分支一致，
    就是说你远程是怎么样的，执行这个命令后你本地的就是怎么样的，没有进入版本的文件会直接扔掉
