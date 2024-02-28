pipeline {
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Pramodh003/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Unit test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=.
                    '''
                    
                }
            }
        }
        stage('Owasp dependency check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('Deploy to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Docker build and tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh "docker build -t pramod003/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }
        stage('Trivy Image scan') {
            steps {
                sh 'trivy image pramod003/ekart:latest > trivy-report.txt'
            }
        }
        stage('Docker push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds', toolName: 'docker') {
                        sh "docker push pramod003/ekart:latest"
                    }
                }
            }
        }
        stage('Kubernetes deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'test', restrictKubeConfigAccess: false, serverUrl: 'https://b9ecc628-862d-45ae-9117-202f94b83fb4.k8s.ondigitalocean.com') {
                    sh 'kubectl apply -f deploymentservice.yaml -n test'
                    sh "kubect get svc -n test"
                    
                }
            }
        }
    }
}
