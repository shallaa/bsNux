bsNux
=====




## 서버환경설정

#### GIT 설치 및 설정 

```
# yum install git-core 
# git config --global user.name 'jidolstar'
# git config --global user.email 'jidolstar@cookilab.com'
# git config --global color.ui "auto"
# git config --list
```

#### SSH Keys 만들기 : https://help.github.com/articles/generating-ssh-keys 
```
# ssh-keygen -t rsa -C "jidolstar@cookilab.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
....

# ls (생성된 id_rsa, id_rsa.pub 확인)
authorized_keys  id_rsa  id_rsa.pub  known_hosts

# cat id_rsa.pub (공유키 확인)
ssh-rsa ....+tMy4VjOFQ== jidolstar@cookilab.com

# clip < ~/.ssh/id_rsa.pub (클립보드에 복사)
```

위 키 깃헙에 등록! https://github.com/settings/ssh 에 가서 Add SSH key 하자! 
그리고 아래처럼 잘 접속되는지 확인 

```
# ssh -T git@github.com 
Warning: Permanently added the RSA host key for IP address '192.30.252.131' to the list of known hosts.
Hi jidolstar! You've successfully authenticated, but GitHub does not provide shell access.
```

#### GITHUB로부터 crontab을 이용해 정기적으로 PULL처리 

##### Clone하기
```
# git clone git@github.com:projectBS/bsShortURL.git /var/django/ligo/test
Initialized empty Git repository in /var/django/ligo/test/.git/
remote: Reusing existing pack: 66, done.
remote: Total 66 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (66/66), 99.73 KiB | 85 KiB/s, done.
Resolving deltas: 100% (10/10), done.

# ls -al
total 32
drwxr-xr-x 6 root root 4096 May  8 20:17 .
drwxr-xr-x 4 root root 4096 May  8 19:47 ..
drwxr-xr-x 2 root root 4096 May  8 20:17 chromeextention
drwxr-xr-x 4 root root 4096 May  8 20:17 gae
drwxr-xr-x 8 root root 4096 May  8 20:18 .git
drwxr-xr-x 2 root root 4096 May  8 20:17 icons
-rw-r--r-- 1 root root 3402 May  8 20:17 LICENSE
-rw-r--r-- 1 root root 1733 May  8 20:17 README.md
```

##### GIT의 sparse checkouts 기능을 사용해 서브 디렉토리만 checkout하도록 처리 
참고 : http://blog.quilitz.de/2010/03/checkout-sub-directories-in-git-sparse-checkouts/ 

```
#echo "gae/" >> .git/info/sparse-checkout
#cat .git/info/sparse-checkout
gae/
# git read-tree -m -u HEAD
# ls -al
total 16
drwxr-xr-x 4 root root 4096 May  8 20:22 .
drwxr-xr-x 4 root root 4096 May  8 19:47 ..
drwxr-xr-x 4 root root 4096 May  8 20:17 gae
drwxr-xr-x 8 root root 4096 May  8 20:22 .git

# git pull origin gh-pages
Already up-to-date.
```

##### Pull처리할 Shell Script 작성

```
#vi ~/git-pull.sh
#!/bin/bash
echo "[`date +%Y-%m-%d_%H:%M:%S`]"
cd /var/ligo/bsShortURL
tmp=`git pull origin gh-pages`
if [ "$tmp" = "Already up-to-date." ];
then
        echo up-to-date
else
        killall python
        /var/ligo/bsShortURL/django/manage.py runfcgi host=127.0.0.1 port=8088
        echo update and server restart
fi
echo "--------------------------------------------"
```

##### 작성한 Shell Script가 Crontab을 이용해 주기적으로 실행되도록 함 

* 생성 
```
# crontab -e 
*/1 * * * * sh ~/git-pull.sh >> ~/git-pull.log
```

* [참고1] 생성된것 보기 
```
# crontab -l
*/1 * * * * sh ~/git-pull.sh >> ~/git-pull.log
```

* [참고2] crond 시작, 종료, 강제종료  
```
# /etc/rc.d/init.d/crond start{restart | stop}
# kill -9 pid 또는 killall crond
```

* [참고4] 실행중 확인 
```
# tail -f git-pull.log
[2014-05-08 22:19:01]
Already up-to-date.
-----------------------------------
[2014-05-08 22:20:01]
Already up-to-date.
-----------------------------------
[2014-05-08 22:21:01]
Already up-to-date.
-----------------------------------
[2014-05-08 22:22:01]
Already up-to-date.
-----------------------------------
```


#### SSH 자동으로 연결끊어짐 주기를 늘림
다음처럼 300초에서 3000으로 늘림. 다음 로그인후 적용!
```
# vi /etc/bashrc
TMOUT=3000
```

다음 로그인 후 확인
```
# echo $TMOUT
3000
```

----------
Copyrightⓒ 2013, ProjectBS Committee. All rights reserved.
