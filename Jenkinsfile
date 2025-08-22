pipeline {
    agent any
    tools {
        jdk "jdk17"
        maven "maven3"
    }
    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }
    stages {
        stage('checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/MHaneefa93/Boardgame.git'
            }
        }
       stage('compile') {
            steps {
                sh "mvn compile"    
            }
        }
       stage('test') {
            steps {
                sh "mvn test"
            }
        }
       stage('file system scan') {
            steps {
                sh '''docker run --rm -v $(pwd):/project aquasec/trivy:latest --format table -o /project/trivy-fs.html fs /project'''
             }
        }
       stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectKey=broadgame -Dsonar.projectName=broadgame -Dsonar.java.binaries=."
                }
            }
        }
       stage('quality gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
       stage('build') {
            steps {
                sh "mvn package"
            }
        }
       stage('publish to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
       stage('build & tag docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t haneefa93/broadgame:1.0 ."
                    }    
                }
            }
        }
        stage('push docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker push haneefa93/broadgame:1.0"
                    }    
                }
            }
        }
        stage('deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'minikube', contextName: '', credentialsId: 'k8s', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.49.2:8443') {
                    sh "kubectl apply -f deployment-service.yaml"    
                }    
            }
        }
    }
}