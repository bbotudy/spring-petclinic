pipeline {
    agent any
    tools {
        jdk 'JDK21'
        maven 'M3'
    }
    environment {
        DOCKERHUB_CRED = credentials('1jo')
    }

    stages {
        // Github로부터 소스코드 다운로드
        stage('Git Clone') {
            steps {
                echo 'Git clone'
                git url: 'https://github.com/bbotudy/spring-petclinic.git',
                branch: 'main'
            }
        }

        // Maven을 이용한 Build
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }

        // Docker Image 생성
        stage('Docker Image Create') {
            steps {
                echo 'Docker Image Create'
                sh '''
                sh 'echo ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-stdin'
                docker build -t spring-petclinic:${BUILD_NUMBER} .
                docker tag spring-petclinic:${BUILD_NUMBER} ldy88/spring-petclinic:latest
                '''
            }
        }

        // Docker hub Image Push
        stage('Docker Image Push') {
            steps {
                echo 'Docker Image Push'
                sh 'echo ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-stdin'
                sh 'docker image push ldy88/spring-petclinic:latest'
            }
            post {
                always {
                    sh '''
                    docker rmi -f spring-petclinic:${BUILD_NUMBER}
                    docker rmi -f ldy88/spring-petclinic:latest
                    '''
                }
            }
        }

        // WAS01 배포
        stage('Deploy WAS01') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'was01',
                transfers: [sshTransfer(
                execCommand: '''
                docker rm -f spring-petclinic || true
                docker pull ldy88/spring-petclinic:latest
                docker run -itd -p 80:8080 --name=spring-petclinic ldy88/spring-petclinic:latest
                ''',
                execTimeout: 120000)],
                verbose: true)])
            }
        }

        // WAS02 배포
        stage('Deploy WAS02') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'was02',
                transfers: [sshTransfer(
                execCommand: '''
                docker rm -f spring-petclinic || true
                docker pull ldy88/spring-petclinic:latest
                docker run -itd -p 80:8080 --name=spring-petclinic ldy88/spring-petclinic:latest
                ''',
                execTimeout: 120000)],
                verbose: true)])
            }
        }
    }
}
