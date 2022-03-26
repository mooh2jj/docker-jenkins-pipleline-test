pipeline{
    agent any

    stages {

        stage('Prepare') {
            agent any

            steps {
                git credentialsId: 'git-creds', url: 'https://github.com/mooh2jj/docker-jenkins-pipleline-test.git'
            }

            post {

                success {
                    echo 'prepare success'
                }

                always {
                    echo 'done prepare'
                }

                cleanup {
                    echo 'after all other post conditions'
                }
            }
        }

        stage('Build Gradle') {
            steps{
                sh 'chmod +x gradlew'
                sh  './gradlew clean build'

                sh 'ls -al ./build'
            }
        }
        stage('Docker build image'){
            steps{
                sh 'docker build . -t mooh2jj/docker-jenkins-pipeline-test'
            }
        }
        stage('Docker push image') {
            steps {
                withCredentials([string(credentialsId: 'docker-pwd', variable: 'dockerHubPwd')]) {
                    sh "docker login -u mooh2jj -p ${dockerHubPwd}"
                }
                sh 'docker push mooh2jj/docker-jenkins-pipeline-test'
            }

            post {
                success {
                    echo 'success'
                }

                failure {
                    echo 'failed'
                }
            }
        }
        stage('Run Container on SSH Dev Server'){
            steps{
                echo 'SSH'
                sshagent (credentials: ['ubuntu-server2']) {
                    // 인스턴스 중지 후 다시 하면 오류남 -> run 만 돌리고 그 다음 이 3명령어로 하면 성공됨
                    // 처음에 run -> 기존 run한 컨테이너 제거위한 filter 명령어 docker rm -> 그리고 새로 run
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.40.133 'docker pull mooh2jj/docker-jenkins-pipeline-test'"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.40.133 'docker ps -q --filter name=docker-jenkins-pipleline | grep -q . && docker rm -f \$(docker ps -aq --filter name=docker-jenkins-pipleline)'"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.40.133 'docker run -d --name docker-jenkins-pipleline -p 8080:8080 mooh2jj/docker-jenkins-pipeline-test'"
                }

            }

        }

        stage("Slack Notification") {
            steps {
                echo 'slack test'
            }
            post {
                success {
                    slackSend channel: '#프로그래밍', color: 'good', message: "success deploy"
                }
                failure {
                    slackSend channel: '#프로그래밍', color: 'danger', message: "failure deploy"
                }
            }
        }

    }
}