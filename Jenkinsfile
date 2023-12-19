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
        stage('Docker') {
            steps {
                dir('Ansible'){
                  script {
                         ansiblePlaybook credentialsId: 'ssh_ansible', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml'
                        }     
                   }    
              }
        }
        stage("Docker image scanning with Trivy"){
            steps {
                script{
                // Install trivy
                sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s  v0.44.1'
                sh 'chmod +x ./bin/trivy'
                sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'
                def dockerImageName = "devsecopswithansible.azurecr.io/petstore:latest"
                // Scan all vuln levels
                sh "./bin/trivy image --ignore-unfixed --scanners vuln --vuln-type os,library --format template --template @html.tpl -o trivy-scan.html ${dockerImageName}"      
                    publishHTML target : [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'trivy-scan.html',
                        reportName: 'Trivy Scan',
                        reportTitles: 'Trivy Scan'
                    ]
        
                }
            }
        }
        stage('CodeQl') {
            steps{
                withCodeQL(codeql: 'codeql') {
                    sh 'codeql pack install test/'
                    sh 'codeql test run test/'
                    }
            }   
        }
        stage('k8s using ansible'){
            steps{
                dir('Ansible') {
                    script{
                        ansiblePlaybook credentialsId: 'ssh_ansible', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'k8s.yaml'
                    }
                } 
            }
        }
    }
}