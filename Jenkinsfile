pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'Nodejs'
    }
    environment {
        SCANNER_HOME=tool 'sonarscanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', credentialsId: 'docker', url: 'https://github.com/Tanaji199/bingo1.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonarserver') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Bingo \
                    -Dsonar.projectKey=Bingo '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonartoken' 
                } 
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
            stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t bingo ."
                       sh "docker tag bingo tanaji119/bingo:latest "
                       sh "docker push tanaji119/bingo:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image tanaji/bingo:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name bingo -p 3000:3000 tanaji119/bingo:latest'
            }
        }

    }
}
