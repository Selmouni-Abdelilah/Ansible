pipeline {
    agent any
    tools {
        maven "maven"
        }
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'GitHubcredentials', url: 'https://github.com/Selmouni-Abdelilah/Ansible.git']])
            }
        }
         stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
         }
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
         }
        stage('Build Artifacts') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Code Quality Analysis + SAST'){
            steps {
                script {
                    def scannerHome = tool 'Sonar-Scanner';
                    withSonarQubeEnv(credentialsId: 'token_sonar',installationName:'Sonarqube'){
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=library \
                        -Dsonar.projectName=CICD \
                        -Dsonar.host.url=http://172.201.249.253:9000 \
                        -Dsonar.token=token_sonar \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=."    
                    }
                }       
            }

        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage('Dependency Check Report') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'dependency-check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }    
        }
    }
}