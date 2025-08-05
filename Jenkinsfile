pipeline {
	// pipeline: Declarative Pipeline 문법의 시작.
	//agent any: 워크스페이스가 설정된 아무 노드(마스터든 에이전트든)에서 실행.
    agent any

    environment {
        // 배포 대상 서버 정보 (Jenkins → Manage → Configure System 에 정의해도 좋습니다)
        // DEPLOY_USER = 원격 서버에 접속할 유저 (jenkins)
        // DEPLOY_HOST = 배포 대상 서버 IP/호스트명
        // DEPLOY_PATH = JAR을 복사하고 실행할 폴더 경로
        DEPLOY_USER = 'jenkins'
        DEPLOY_HOST = '10.0.2.15'
        DEPLOY_PORT = '22'
        DEPLOY_PATH = '/home/jenkins/apps/demo'
    }

    stages {
        stage('Checkout') {
            steps {
                // GitHub SSH 주소 + credentialsId는 Jenkins에 등록한 SSH 키 ID
                git credentialsId: 'wamsi98',
                    url: 'git@github.com:wamsi98/jenkins.git',
                    branch: 'main'
            }
        }

        stage('Build & Test') {
            steps {
                // Gradle wrapper로 클린 빌드, 테스트 포함
                // Gradle Wrapper의 역할 ?
                // clean -> 이전 빌드 결과물 삭제 + build -> 컴파일 + 단위테스트 실행
                // gradlew 실행권한 부여
                sh 'chmod +x gradlew && ./gradlew clean build'
            }
        }

        stage('Archive Artifact') {
            steps {
                // build/libs/ 아래 생성된 JAR 파일을 Jenkins에 보관
                // fingerprint: true로 설정하면, 이 아티팩트가 어떤 빌드에서 나왔는지 추적 가능
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                // deploy-ssh: 원격 서버 접속용 SSH 키를 Jenkins Credentials에 등록한 ID
                // sshagent 블록 내부에서만 SSH 에이전트 포워딩이 활성화됩니다.
                // credentials: ['deploy-ssh'] → Jenkins에 등록된 원격 서버 접속용 SSH 비공개 키 ID
                sshagent (credentials: ['deploy-ssh']) {
					sh "ssh -p 22 jenkins@127.0.0.1 'pkill -f JenkinsApplication || true'"
  					sh "scp -P 22 build/libs/*.jar jenkins@127.0.0.1:${DEPLOY_PATH}/"
  					sh "ssh -p 22 jenkins@127.0.0.1 'nohup java -jar ${DEPLOY_PATH}/*.jar > ${DEPLOY_PATH}/app.log 2>&1 &'"
                }
            }
        }
    }

    post {
        failure {
            // 배포 실패 시 알림(이메일, Slack 등) 추가 가능
            echo 'Pipeline failed!'
        }
    }
}