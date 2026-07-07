pipeline {
    agent any
    options{
        timestamps()
    }
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    environment {
    SCANNER_HOME = tool 'sonar-scanner'
    JDK17_HOME = tool 'jdk-17'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '257d5260-8fce-42a0-ab29-6f854e8f62e1', poll: false, url: 'https://github.com/jaiswaladi246/Ekart.git'
            }
        }
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format HTML --format XML --nvdApiKey d79976d6-cde9-4c94-9a58-84e4e0c20c78', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'

            }
        }
        stage('SonarQube'){
            steps{
               withSonarQubeEnv('sonar-server'){
                    sh '''
                    export JAVA_HOME=$JDK17_HOME
                    export PATH=$JAVA_HOME/bin:$PATH
                    java -version
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Shopping-Cart 
                    '''
                    }
            }
        }
        stage('Build'){
            steps{
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Docker Build & Push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'c20b3140-73ce-4c6a-bd31-d5f71bc3ade8') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart deekshitha57/shopping-cart:latest"
                        sh "docker push deekshitha57/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
}

