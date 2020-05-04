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
                sleep(10)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}

