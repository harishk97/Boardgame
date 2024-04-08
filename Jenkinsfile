pipeline {
    agent any
//declartive tools to install for this pipeline
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/harishk97/Boardgame.git'
            }
        }
        // stage('Maven Compile and Test') {
        //     steps {
        //         sh "mvn --version"
        //         sh "mvn compile"
        //         sh "mvn test"
        //     }
        // }
        // stage('trivy file scan') {
        //     steps {
        //         sh "trivy fs --format table -o fs-report.html ."
        //     }
        // }
        // stage('sonar stage') {
        //     steps {
        //         withSonarQubeEnv('sonar'){
        //             sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=Boardgame -Dsonar.java.binaries=. '''
        //         }
        //     }
        // }
        // stage('Quality Gate') { 
        //     steps {
        //         script{
        //             waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
        //         }
        //     }
        // }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Push to Nexus Repo') { 
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -X"
                }
            }
        }
        stage('Docker build and tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockerHub', toolName: 'docker') {
                        sh "docker build -t harry997/boardgame:latest ."
                        sh "docker image ls"
                        sh "sleep 5"
                        sh "docker push harry997/boardgame:latest"
                }
                //I didnt use image scanning for this image. but if required we can add a trivy scan to image
                }
            }
        }
        stage('Kubernetes deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'minikube', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.49.2:8443') {
                    sh "minikube kubectl -- apply -f deployment.yaml"
                    sh "sleep 10"
                    sh "minikube kubectl -- get pods"
                }
            }
        }
        
    }
}
