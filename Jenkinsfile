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
        stage('Maven Compile') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-11'  
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint='
                }
            }
            steps {
                sh 'mvn compile'
                sh 'apt-get update -y && apt-get install -y iputils-ping curl dnsutils'
                sh 'ping -c 1 github.com'
                sh 'curl -I https://github.com'
                sh 'nslookup github.com'
            }
        }
        stage('Software Composition Analysis (SCA)') {
            agent {
                docker {
                    image 'owasp/dependency-check:latest'  
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock -v my-docker-volume-dependency-check-data:/usr/share/dependency-check/data --entrypoint='
                }
            }
            steps {
                sh '/usr/share/dependency-check/bin/dependency-check.sh --scan . --project "VulnerableJavaWebApplication" --format HTML --format XML --format JSON'
                archiveArtifacts artifacts: 'dependency-check-report.html'
                archiveArtifacts artifacts: 'dependency-check-report.json'
                archiveArtifacts artifacts: 'dependency-check-report.xml'
            }
        }
        stage('Secret Scanning') {
            agent {
                docker {
                    image 'ghcr.io/trufflesecurity/trufflehog:latest'  
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint='
                }
            }
            steps {
                sh 'trufflehog --no-update filesystem . --json > trufflehogscan.json'
                sh 'cat trufflehogscan.json'
                archiveArtifacts artifacts: 'trufflehogscan.json'
            }
        }
        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:latest'  // Using Docker image
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'docker rmi java-vulnerable-application:0.1 || true'  // Remove existing image if present
                sh 'docker build -t java-vulnerable-application:0.1 .'  // Build the Docker image
            }
        }
        stage('Run Docker Image') {
            agent {
                docker {
                    image 'docker:latest'  // Using Docker image
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                // Clean up any existing container with the same name
                sh 'docker rm -f vulnerable-java-application || true' 
                sh 'docker run --name vulnerable-java-application -p 9000:9000 -d java-vulnerable-application:0.1'  // Run the Docker container
            }
        }
    }
}
