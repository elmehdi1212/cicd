pipeline {
    agent any
	tools{
		jdk 'java2107'
		maven 'maven387'
	}
	environment{
      SONAR_SCANNER_HOME = tool 'sonar7'
	  IMAGE_NAME= "java-app"
	  IMAGE_TAG = "${BUILD_NUMBER}"
	}
    stages {
        stage('Initialize Pipeline'){
            steps {
                echo 'Initializing Pipeline ...'
				sh 'java --version'
				sh 'mvn --version'
            }
        }
        stage('Checkout GitHub Codes'){
            steps {
                echo 'Checking out GitHub Codes ...'
				checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jenkins-lc', url: 'https://github.com/elmehdi1212/cicd.git']])
		    }
        }
        stage('Maven Build'){
            steps {
                echo 'Building Java App with Maven'
				sh 'mvn clean  package'
			
            }
        }
        stage('JUnit Test of Java App'){
            steps {
                echo 'JUnit Testing'
				sh 'mvn test'
		
            }
        }
        stage('SonarQube Analysis'){
            steps {
                echo 'Running Static Code Analysis with SonarQube'
				withCredentials([string(credentialsId: 'sonar-lc', variable: 'sonarToken')]) {
					withSonarQubeEnv('sonar') {

					  sh  """
	                     ${SONAR_SCANNER_HOME}/bin/sonar-scanner  \
				         -Dsonar.projectKey=jenkins-lc  \
	                     -Dsonar.sources=.   \
                         -Dsonar.host.url=http://sonarqube-dind:9000   \
						 -Dsonar.java.binaries=target/classes \
                          -Dsonar.token=$sonarToken
		                   """

					}
             }
	
            }
        }
        stage('Trivy FS Scan'){
            steps {
                echo 'Scanning File System with Trivy FS ...'
				sh 'trivy fs --format table -o FSScanReport.html'
		
            }
        }
        stage('Build & Tag Docker Image'){
            steps {
                echo 'Building the Java App Docker Image'
				script {
					sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
				}
		
            }
        }
        stage('Trivy Security Scan'){
            steps {
                echo 'Scanning Docker Image with Trivy'
				sh 'trivy --severity HIGH,CRITICAL --cache-dir ${WORKSPACE}/.trivy-cache --no-progress --format table -o trivyFSScanReport.html image ${IMAGE_NAME}:${IMAGE_TAG}'
		
            }
        }

	    stage('Authenticated with GCP, tag and push image to artifact registry'){

			steps{
				echo 'Authenticated with GCP, tag and push image to artifact registry'
			}

		}

}
}
