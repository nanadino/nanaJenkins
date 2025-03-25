
<img src="https://capsule-render.vercel.app/api?type=waving&color=ADEDFF&height=300&section=header&text=🐳Docker-Jenkins🐳&fontSize=55&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />


<br>

## 📍 Outline
- [1️⃣ Contributors](#1%EF%B8%8F⃣-contributors)
- [2️⃣ Purpose](#2%EF%B8%8F⃣-purpose)
- [3️⃣ Contents](#3%EF%B8%8F⃣-contents)
- [4️⃣ Performance Optimization](#4%EF%B8%8F⃣-performance-optimization)
- [5️⃣ Conclusion](#5%EF%B8%8F⃣-Conclusion)
  
<br>
<br>

## 1️⃣ Contributors
<br>

| <img src="https://avatars.githubusercontent.com/u/87555330?v=4" width="200px"> | <img src="https://avatars.githubusercontent.com/u/82265395?v=4" width="200px"> | <img src="https://github.com/JaeHee-devSpace.png" width="200px"> | <img src="https://avatars.githubusercontent.com/u/115103394?v=4" width="200px"> |
| :---: | :---: | :---: | :---: |
| [김민성](https://github.com/minsung159357) | [구민지](https://github.com/minjee83) | [박재희](https://github.com/JaeHee-devSpace) | [이현정](https://github.com/nanahj) |


<br>

# Hybrid Cloud CI/CD 프로젝트

## 📌 개요
하이브리드 클라우드 환경에서의 CI/CD 자동화 목표.Jenkins를 활용하여 소스 코드 변경 사항을 자동으로 빌드하고, 원격 서버로 배포 및 실행하는 과정을 포함
 
 ## 🚀 1단계: 개인 시스템 CI/CD
 ### 🛠 구성 요소
 - **myserver01** (CI 서버): 코드 빌드 및 JAR 파일 생성
 - **myserver02** (CD 서버): JAR 파일 배포 및 실행
 
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




## 🔥 결론
이 프로젝트를 통해 Jenkins 기반의 자동화된 CI/CD 파이프라인을 구축하고, 하이브리드 클라우드 환경에서도 원활한 배포를 실현. 로컬 및 원격 시스템 간의 경계를 허물고, 배포 자동화의 유연성을 극대화





<img src="https://capsule-render.vercel.app/api?type=waving&color=ADEDFF&height=150&section=footer" width="1000" />
