pipeline{
    agent any

    environment {
        EC2_IP = '54.158.63.231'
    }
    
    stages {
        stage('checkout') {
            steps {
                git branch: 'main', credentialsId: 'Github', url: 'https://github.com/manjualex/git_batchdemo.git'
        
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('SonarQube-Analysis') {
            steps {
                withSonarQubeEnv('Sonar-Server-9.9') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=Java-Project'
                }
            }
        }
        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'Stateful-Tracker', classifier: '', file: 'target/Stateful-Tracker-2.0.0-SNAPSHOT.war', type: 'war']], credentialsId: 'Nexus-Credentials', groupId: 'com.microsoft.webapp.samples', nexusUrl: '54.174.179.151:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-build-artifacts', version: '2.0.0-SNAPSHOT'
            }
            }
        stage('Download the Artifact') {
            steps {
               echo "Downloading artifact from Nexus..."
               script {
                def nexusUrl = 'http://54.174.179.151:8081'
                def repository = 'maven-build-artifacts'
                def groupId = 'com.microsoft.webapp.samples'
                def artifactId = 'Stateful-Tracker'
                def version = '2.0.0-SNAPSHOT'
                def file = 'Stateful-Tracker-2.0.0-SNAPSHOT.war'

                def download_url = sh(script: "curl -s 'http://54.174.179.151:8081/service/rest/v1/search?repository=maven-build-artifacts&group=com.microsoft.webapp.samples' | jq -r '.items | sort_by(.version) | last | .assets[0].downloadUrl'", returnStdout: true).trim()
                sh "curl -o ${artifactId}-${version}.war ${download_url}"
               } 
            }
        }
        stage('Deploy Artifact to EC2') {
            steps {
                echo "Deploying artifact to EC2..."
                sshagent(['Deploy-Server-Agent']) {
                    sh 'scp -o StrictHostKeyChecking=no target/Stateful-Tracker-2.0.0-SNAPSHOT.war ubuntu@${EC2_IP}:/var/lib/tomcat9/webapps/ '
                }
            }
        }
       
        }
    post {
        success {
              echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline execution failed. Please check the logs."
        }
    }    
    }
