
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
    steps{
        git branch: 'main', url: 'https://github.com/qoonleyh/Devsecops-Netflix-Project.git'
    }
}
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate") {
    steps {
        timeout(time: 2, unit: 'MINUTES') {
            script {
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
            }
        }
    }
}
stage('Fix Dockerfile') {
    steps {
        sh "sed -i 's/node:16.17.0-alpine/node:18-alpine/g' Dockerfile"
    }
}
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
  stage('OWASP FS SCAN') {
    steps {
        echo 'OWASP Dependency Check skipped - NVD feed unavailable'
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
                   withDockerRegistry(credentialsId: 'docker'){  
                       sh "docker build --build-arg TMDB_V3_API_KEY=YOUR_TMDB_API_KEY -t netflix ."
                       sh "docker tag netflix qoonleyh/netflix:latest "
                       sh "docker push qoonleyh/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image qoonleyh/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 qoonleyh/netflix:latest'
            }
        }
    }
    post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'ommogaji@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
    }
}
}
