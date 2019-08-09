---
layout: post
title: "MariaDB root 계정 패스워드 수정"
tags: [MariaDB, Linux]
categories: [MariaDB]
subtitle: "Column 'Password' is not updatable"
#feature-img: "md/img/thumbnail/why-used-jpa.png"
#thumbnail: "md/img/thumbnail/why-used-jpa.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# root 계정의 패스워드 설정에서 난감했던 상황, Column 'Password' is not updatable

---

### 들어가기전

프로젝트의 환경 셋팅을 하던 중 만났던 난감한 상황을 기록하려 합니다.

### 1. 상황

저는 로컬에서 HomeBrew를 사용하여 MariaDB를 설치했습니다.

1. MariaDB를 로컬에 설치
> HomeBrew(Mac macOS용 패키지 관리 Tool)를 사용하여 설치한다.

이때 root 계정의 패스워드를 수정해야 하는 상황 때문에, 예를 들어 다음 리눅스 명령어를 통해 수정하려 했습니다.

``` terminal
MariaDB [mysql]> update USER set PASSWORD=PASSWORD(‘root’) where USER=‘root’;
```

하지만 `Password` 칼럼을 업데이트할 수 없다는 결과를 얻을 수 있었습니다.

``` terminal
ERROR 1348 (HY000): Column ‘Password’ is not updatable
```

### 2. 분석

_권한 문제?_

**Column ‘Password’ is not updatable** 라는 결과 때문에 권한 문제인 줄 알았습니다.

따라서 이와 연관된 자료를 찾아보았지만 문제를 해결하지 못했습니다.

심지어 MariaDB를 설치하는 과정에서 놓친 부분이 있었던건 아닌지 의심가지고 새로 설치까지 했지만, 결과는 마찬가지였습니다.

### 3. 해결방법

해결방법으론 다음과 같이 MySQL 쿼리로 해결할 수 있습니다.

``` terminal
MariaDB [mysql]> set PASSWORD for ‘root’@’localhost' = PASSWORD(‘root’);
MariaDB [mysql]> FLUSH PRIVILEGES;
``` 

기존의 생각했던 update 쿼리와 달리, 루트 계정의 패스워드를 수정하기 위해선 `set`으로 시작하는 쿼리를 사용해야됩니다.

#### 3.1. 루트 패스워드 수정 쿼리

MySQL 5.6 이하

``` terminal
update USER set PASSWORD=PASSWORD(‘바꿀 비밀번호’) where USER=‘계정’;
```

MySQL 5.7 이상

``` terminal
set PASSWORD for ‘계정’@’host정보' = PASSWORD(‘바꿀 비밀번호’);
```

이처럼 본인의 MySQL의 버전에 따라 수행할 수 있는 쿼리가 다르다는걸 알 수 있었습니다.

#### 3.2. MariaDB 버전 확인 방법

``` terminal
$ mysql --version
``` 