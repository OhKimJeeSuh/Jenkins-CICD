# ♾️Jenkins-CICD 자동화 파이프라인 구축
<br>

### 📋 목차
1. [📝 개요](#1--개요)
2. [🎓 팀원소개](#2--팀원소개)
3. [🛠 기술 스택](#3--기술-스택)
4. [🔧 CI/CD를 위한 준비 사항](#4--cicd를-위한-준비-사항)
   - [🔗 Jenkins - GitHub 연동](#-1-jenkins---github-연동)
      - ngrok 설치 및 설정
      - Webhook 등록
      - tools 추가
      - jenkins-github 연동 확인
   - [🏗️ github jar 파일 build하기](#2-%EF%B8%8F-github--jar-파일-build하기)
5. [🗂️ docker Jenkins - 내부 Ubuntu bind-mount](#5-%EF%B8%8F-docker-jenkins---내부-ubuntu-bind-mount)
6. [🌐 docker Jenkins - 외부 Ubuntu 통신](#6--docker-jenkins---외부-ubuntu-통신)
7. [🔄 inotifywait를 이용하여 자동으로 jar 파일 실행하기](#7--inotifywait를-이용하여-자동으로-jar-파일-실행하기)
8. [💥 트러블슈팅](#-트러블-슈팅)

## 1. 📝 개요
<br>
이 프로젝트는 Jenkins를 활용하여 CI/CD 파이프라인을 구축하는 과정을 설명합니다. GitHub 코드 저장소와 Jenkins를 연동하여 소스 코드가 변경될 때마다 자동으로 빌드, 테스트 및 배포를 수행하는 자동화 시스템을 구현했습니다. Docker 컨테이너에서 실행되는 Jenkins와 Ubuntu 서버 간의 통신 및 파일 공유 방법, 그리고 inotifywait를 이용한 자동 배포 기능까지 포함됩니다.
<br> <br>

## 2. 🎓 팀원소개

|<img src="https://avatars.githubusercontent.com/u/114637614?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/165532198?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/193404366?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/79669001?v=4" width="150" height="150"/>|
|:-:|:-:|:-:|:-:|
|오현두<br/>[@HyunDooBoo](https://github.com/HyunDooBoo)|김소연<br/>[@ssoyeonni](https://github.com/ssoyeonni)|지수근<br/>[@SuGeunJee](https://github.com/SuGeunJee)|서소원<br/>[@PleaseErwin](https://github.com/PleaseErwin)|

<br><br>


## 3. 🛠 기술 스택

| 기술               | 도구                |
|--------------------|-------------------------|
| **버전 관리**    | ![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)         |
| **CI/CD 도구**    | ![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)     |
| **컨테이너** | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)             |
| **빌드 도구** | ![Gradle](https://img.shields.io/badge/Gradle-02303A?style=for-the-badge&logo=gradle&logoColor=white)            |
| **배포 환경** | ![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)            |
| **네트워크 터널링** | ![ngrok](https://img.shields.io/badge/ngrok-1F1E37?style=for-the-badge&logo=ngrok&logoColor=white)         |

<br><br>

## 4. 🔧 CI/CD를 위한 준비 사항
### 🔗 1. jenkins - github 연동
> **ngrok** : webhook시 GitHub가 Jenkins에 HTTP POST 요청을 보내야 하는데, 현재 jenkins가 private IP를 가지므로 ngrok 사용 해야한다.
> <br><br>
> **webhook** : Jenkins와 GitHub를 연동하여 코드 수정 시 Jenkins로 자동 빌드 흐름을 만들기 위해 webhook 설정이 필요하다.

<br>

### 1) ngrok
**1. ngrok 다운로드**
https://ngrok.com/

- Getting Started → Setup & Installation
- 사이트에 로그인하여 Windows용 다운로드<br>
![Image](https://github.com/user-attachments/assets/250a87a5-738a-453a-9b58-de4714c1b415)

<br>

- Getting Started → Your Authtoken
- 토큰 발급받기
- Command Line 복사
![Image](https://github.com/user-attachments/assets/c511e507-50ef-4567-97a4-de7835491dab)
<br>

**2. ngrok 실행 후 복사한 Command Line 붙여넣기<br>**
![Image](https://github.com/user-attachments/assets/38ee159a-99d6-46b1-97a0-e279a5fea123)


- C:\Users\사용자이름\AppData\Local\ngrok 폴더에 yml 파일 생성 확인
  ![Image](https://github.com/user-attachments/assets/8c7f0b66-e638-401b-9b0b-d1be4db57121)

<br>

- PowerShell 관리자 모드에서 .\ngrok http http://ip:Jenkins_port로 실행
```
.\ngrok http http://localhost:8080
```

- 노란 박스 부분 복사
![Image](https://github.com/user-attachments/assets/6ee49a52-2464-4a5e-9f94-628a14562fef)


### 2) webhook
**깃허브에 webhook 설정**
- 연동할 github Repository에서 Settings → Webhooks → Add webhook
![Image](https://github.com/user-attachments/assets/92f260eb-92e2-4ca5-a6a4-32cd4dd7f338)

- Payload URL에 ngrok에서 복사한 주소와 그 뒤에 /github-webhook/ 넣기
  ![Image](https://github.com/user-attachments/assets/67ebcf02-e631-4fac-9121-1334a7a5a6ca)
  ![Image](https://github.com/user-attachments/assets/a850a8c2-eb21-41c6-a387-5e5915b381e9)
  ![Image](https://github.com/user-attachments/assets/688bd3ca-5488-41fd-ba8b-c4a648c577d8)

<br>

### 3) tools 추가

<br>

### 4) jenkins-github 연동 확인
**1. jenkins 접속하여 Dashboard → 새로운 item**
   
**2. Pipeline 선택**
   ![Image](https://github.com/user-attachments/assets/a2a7119e-65ba-4baf-aca6-db03b611c145)

**3. Githubhook trigger for GITScm polling 체크**
   ![Image](https://github.com/user-attachments/assets/509aaaea-760c-48a6-badb-83fb49580f5f)

**4. script에 해당 webhook 설정한 깃허브 주소 넣기**
- 만약 branch가 main이면 branch: ‘main’ 명시해줘야함
- 제대로 pull 받아 왔는지 확인하기 위해 ls 추가
  ![Image](https://github.com/user-attachments/assets/0d00a027-a5c2-405f-9e13-d34d1c28da2a)
  ```
  pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                git branch: 'main', url: 'https://깃허브주소.git'

                sh "ls"
            }
         }  
      }
   }
  ```

**5. 현재 repository에 있는 파일들과 콘솔 출력ls가 일치하는 지 확인**
   ![Image](https://github.com/user-attachments/assets/966893ac-1077-4e9b-b7cb-e9f800f709dc)
   ```
   [Pipeline] sh
   + ls
   README.md
   step07_CICD
   [Pipeline] }
   [Pipeline] // stage
   [Pipeline] }
   [Pipeline] // node
   [Pipeline] End of Pipeline
   Finished: SUCCESS
   ```

**6. 깃허브 내용 수정 시 자동 빌드 되는 것 확인**
   ![Image](https://github.com/user-attachments/assets/4f1b4178-e232-4f87-8977-ba00bc44370e)

<br>
<br>

### 2. 🏗️ github  jar 파일 build하기
**1. 현재 깃허브에 올라간 파일 목록**
   ![Image](https://github.com/user-attachments/assets/03dc3e40-029d-44a5-aa74-85f7113fea55)
   - 위 파일 중 **gradlew**를 빌드해야 함
   
**2. Configure → Pipeline → Script 수정**
   ```
   pipeline {
      agent any

      stages {
          stage('Clone Repository') {
              steps {
                  git branch: 'main', url: 'https://github.com/ssoyeonni/20250324a.git'
              }
          }          
          stage('Build') {
              steps {
                  dir('./step07_CICD') {                   
                     sh 'chmod +x gradlew'    
					  		     sh './gradlew build'
                  }
              }
          }               
      }
      post {
          success {
              echo '빌드 성공'
          }
          failure {
              echo '빌드 실패'
          }
      }
   }

   ```

3. Workspace/step07_CICD/build/libs/ 에서 build 된 jar 파일 확인
   ![Image](https://github.com/user-attachments/assets/51f40088-535a-4320-b217-fc80fdffc523)


<br>
<br>
<br>

## 5. 🗂️ docker Jenkins - 내부 Ubuntu bind-mount
- git push가 이루어졌을 때 jenkins에서 자동으로 감지 후, build 하여 bind mount로 복사 -> 로컬 ubuntu에서 파일 확인 가능

### 1. 도커 실행 명령어
`$ docker run -d --name jenkins-new -v /home/ubuntu/jarappdir:/var/jenkins_home/jarappdir -p 8088:8080 jenkins-backup` <br><br>
ubuntu 서버의 /home/ubuntu/jarappdir 과 jenkins docker 서버의 /var/jenkins_home/jarappdir를 bind mount 한다.

### 2. Jenkins Script - bind mount된 폴더로 jar 파일 복사
```bash
pipeline {
    agent any
    
    stages {
        stage("Clone Repository"){
            steps {
                git branch: 'main', url: 'https://github.com/SuGeunJee/20250324a.git'
            }
        }
    
        stage('Build') {
            steps {
                
                dir('step07_cicd1') {
                    // gradlew에 실행 권한 부여
                    sh 'chmod +x gradlew'
                    
                    // gradlew로 빌드 실행
                    sh './gradlew clean build'
                    
                    sh 'ls -alh'
                    
                }
            }
        }
        
        stage('Deploy JAR') {
            steps {
                // JAR 파일 복사
                sh 'cp step07_cicd1/build/libs/step07_cicd1-0.0.1-SNAPSHOT.jar /var/jenkins_home/jarappdir/'
            }
        }
    }
}
```

### 3. 결과
Jenkins Docker에서 빌드 한 jar 파일, Ubuntu Host 공유 폴더로 공유 성공!! <br>

<img width="444" alt="image" src="https://github.com/user-attachments/assets/eb941f8d-5c37-41ac-9ed2-581c6943be3e" />

## 6. 🌐 docker Jenkins - 외부 Ubuntu 통신

### 1. docker Jenkins 서버 정보
- ip : 172.17.0.2
- id : jenkins

### 2. 외부 Ubuntu 서버 정보
- ip : 192.168.0.13
- id : ubuntu

### 3. SSH, SCP 통신을 위한 인증 설정
현재 아래와 같은 스크립트를 Jenkins에서 실행시키면, build 오류가 발생한다. <br>
그 이유는 ssh Public key가 공유되어 있지 않기 때문이다. <br>

`Host key verification failed.`

따라서, docker jenkins server에서 ssh-key를 생성한 후, public key를 외부 Ubuntu server에 공유를 해주면 된다! <br>

### 4. ssh key 생성
`ssh-keygen -t rsa -b 4096` <br> <br>
<img width="476" alt="image-6" src="https://github.com/user-attachments/assets/532ae3cb-5559-4408-984a-401e3225aac6" />


### 5. ssh public key를 공유
`ssh-copy-id <공유할 서버의 이름>@<공유할 서버의 ip>` <br> <br>
<img width="704" alt="image-5" src="https://github.com/user-attachments/assets/943476c5-1023-45dc-95d3-31b3b8112af9" />

### 6. 비밀번호를 물어보지 않고, SSH 성공 테스트
<img width="482" alt="image-7" src="https://github.com/user-attachments/assets/b93292e3-1f15-4817-8cf5-c667d1799de8" />

### 7. Jenkins Script - 외부 Ubuntu로 scp 통신
```bash
pipeline {
    agent any
    
    stages {
        stage("Clone Repository"){
            ...
            }
        }
    
        stage('Build') {
            steps {
                ...
            }
        }
        
        stage('Deploy JAR') {
            steps {
                ...
            }
        }
        stage('JAR COPY') {
            steps {
                sh 'scp /var/jenkins_home/jarappdir/step07_cicd1-0.0.1-SNAPSHOT.jar ubuntu@192.168.0.13:/home/ubuntu/jarappdir'
            }
        }
    }
    post {
        success {
            echo '✅ 빌드 및 배포 성공!'
        }
        failure {
            echo '❌ 빌드 또는 배포 실패! 오류 확인 필요!'
        }
    }
    
}
```
### 8. jenkins에서 빌드된 jar 파일 외부 ubuntu에 이관 성공!!
<br>
<img width="785" alt="image-8" src="https://github.com/user-attachments/assets/5326c60c-80f1-4500-8c80-81b49c4c69be" />

## 7. 🔄 inotifywait를 이용하여 자동으로 jar 파일 실행하기
### 1. inotifywait란?
- 리눅스 시스템에서 파일이나 디렉토리의 변경 사항을 모니터링하기 위한 간단한 명령줄 도구

이관해준 jarappdir 폴더를 감지하다가 변화가 생기면 현재 실행되고 있는 jar파일을 kill 하고, 새로운 jar 파일을 실행하는 스크립트 <br>
이관된 서버에서 실행!

```bash
#!/bin/bash
# 파일명: /home/ubuntu/jar_watcher.sh

# 필요한 패키지 설치 확인
if ! command -v inotifywait &> /dev/null; then
    echo "inotifywait not found. Installing inotify-tools..."
    sudo apt-get update
    sudo apt-get install -y inotify-tools
fi

# 감시할 디렉토리와 JAR 파일명 설정
WATCH_DIR="/home/ubuntu/jarappdir"
JAR_FILE="step07_cicd-0.0.1-SNAPSHOT.jar"
LOG_FILE="app.log"

echo "Starting JAR file watcher for $WATCH_DIR/$JAR_FILE"
echo "Logs$ will be written to $WATCH_DIR/$LOG_FILE"

# 무한 루프로 파일 변경 감시
while true; do
    # inotifywait로 파일 변경 감시
    inotifywait -e close_write,moved_to --format "%f" "$WATCH_DIR" | while read FILENAME
    do
        # 지정된 JAR 파일이 맞는지 확인
        if [ "$FILENAME" = "$JAR_FILE" ]; then
            echo "$(date): JAR file updated. Restarting application..."
            
            # 기존 프로세스 종료
            PID=$(pgrep -f "$JAR_FILE")
            if [ ! -z "$PID" ]; then
                echo "$(date): Stopping existing application (PID: $PID)..."
                kill $PID
                sleep 3
                
                # 프로세스가 종료되지 않았다면 강제 종료
                if ps -p $PID > /dev/null; then
                    echo "$(date): Force stopping application..."
                    kill -9 $PID
                fi
            else
                echo "$(date): No existing application found."
            fi
            
            # 새 애플리케이션 실행
            echo "$(date): Starting new application..."
            cd "$WATCH_DIR"
            nohup java -jar "$JAR_FILE" --server.port=8082 > "$LOG_FILE" 2>&1 &
            NEW_PID=$!
            echo "$(date): Application deployed with PID: $NEW_PID"
        fi
    done
done
```

### 2. 실행 결과
- 첫 번째 실행 결과 <br>
<img width="1104" alt="화면 캡처 2025-03-25 122228" src="https://github.com/user-attachments/assets/480a618b-87b9-4ee8-bf73-e200ea00bf3a" />

- git push로 인한 jenkins 감지 ci - cd 결과 <br>
<img width="1100" alt="화면 캡처 2025-03-25 122429" src="https://github.com/user-attachments/assets/a5dd85d3-7991-4233-bf1a-c4c0318a898e" />

## 3. 한계점
현재 이 스크립트는 기존의 jar를 kill 하고, 새로운 jar를 실행하는 시점에서 잠깐의 down time이 발생한다. <br>
즉, 무중단 배포는 아님!! <br>

<img width="785" alt="image-9" src="https://github.com/user-attachments/assets/77448545-7969-407c-a36e-afd4d40c6365" />


<br><br>

# 💥 트러블 슈팅

## Ⅰ. Jenkins 기반 CICD 서버에서 JAR 실행 오류 해결🚀

Jenkins를 사용하여 **CI/CD 서버**를 구축하고, 배포된 JAR 파일을 실행하는 과정에서 **서버가 실행되지 않는 문제**가 발생했다.

---

## ❌ 문제 발생

배포된 JAR 파일 실행 시 **서버가 정상적으로 실행되지 않음**

확인 결과, **`application.properties`의 `server.port=80`** 으로 설정되어 있었다.

## ❌ 오류 메시지

![image](https://github.com/user-attachments/assets/20cd2a8a-5d0b-41c8-b6ce-9547e8a6eb68)

### 📌 **원인 분석**

- **0~1023번 포트**는 **Well-Known Port**로, 루트 권한이 필요함
- 루트 권한 없이 실행하려다 보니 **Spring Boot가 정상적으로 실행되지 않음**
- 실행 중 다음과 같은 **오류 로그 출력됨**

---

## ✅ 해결 방법

1️⃣ **1024 이상의 포트 사용 추천**

- Well-Known Port를 사용하지 않도록 설정하는 것이 안전함
- `server.port=8080` 또는 `server.port=9090` 등의 **1024 이상 포트로 변경**

2️⃣-1️⃣ **JAR 실행 시 포트 지정**

- 아래와 같이 실행하면 정상 동작함 ⬇️
    
    
    ![image](https://github.com/user-attachments/assets/d2d52e76-8a54-4535-9cbf-16b2fcc8d2bb)
    ![image](https://github.com/user-attachments/assets/2cd57df2-2987-4d24-b24e-3cd01a5fede3)
    

2️⃣-2️⃣ [**application.properties](http://application.properties) 포트 수정**

![image](https://github.com/user-attachments/assets/2ccc90fd-45fd-4006-811a-7022f8f9a2b6)

---

## 🎯 결론

✅ **루트 권한 없이 0~1023번 포트 사용 불가**

✅ **1024번 이상의 포트로 변경 후 정상 실행됨**

✅ **Jenkins를 통한 CI/CD 배포 시 포트 설정 확인 필수!**

---

## Ⅱ. 권한 설정 오류🚀

기존에는 **CICD Linux 서버**에서 **Docker로 Jenkins**를 올려 사용했으나, **Linux 서버 자체에 Jenkins를 설치**하여 실행하도록 변경함.

---

## ❌ 문제 발생

기존 Docker에서 jenkins를 올려 썼을 때 사용하던 스크립트를 일부만 수정한 뒤 실행 했더니, **JAR 파일을 저장하는 폴더**(`/home/ubuntudoo/jarappdir/`)에서 **권한 오류** 발생.

## ❌ 오류 메시지

![image](https://github.com/user-attachments/assets/da98c231-db20-4977-b6bb-0ebd826f281c)

### 📌 **원인 분석**

- **Docker로 실행할 때**는 Jenkins가 컨테이너 내부에서 실행되어 기존의 Linux 사용자(`ubuntudoo`)와 충돌 없이 동작함.
- **Linux 서버 자체에 Jenkins를 설치**하면 **새로운 사용자(`jenkins`)가 생성됨**.
- Jenkins가 실행하는 **스크립트는 `jenkins` 사용자 권한에서 실행**되므로,
    
    기존에 JAR 파일을 저장하던 **`/home**/ubuntudoo/jarappdir/` **디렉토리에 대한 접근 권한이 없음**
    

---

## ✅ 해결 방법

1️⃣ **Jenkins와 ubuntu 사용자를 포함하는 사용자 그룹 생성**

```
sudo groupadd ubunkins
```

2️⃣ **`jenkins`와 `ubuntu` 사용자를 그룹에 추가**

```
sudo usermod -aG ubunkins jenkins
sudo usermod -aG ubunkins ubuntu
```

**3️⃣ JAR 파일이 저장될 디렉토리의 그룹 소유자 변경**

```
sudo chown -R ubuntu:ubunkins /home/ubuntu/cicd
```

**4️⃣ 그룹에 쓰기 권한 부여**

```
sudo chmod -R 775 /home/ubuntu/cicd
```

**5️⃣ Jenkins에서 그룹 권한이 적용되도록 다시 로그인 또는 재부팅**

```
sudo systemctl restart jenkins
```

---

## 🎯 최종 결과

✅ **Jenkins가 `scp` 명령어를 사용하여 정상적으로 JAR 파일을 `/home/ubuntu/cicd/`에 저장할 수 있음!** 🚀

---

## Ⅲ. SSH Key 인증 오류 🚀

### ❌ 문제 발생

```
+ scp /var/lib/jenkins/workspace/auto-pipe01/step07_cicd/build/libs/step07_cicd-0.0.1-SNAPSHOT.jar ubuntu@192.168.0.21:/home/ubuntu/jarappdir
Host key verification failed.
scp: Connection closed
```

Jenkins에서 빌드된 JAR 파일을 scp 명령어를 이용해 **192.168.0.21 서버**에 전송하려 했으나, **Host key verification failed** 오류 발생

---

## ❌ 오류 메시지

```
+ scp -i /home/ubuntu/.ssh/id_rsa /var/lib/jenkins/workspace/auto-pipe01/step07_cicd/build/libs/step07_cicd-0.0.1-SNAPSHOT.jar ubuntu@192.168.0.21:/home/ubuntu/jarappdir
Warning: Identity file /home/ubuntu/.ssh/id_rsa not accessible: Permission denied.
Host key verification failed.
scp: Connection closed
```

`/home/ubuntu/.ssh/id_rsa` private key를 ssh 인증에 사용하도록 지정했으나 **Host key verification failed** 오류와 **Permission denied** 오류 발생
<br><br>

![usermod_jenkins](https://github.com/user-attachments/assets/83e17a07-504f-4db0-8062-3aa1d0fbcfe0)

```
chmod 777 .ssh
chmod 777 id_rsa
```

`id_rsa` 파일의 소유자는 `ubuntu`이다. `id_rsa` 파일에 `jenkins` 유저가 접근할 수 있도록 `ubuntu` 그룹에 유저 `jenkins`를 추가하고 `.ssh` 폴더와 `id_rsa` 파일의 권한을 변경했지만 여전히 같은 오류가 발생했다.

---

## ✅ 해결 방법


1️⃣ Jenkins에서 CI/CD 자동화로 실행할 때 유저는 `jenkins`로 동작하므로, `/home/ubuntu/.ssh` 하위의 파일이 아닌 `/var/lib/jenkins/.ssh` 하위의 파일을 사용해 ssh 접속을 시도한다.
<br><br>
![jenkins_key](https://github.com/user-attachments/assets/de07aa6d-31e2-49fd-ba3d-dff28611ab3a)
<br><br>
2️⃣ `jenkins` 유저로 로그인한 후 ssh key를 생성하면 `/var/lib/jenkins/.ssh` 폴더에 소유자가 `jenkins`인 public key와 private key가 생성된다.
<br><br>
![jenkins_copykey](https://github.com/user-attachments/assets/6371245d-2505-43ad-8fb1-f14d4d884ca0)
<br><br>
3️⃣ 192.168.0.21 `ubuntu` 서버의 authorized_keys 파일에 public key를 등록한 이후로는 비밀번호 없이 ssh 접속이 가능하다.
<br>

---

## 🎯 최종 결과

✅ **Jenkins가 `scp` 명령어를 사용하여 정상적으로 JAR 파일을 원격 ubuntu 서버의  `/home/ubuntu/jarappdir/` 에 저장할 수 있음!** 🚀




