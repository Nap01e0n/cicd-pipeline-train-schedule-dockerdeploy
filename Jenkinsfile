pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
          when {
            branch 'master'
          }
          steps {
            script {
              app = docker.build("nap01e0n/train_schedule")
              app.inside {
                sh 'echo $(curl localhost:80800)'
              }
            }
          }
        }
        stage('Push Docker Image') {
          when {
            branch 'master'
          }
          steps {
            script {
              docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                app.push("${env.BUILD_NUMBER}")
                app.push("latest")
              }
            }
          }
        }
        stage('DeployToProduction') {
          when {
            branch 'master'
          }
          steps {
            input 'Deploy To Production?'
            milestone(1)
            withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
              script {
                       sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@54.224.230.46 \"docker pull nap01e0n/train_schedule:${env.BUILD_NUMBER}\""
                       try {
                           sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@54.224.230.46 \"docker stop train_schedule\""
                           sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@54.224.230.46 \"docker rm train_schedule\""
                       } catch (err) {
                           echo: 'caught error: $err'
                       }
                       sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@54.224.230.46 \"docker run --restart always --name train_schedule -p 8080:8080 -d nap01e0n/train_schedule:${env.BUILD_NUMBER}\""
                   }
            }
          }
        }
    }
}
