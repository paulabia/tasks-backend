pipeline {
    agent any
    stages {
        stage ('Build Backend'){
            steps {// Criando o passo de build sem executar os testes
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests'){
            steps {// rodando testes, não precisa rodar o clean, pois já gerou o .war no step anterior
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis'){
            environment {// Variavel de ambiente com o nome do plugin configurado do sonar
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL'){
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=e3edb0e8f5a9a868e8603d6d0efc9cc13611bd7c -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**/Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(10) // esperando para que qdo chamar a resposta esteja pronta. 
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend'){
            steps {
                deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('Deploy Frontend'){
            steps {
                dir('frontend') {// Baixando o código num diretorio específico
                    git credentialsId: 'github_login', url: 'https://github.com/paulabia/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
                
            }
        }
        stage ('Deploy Prod') {
            steps {
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
    }
}

