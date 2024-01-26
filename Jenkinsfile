def COLOR_MAP = [
	'SUCCESS' : 'good',
	'FAILURE' : 'danger',
	]

pipeline {
	agent any

	environment {
		registryCredentials = 'ecr:us-east-1:aws-key'
		appRegistry = '452059184141.dkr.ecr.us-east-1.amazonaws.com/cjaydevops'
		registryURL = 'https://452059184141.dkr.ecr.us-east-1.amazonaws.com'
		cluster = 'cjaydevops2'
		service = 'cjaydevopsSVC'

	}

	tools {
		maven "MAVEN3"
		jdk "OracleJDK11"
	}

	stages {
		stage('Fetch code') {
			steps {
				git branch: 'docker', url: 'https://github.com/hkhcoder/vprofile-project.git'
			}
		}
		stage('Test') {
			steps {
				sh 'mvn test'
			}
		}
		stage('Checkstyle Analysis') {
			steps {
				sh 'mvn checkstyle:checkstyle'
			}
		}
		stage('Sonar Analysis') {
			environment {
				scannerHome = tool 'sonar4.7'
			}
			steps {
				withSonarQubeEnv('SonarCloud') {
					sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=cjayDevops \
					-Dsonar.organization=cjay-practice-projects \
					-Dsonar.projectName=cjayDevops \
					-Dsonar.projectVersion=1.0 \
					-Dsonar.scanner.force-deprecated-java-version=true \
					-Dsonar.sources=src/ \
					-Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
					-Dsonar.junit.reportsPath=target/surefire-reports/ \
					-Dsonar.jacoco.reportsPath=target/jacoco.exec \
					-Dsonar.java.checkstyle.reportpaths=target/checkstyle-result.xml'''
				}
			}
		}
		stage('Quality Gate') {
			steps {
				timeout(time: 1, unit: 'HOURS') {
					waitForQualityGate abortPipeline: true
				}
			}
		}
		stage('Build Image') {
			steps {
				script {
					dockerImage = docker.build(appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
				}
			}
		}
		stage('Push Image to ECR') {
			steps {
				script {
					docker.withRegistry(registryURL, registryCredentials) {
					dockerImage.push("$BUILD_NUMBER")
					dockerImage.push('latest')
				}
				}
				
			}
		}
		stage('Deploy to ECS') {
			steps {
				withAWS(credentials: 'aws-key ', region: 'us-east-1') {
					sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
				}
			}
		}
	}
	post {
		always {
			slackSend channel: '#crm', 
			color: COLOR_MAP[currentBuild.currentResult],
			message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n"

		}
	}

}
