## svn仓库转git保留commit信息

* 依赖的工具包  
```bash  
    yum install git-svn
```  
* 获取SVN历史提交用户信息，创建SVN用户与GIT用户关系映射文件(提交过的用户都需要获取，否则任务会被中断 )  
```bash    
    #获取历史提交的作者信息
    svn log ^/ --xml | grep -P "^<author" | sort -u | perl -pe 's/<author>(.*?)<\/author>/$1 = /'
    svnuser = gituser <mail>  
    svnuser1 = gituser1 <mail>  
```  
* 拉取svn仓库    
```bash   
    拉取所有的trunk、branch和tags
    git svn clone --stdlayout --authors-file=authors.txt --no-metadata svn://172.16.16.4/wxapi.soyoung.com wxapi.soyoung.com  
    仅拉取trunk和branck  
    git svn clone --trunk=trunk --branches=branches --authors-file=authors.txt --no-metadata svn://172.16.16.4/wxapi.soyoung.com wxapi.soyoung.com  
```   
* 分支处理  
```bash  
    git branch -r
    git branch -d <branchname> # 删除不需要的分支
```  
* 添加和push git仓库  
```bash  
    #处理trunk，提交至git的master
    cd projectname  
    git remote add origin git@xxx.xxx.xxx.xxx:root/projectname.git  
    git push -u origin master  
    #切换到分支，并提交到git的分支中  
    git checkout remotes/spuwxapi.soyoung.com  
    git branch -a  
    git checkout -b spuwxapi  
    git push --all  
```  
* 遇到的报错  
```bash
    Use of uninitialized value $u in substitution (s///) at /usr/share/perl5/vendor_perl/Git/SVN.pm line 106.
    Use of uninitialized value $u in concatenation (.) or string at /usr/share/perl5/vendor_perl/Git/SVN.pm line 106.
    #解决方案，修改SVN.pm line 106.
    $u =~ s!^\Q$url\E(/|$)!! or die
            "$refname: '$url' not found in '$u'\n";
    修改为：
    if(!$u) {
            $u = $pathname;
    }else {
            $u =~ s!^\Q$url\E(/|$)!! or die
            "$refname: '$url' not found in '$u'\n";
    }

    #第二个报错，产生原因为在trunk第一次创建时，trunk是一个链接文件，而不是一个不同文件夹，此时工具无法从r1开始工作，手动将r4指向HEAD后，working
    fatal: Not a valid object name
    ls-tree -z ./: command returned error: 128
    Passing the "-r4:HEAD" parameter to "git svn clone" should work. It
    looks like the repository was initially miscreated and "trunk" was a
    symlink (and not a directory) in r1.
```
