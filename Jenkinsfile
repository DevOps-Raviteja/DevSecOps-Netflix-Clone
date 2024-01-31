pipeline {
    agent any 
    tools {
        jdk 'JDK-17'
        nodejs 'Nodejs-16'
    }
    environment {
        SCANNER_HOME    =   tool 'sonar-scanner'
        RELEASE         =   "version"
        IMAGE_TAG       =   "${RELEASE}-${BUILD_NUMBER}"
    }
    stages {
        stage ('Clean Work-Space'){
            steps {
                cleanWs()
            }
        }
        stage ('Git-CheckOut'){
            steps {
                git branch: 'master', url: 'https://github.com/Ravitejadarla5/DevSecOps-Netflix-Clone.git'
            }
        }
        stage ('Static-Code-Analysis'){
            steps {
                withSonarQubeEnv('sonar-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage ('Quality-Gate'){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage ('Install-Dependency'){
            steps{
                sh 'npm install'
            }
        }
        stage ('Vulnerabilities-Check'){
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        stage ('File-System-Scan'){
            steps{
                sh 'trivy fs .'
            }
        }
        stage ('Docker-Build') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', tool: 'docker'){
                        sh "docker build --build-arg TMDB_V3_API_KEY=cf7d34a30edb3e623a8c3529bdaf0c52 -t netflix-app:${IMAGE_TAG} ."
                    }
                }
            }
        }
        stage ('Docker-Image-Scan'){
            steps {
                sh "trivy image netflix-app:${IMAGE_TAG}"
            }
        }
        stage ('Docker-Push'){
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', tool: 'docker'){
                        sh "docker tag netflix-app:${IMAGE_TAG} ravitejadarla5/netflix-app:latest"
                        sh "docker push ravitejadarla5/netflix-app:latest"
                    }
                }
            }
        }
        stage ('Clear-Artfacts'){
            steps{
                sh "docker image rmi ravitejadarla5/netflix-app:latest"
            }
        }
    }
}