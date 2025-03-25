# Jenkins-CICD


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