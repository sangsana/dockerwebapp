pipeline {
    agent {
        node {
            label 'Prod'
        }
    }
    tools {
        maven 'maven'
    }

    stages {
        stage('Code Checkout') {
            steps {
                git "https://github.com/sangsana/dockerwebapp.git"
            }
        }    
        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh "mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=MyProject -Dsonar.projectName='MyProject'"
                }
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn clean package'
                sh 'cp -r target Docker-app'
            }
        }
        stage('Artifact Code Store') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'Nexus-pass', groupId: 'com.visualpathit', nexusUrl: '54.175.211.120:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'MyRepo', version: 'v2'
            }
        }
        stage('Docker Image Build') {
            steps {
                sh 'docker build -t sangsana/swarm:app Docker-app'
                sh 'docker build -t sangsana/swarm:db Docker-db'
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image sangsana/swarm:db'
                sh 'trivy image sangsana/swarm:app'
            }
        }
        stage('Push image to Registry') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DockerHub') {
                        sh 'docker push sangsana/swarm:app'
                        sh 'docker push sangsana/swarm:db'
                    }
                }
            }
        }
        stage('Deploy to Stack') {
            steps {
                sh 'docker stack deploy -c compose.yml SwarmApp'
            }
        }
    }
}
