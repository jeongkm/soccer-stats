# Soccer Stats



## 사전 작업

- ALM VM 준비 : https://github.com/jeongkm/alm-infra



## Ansible Playbook - Inventory 파일 업데이트



1. git fork

   - https://github.com/jeongkm/soccer-stats.git

2. git clone

   - git clone git@github.com:kmsandbox/soccer-stats.git

3. inventory.ini 변경

   - cd soccer-stats/provision

   - vi inventory.init

   - 대상 앱 서버의 FQDN과 IP 입력

      - [app_server]
         appserver-1 ansible_host=172.16.190.73 ansible_connection=ssh  ansible_port=22 ansible_user=jenkins

   - git push

      git status

      git add *

      git commit -m "Update inventory.init"

      git push

      



## Nexus Repository 생성



1. Nexus UI 접속 : http://nexus-1.alm.ibm.com:8081/
2. admin 으로 로그인

   - ID/Password : admin / admin123
3. 설정 > Repository > Create repository
4. Select Recipe > maven2 (hosted)

   - Name : ansible-meetup 입력

   - Strict Content Type Validation  체크 해제
5. 저장



## SonarQube API 토큰 확인



1. SonarQube UI 접속 : http://sonar-1.fyre.ibm.com:9000
2. admin 으로 로그인

   - ID/Password : admin / admin
3. 최초 로그인 시, token 생성 프롬프트에서 토큰 생성
   - 토큰 이름 : token for jenkins
   - 생성된 토큰 확인 - Jenkins Credentials에 등록
     - token for jenkins: **6cc030d4fcb21c73c3f8a6a4d49b980c5d0032fa**
4. 토큰 생성 메뉴 위치 확인 : Admin > My Account > Security > Tokens



## Jenkins 서버 Ansible 설정 변경



1. Jenkins 서버에 SSH 접속
2. Ansible의 호스트 키 체크 해제

```bash
vi /etc/ansible/ansible.conf

# 71라인 커멘트 제거
host_key_checking=False
```



## Jenkins 시스템 구성



### Global Tool Configuration



1. Jenkins 관리 > Global Tool Configuration
2. SonarQube Scanner 섹션 > Add SonarQube Scanner
   - Name : sonar
3. Maven 섹션 > Add Maven
   - Name : m3
4. Save



### Plugin Manager (아리까리..)

1. Jenkins 관리 > 플러그인 관리
2. 설치 확인 탭 > Maven 검색



### Manage Credentials

1. Jenkins 관리 > Manage Credentials
2. Jenkins Store 클릭 > Global credentials (unrestricted) 클릭
3. Add Credentials - SonarQube token
   - Kind : Secret text
   - Secret : 6cc030d4fcb21c73c3f8a6a4d49b980c5d0032fa
   - ID : sonarqube-token
4. Add Credentials - Nexus
   - Kind : Username with password
   - Username : admin
   - Password : admin123
   - ID : nexus
5. Add Credentials - App Server SSH private key for jenkins
   - Kind : SSH Username with private key
   - ID : ssh-jenkins
   - Username : jenkins
   - Private Key  > Enter directly > Add
     - https://github.com/jeongkm/alm-infra/blob/master/ansible/keys/jenkins 의 내용 입력



### System Settings



1. SonarQube servers 섹션 > SonarQube installations > Add SonarQube
   - Name : sonar
   - Server URL : http://sonar-1.alm.ibm.com:9000 
   - Server authentication token : sonarqube-token
2. 저장



##  파이프라인 구성



1. Jenkins UI 접속 : http://jenkins-1.alm.ibm.com:8080
2. admin 으로 로그인

   - ID/Password : admin / admin123
3. 새로운 Item > Pipeline 선택 > item name : alm-pipeline 입력 > OK 클릭
4. General > "이 빌드는 매개변수가 있습니다" 체크 > 매개변수 추가 > String Parameter > 아래 매개변수 모두 입력

   - FULL_BUILD : true
   - HOST_PROVISION : appserver-1
   - GIT_URL : https://github.com/jeongkm/soccer-stats.git
   - NEXUS_URL : nexus-1.alm.ibm.com:8081
5. Pipeline 섹션
   - Definition : Pipeline script from SCM
   - SCM : Git
   - Repositories : https://github.com/jeongkm/soccer-stats.git
6. 저장



## 파이프라인 실행



1. Jenkins pipeline : alm-pipeline 열기
2. Build with Parameters > 빌드하기
3. Console Output 확인 
4. Approval 단계에서 Proceed 선택
5. 앱 배포 완료 확인



## 앱 배포 결과 확인



URL : http://appserver-1.alm.ibm.com:8080/matches/juventus/milan



---
---

# 원본

Soccer Stats is an example application to be used as a proof of concept for a presentation at [Ansible Meetup in São Paulo](https://www.meetup.com/Ansible-Sao-Paulo/events/243212921/).

## Pre-requistes

* JDK 1.8
* Maven 3.3+

## Environment

It's a sample Rest API built upon Spring Rest Framework. The database is based on data gathered from 2015/2016 season of Italian Soccer National Championship.

During the Spring Context bootstrap a temporary database is created using H2 with data imported from a spreedsheet.

## Installation

Just run `mvn clean package` on the project directory and your ready to go.

## Using

Bring the application up by running `java -jar soccer-stats-X.X.X.jar`, where's `X.X.X` is the project's version.

After the startup the endpoint should be availble at `http://localhost:8080/matches/{team_name}` where `{team_name}` must be a Italian team name like `juventus`, `milan`, `udinese` and so on.

To bring a specific match, try the endpoint `http://localhost:8080/matches/{home_team_name}/{visitor_team_name}` replacing the param vars to the match you'd like to see, for example:

[http://localhost:8080/matches/juventus/milan](`http://localhost:8080/matches/juventus/milan`)

## Credits

[Football-Data](http://www.football-data.co.uk/) for providing the data used for this lab.


