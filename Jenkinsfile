pipeline{
    
    agent any
	tools{
		maven 'maven'
	} 
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/Risheekdev/demo-counter-app.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('Maven build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
        stage('sonarqube'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-key') {
                        sh 'mvn clean package sonar:sonar'
                }
            }
        }
        
    }
    stage('Quatity gate checking'){

        steps{

            script{
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-key'
            }
        }
    }
    stage('upload war to nexus'){

        steps{

            script{

                def readPomVersion = readMavenPom file: 'pom.xml'
                def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "poc-snapshot" : "poc-project"

                nexusArtifactUploader artifacts: [[artifactId: 'springboot', classifier: '', file: 'target/Uber.jar', type: 'jar']], credentialsId: 'nexus-auth', groupId: 'com.example', nexusUrl: '13.233.104.249:8081', nexusVersion: 'nexus3', protocol: 'http', repository: nexusRepo, version: "${readPomVersion.version}"
            }
        }
    }
    stage('Docker image Build'){
        steps{

            script{

                sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                sh 'docker image tag $JOB_NAME:v1.$BUILD_ID rishi0310/$JOB_NAME:v1.$BUILD_ID'
                sh 'docker image tag $JOB_NAME:v1.$BUILD_ID rishi0310/$JOB_NAME:latest'

            }
        }
    }
    stage('push image to dockerHub'){

        steps{

            script{

                withCredentials([usernameColonPassword(credentialsId: 'doc-cred', variable: 'dockerhub-cred')]) {
                 
                    sh 'docker login -u rishi0310 -p ${doc-cred}' 
                    sh 'docker push tag $JOB_NAME:v1.$BUILD_ID rishi0310/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker push tag $JOB_NAME:v1.$BUILD_ID rishi0310/$JOB_NAME:latest'



                }

            }
        }
    }

        
  }

}
