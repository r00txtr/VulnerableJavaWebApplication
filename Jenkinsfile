pipeline {
    agent any  // Use any available agent for the pipeline
    stages {
        stage('Check Node') {
            steps {
                script {
                    echo "Running on: ${env.NODE_NAME}"  // Outputs the name of the node
                }
            }
        }
        stage('Maven Compile and Static Application Security Testing (SAST)') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-11'  
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint='
                }
            }
            steps {
                sh 'mvn clean compile spotbugs:spotbugs'
                sh 'ls target/'
            }
        }
    }
}
