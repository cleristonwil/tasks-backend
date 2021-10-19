pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'Sonar_Scanner'
            }
            steps {
                withSonarQubeEnv('Sonar_Local') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://192.168.51.19:9000 -Dsonar.login=5c8131b10f90592fcf629929aa73c9b6b80d47b3 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(5)
                timeout(time:1,unit:'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test') {
            steps {
                dir('api-test') {
                git 'https://github.com/cleristonwil/tasks-api-test'
                bat 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git 'https://github.com/cleristonwil/tasks-frontend'
                    bat 'mvn clean package'    
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test') {
            steps {
                dir('functional-test') {
                git 'https://github.com/cleristonwil/tasks-functionall-tests'
                bat 'mvn test'
                }
            }
        }
                def remote = [:]
                remote.name = 'docker-reg-priv'
                remote.host = '192.168.51.19'
                remote.user = 'root'
                remote.password = '758812'
                remote.allowAnyHosts = true
        stage ('Deploy Prod') {
                sshCommand remote: remote, command: "/root/pasta-compartilhada/tasks-backend/docker-compose build"
                sshCommand remote: remote, command: "/root/pasta-compartilhada/tasks-backend/docker compose up -d"
                
            }
        }
    }     
}