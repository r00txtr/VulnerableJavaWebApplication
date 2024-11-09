pipeline {
    agent any
    stages {
        stage('Maven Compile and SAST') {
            agent {
                docker {
                    image 'maven:3.9.9-eclipse-temurin-11'
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint='
                }
            }
            steps {
                echo 'Running Maven compile and SpotBugs analysis...'
                sh 'mvn clean compile spotbugs:spotbugs'
                sh 'pwd'
                sh 'ls -la'
                sh 'cp target/spotbugsXml.xml ./spotbugsXml.xml'
                sh 'cp target/site/spotbugs.html ./spotbugs.html'
                archiveArtifacts artifacts: 'spotbugs.html'
                archiveArtifacts artifacts: 'spotbugsXml.xml'
                
            }
        }
        
        stage('Dependency Check SCA') {
            agent {
                docker {
                    image 'owasp/dependency-check:latest'
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock -v my-docker-volume-dependency-check-data:/usr/share/dependency-check/data --entrypoint='
                }
            }
            steps {
                echo 'Running OWASP Dependency Check...'
                sh '/usr/share/dependency-check/bin/dependency-check.sh --scan . --project "VulnerableJavaWebApplication" --format HTML --format XML --format JSON'
                archiveArtifacts artifacts: 'dependency-check-report.html'
                archiveArtifacts artifacts: 'dependency-check-report.json'
                archiveArtifacts artifacts: 'dependency-check-report.xml'
            }
        }
        
        stage('TruffleHog Secret Scanning') {
            agent {
                docker {
                    image 'trufflesecurity/trufflehog:latest'
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint='
                }
            }
            steps {
                echo 'Running TruffleHog for secret scanning...'
                sh 'trufflehog --no-update filesystem . --json > trufflehogscan.json'
                archiveArtifacts artifacts: 'trufflehogscan.json'
            }
        }
        
        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo 'Building Docker image...'
                sh 'docker rmi java-vulnerable-application:0.1 || true'
                sh 'docker build -t java-vulnerable-application:0.1 .'
            }
        }
        
        stage('Run Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo 'Starting Docker container...'
                sh 'docker rm -f vulnerable-java-application || true'
                sh 'docker run --name vulnerable-java-application -p 9000:9000 -d java-vulnerable-application:0.1'
            }
        }

        stage('Dynamic Analysis (DAST) with ZAP') {
            agent {
                docker {
                    image 'ghcr.io/zaproxy/zaproxy:stable'
                    args '--privileged -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint= -v .:/zap/wrk/:rw'
                }
            }
            steps {
                 echo 'Running ZAP DAST scan...'
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE'){
                    sh 'zap-full-scan.py -t https://172.18.0.3:9000 -r result-zap-full.html -x result-zap-full.xml'
                }

                sh 'cp /zap/wrk/result-zap-full.html ./result-zap-full.html'
                sh 'cp /zap/wrk/result-zap-full.xml ./result-zap-full.xml'
                archiveArtifacts artifacts: 'result-zap-full.html'
                archiveArtifacts artifacts: 'result-zap-full.xml'
            }
        }
    }
    post {
        always {
            sh 'curl -X POST https://demo.defectdojo.org/api/v2/import-scan/ -H "Authorization: Token 548afd6fab3bea9794a41b31da0e9404f733e222" -F "scan_type=Trufflehog Scan" -F "file=@./trufflehogscan.json;type=application/json" -F "engagement=25"'
            sh 'curl -X POST https://demo.defectdojo.org/api/v2/import-scan/ -H "Authorization: Token 548afd6fab3bea9794a41b31da0e9404f733e222" -F "scan_type=Dependency Check Scan" -F "file=@./dependency-check-report.xml;type=text/xml" -F "engagement=25"'
            sh 'curl -X POST https://demo.defectdojo.org/api/v2/import-scan/ -H "Authorization: Token 548afd6fab3bea9794a41b31da0e9404f733e222" -F "scan_type=SpotBugs Scan" -F "file=@./target/spotbugsXml.xml;type=text/xml" -F "engagement=25"'
            sh 'curl -X POST https://demo.defectdojo.org/api/v2/import-scan/ -H "Authorization: Token 548afd6fab3bea9794a41b31da0e9404f733e222" -F "scan_type=ZAP Scan" -F "file=@./result-zap-full.xml;type=text/xml" -F "engagement=25"'
        }
    }
}
