---
layout: post
title: "github permission error"
date: 2021-04-19 18:05:30 -0400
categories: tips
use_math: true
---

git pull 할 때 업데이트된 문서가 권한이 root로 되어있을 때 해결법

SSH키를 추가했는데도 두 장소에서 git pull 받을 때마다 바뀐 파일의 권한이 root로 변경된다. 

해당 폴더에서

```
sudo chown -R username:usergroup *
```

그럼 권한이 다 유저로 바뀌어 수정하고 저장할 수 있게 된다.

