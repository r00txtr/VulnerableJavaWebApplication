pipeline {
    agent none
    stages {
        stage('Maven Compile') {
            agent {
                label 'Maven-Labels'  // Assumes Maven-Labels agent template is configured and available
            }
            steps {
                sh 'mvn compile'
            }
        }
        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:latest'  // Base docker image; docker:dind isn't necessary here
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'docker rmi java-vulnerable-application'
                sh 'docker build -t java-vulnerable-application:0.1 .'
            }
        }
        stage('Run Docker Image') {
            agent {
                docker {
                    image 'docker:latest'  // Using a docker image without dind for container control
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                // Clean up any existing container with the same name
                sh 'docker rm -f vulnerable-java-application || true' 
                sh 'docker run --name vulnerable-java-application -p 9000:9000 -d java-vulnerable-application:0.1'
            }
        }
    }
}
