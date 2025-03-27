
<img src="https://capsule-render.vercel.app/api?type=waving&color=ADEDFF&height=300&section=header&text=🐳Docker-Jenkins🐳&fontSize=55&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />


<br>

## 📍 Outline
- [1️⃣ Contributors](#1%EF%B8%8F⃣-contributors)
- [2️⃣ Purpose](#2%EF%B8%8F⃣-purpose)
- [3️⃣ Architecture](#3%EF%B8%8F⃣-architecture)
- [4️⃣ Contents](#4%EF%B8%8F⃣-contents)
- [5️⃣ Conclusion](#5%EF%B8%8F⃣-conclusion)
  
<br>
<br>

## 1️⃣ Contributors
<br>

| <img src="https://avatars.githubusercontent.com/u/87555330?v=4" width="200px"> | <img src="https://avatars.githubusercontent.com/u/82265395?v=4" width="200px"> | <img src="https://github.com/JaeHee-devSpace.png" width="200px"> | <img src="https://avatars.githubusercontent.com/u/115103394?v=4" width="200px"> |
| :---: | :---: | :---: | :---: |
| [김민성](https://github.com/minsung159357) | [구민지](https://github.com/minjee83) | [박재희](https://github.com/JaeHee-devSpace) | [이현정](https://github.com/nanahj) |


<br>


## 2️⃣ Purpose

## 📚 개요

### 🎯 목표
**📌 Hybrid Cloud CI/CD 프로젝트**

- **myserver01** (Jenkins 서버)과 **myserver02** (운영 서버)로 가정
- 하이브리드 클라우드 환경에서의 CI/CD 자동화
- Jenkins를 활용하여 소스 코드 변경 사항을 자동으로 빌드하고, 원격 서버로 배포
- **inotify**를 활용하여 JAR 파일 실행

### ⚙️ 전제 조건
- 자동화
- Jenkins 스크립트
- `.sh` 파일

---



## 3️⃣ Architecture

### 🚀 1단계 Architecture 

![CICD아키텍쳐 drawio (2)](https://github.com/user-attachments/assets/a253bec1-5f32-4fb5-b0c5-1a989d7d529b)


### 🚀 2단계 Architecture 

![CICD아키텍쳐2 drawio](https://github.com/user-attachments/assets/7b6e68be-e5ee-47dc-8839-eeb754781f67)

### 🚀 3단계 Ubuntu 에 jenkins 설치

## 1. **Ubuntu 패키지 업데이트**
```
sudo apt update
```

## 2. **Java 설치**
```
sudo apt install -y fontconfig openjdk-17-jre
```

## 3. **Jenkins GPG 키 추가**
```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

## 4. **Jenkins 패키지 저장소 추가**
```
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

## 5. **패키지 리스트 업데이트**
```
sudo apt update
```

## 6. **Jenkins 설치**
```
sudo apt install -y jenkins
```

## 7. **Jenkins 서비스 실행**
```
sudo systemctl start jenkins
```

## 8. **Jenkins 관리자 비밀번호 확인**
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

## 9. **포트포워딩**
![image](https://github.com/user-attachments/assets/2432c906-3140-459b-89f9-7b8c59d9fb6a)
## 10. **Jenkins 웹 접속**
```
http://<서버 IP>:8080
```

## 11. **플러그인 및 사용자 설정**
- **"Install suggested plugins"** 선택 → 자동 설치 진행  
- 관리자 계정 생성  
- Jenkins URL 설정  

 ## 4️⃣ Contents 

 ### 🛠 구성 요소
 - **myserver01** (CI 서버): 코드 빌드 및 JAR 파일 생성
 - **myserver02** (CD 서버): JAR 파일 배포 및 실행
 
### 🚀 1단계 
- 내 로컬 시스템에서 VM 2개 생성하여 CI/CD 진행 (개인 시스템 CI/CD)
    - **myserver01** → **myserver02**

### 🔄 2단계 
- 내 시스템에 이관 (내 **myserver01** → 다른 PC의 **myserver02**)
    - 네트워크 통신
    - **myserver02**에게 `scp` 명령어로 이관
    - **inotify**로 수정 감지하여 JAR 실행

 ### 🔗 CI/CD 흐름
 1. **Continuous Integration (CI)**
    - GitHub에서 최신 코드 pull
    - Gradle을 사용하여 JAR 파일 빌드
    - 빌드된 JAR 파일을 지정된 bind 폴더에 복사
 
 2. **Continuous Deployment (CD)**
    - scp를 이용해 자신의 myserver01 -> 자신의 myserver02 로 JAR 파일 전송
    - 실행 중인 애플리케이션 중지 후 새로운 JAR 실행
    - ssh 통신 및 포트포워딩을 통해 자신의 myserver01 -> 타인의 myserver02 로 jar 파일 전송 


### 📜 Jenkins Pipeline Script
```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-repo.git'
            }
        }
        stage('Build JAR') {
            steps {
                dir('./project_name') {                   
                  sh 'chmod +x gradlew'    
                  sh './gradlew build'
                  sh 'mvn clean package'
                }
                
            }
        }
        stage('Copy jar') {
            steps {
                script {
                    def jarFile = 'project_name/build/libs/project_name-0.0.1-SNAPSHOT.jar'
                    def targetDir = '/var/jenkins_home/appjar' // 컨테이너 내부 경로
                    sh "cp ${jarFile} ${targetDir}/"
                }
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    scp -i ~/.ssh/id_rsa -r project_name/build/libs/project_name-0.0.1-SNAPSHOT.jar user@myserver02 의 address:/home/user/appjardir/
                    ssh -i ~/.ssh/id_rsa user@myserver02 의 address "pkill -f 'java -jar' || true && nohup java -jar /home/user/appjardir/project_name-0.0.1-SNAPSHOT.jar > /home/user/appjardir/app.log 2>&1 &"
                '''
            }
        }
        stage('Transfer SCP') {
            steps {
                script {
                    sh """
                    scp -i ~/.ssh/id_rsa -P 포트번호 project_name/build/libs/project_name-0.0.1-SNAPSHOT.jar user@옮길 가상머신의 address:/home/user/
                    """
                }
            }
        }
    }
    post {
        success {
            echo ':흰색_확인_표시: 빌드 성공!'
        }
        failure {
            echo ':x: 빌드 실패! 오류 확인 필요!'
        }
    }
}
```

---

## 5️⃣ Conclusion
CI/CD 자동화는 Jenkins를 활용하여 코드 변경 시 자동으로 빌드하고, 원격 서버에 배포하는 과정입니다. 이를 통해 개발 및 배포 과정에서의 효율성을 높일 수 있습니다. 1단계에서는 **개인 시스템에서 CI/CD를 진행**하고, 2단계에서는 **원격 서버 간의 자동화 배포를 설정**하여 실시간 수정 감지 및 실행을 구현했습니다. 이를 통해 **빠르고 안정적인 배포 환경을 구축**할 수 있습니다.

<img src="https://capsule-render.vercel.app/api?type=waving&color=ADEDFF&height=150&section=footer" width="1000" />
