pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "justo123/train-schedule"
    }
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
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
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
        stage('DeployToCanary') {
            when {
                branch 'master'
            }
            steps {
                try{
                withCredentials([usernamePassword(credentialsId:'kube_ssh',usernameVariable:'USERNAME',passwordVariable:'PASSWORD')]){
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no cloud_user@52.70.241.7 kubectl delete service train-schedule-service"
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no cloud_user@52.70.241.7 kubectl delete deployment train-schedule-deployment"
                }
                    
                }
                catch (err) {
                }
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId:'kube_ssh',usernameVariable:'USERNAME',passwordVariable:'PASSWORD')]){
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no cloud_user@52.70.241.7 kubectl delete service train-schedule-service"
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no cloud_user@52.70.241.7 kubectl delete deployment train-schedule-deployment"
                }
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
