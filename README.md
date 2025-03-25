# Jenkins-CICD
<br>

## 개요
<br>

## CI/CD를 위한 준비 사항
### 1. jenkins - github 연동
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

### 2. tools 추가

<br>

### 3. jenkins-github 연동 확인
**1. jenkins 접속하여 Dashboard → 새로운 item** <br>
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




## docker Jenkins - Local Ubuntu bind-mount
- ubuntu server /home/ubuntu/jarappdir - jenkins docker server /var/jenkins_home/jarappdir bind mount
- git push가 이루어졌을 때 jenkins에서 자동으로 감지 후, build 하여 bind mount로 복사 -> 로컬 ubuntu에서 파일 확인 가능

### 도커 실행 명령어
`$ docker run -d --name jenkins-new -v /home/ubuntu/jarappdir:/var/jenkins_home/jarappdir -p 8088:8080 jenkins-backup`

### Jenkins Script
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

### 결과
Jenkins Docker에서 빌드 한 jar 파일, Ubuntu Host 공유 폴더로 공유 성공!!

![alt text](image-3.png)

## docker Jenkins - 외부 Ubuntu 통신

### docker Jenkins 서버 정보
- ip : 172.17.0.2
- id : jenkins

### 외부 Ubuntu 서버 정보
- ip : 192.168.0.13
- id : ubuntu

### SSH, SCP 통신을 위한 인증 설정
현재 아래와 같은 스크립트를 Jenkins에서 실행시키면, build 오류가 발생한다.
그 이유는 ssh Public key가 공유되어 있지 않기 때문이다.

`Host key verification failed.`

따라서, docker jenkins server에서 ssh-key를 생성한 후, public key를 외부 Ubuntu server에 공유를 해주면 된다!

### ssh key 생성
`ssh-keygen -t rsa -b 4096`
![alt text](image-6.png)

### ssh public key를 공유
`ssh-copy-id <공유할 서버의 이름>@<공유할 서버의 ip>`
![alt text](image-5.png)

### 비밀번호를 물어보지 않고, SSH 성공 테스트
![alt text](image-7.png)

### Jenkins Script 실행
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
### jenkins에서 빌드된 jar 파일 외부 ubuntu에 이관 성공!!
![alt text](image-8.png)

## 이관 작업 후, inotifywait를 이용하여 자동으로 jar 파일 실행하기
### inotifywait란?
- 리눅스 시스템에서 파일이나 디렉토리의 변경 사항을 모니터링하기 위한 간단한 명령줄 도구

이관해준 jarappdir 폴더를 감지하다가 변화가 생기면 현재 실행되고 있는 jar파일을 kill 하고, 새로운 jar 파일을 실행하는 스크립트
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

### 실행 결과
- 첫 번째 실행 결과
![alt text](<화면 캡처 2025-03-25 122228.png>)

- git push로 인한 jenkins 감지 ci - cd 결과
![alt text](<화면 캡처 2025-03-25 122429.png>)

### 한계점
현재 이 스크립트는 기존의 jar를 kill 하고, 새로운 jar를 실행하는 시점에서 잠깐의 down time이 발생한다.

즉, 무중단 배포는 아님!!

![alt text](image-9.png)
