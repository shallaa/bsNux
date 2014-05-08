bsNux
=====



## SSH 자동으로 연결끊어짐 주기를 늘림

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
