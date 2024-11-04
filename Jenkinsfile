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
                sh 'git config --global --unset credential.helper && git config --system --unset credential.helper'
                sh 'mvn clean compile spotbugs:spotbugs'
                archiveArtifacts artifacts: 'target/spotbugs.html'
                archiveArtifacts artifacts: 'target/spotbugsXml.xml'
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
                    image 'trufflesecurity/trufflehog:latest'  
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint='
                }
            }
            steps {
                sh 'trufflehog --version'
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
