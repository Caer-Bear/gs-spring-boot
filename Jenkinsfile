pipeline {
	agent {
		docker {
			image 'adoptopenjdk/openjdk8:latest'
			args '-v $HOME/.m2:/tmp/jenkins-home/.m2'
		}
	}

	triggers {
		pollSCM 'H/10 * * * *'
	}
	options {
		disableConcurrentBuilds()
		buildDiscarder(logRotator(numToKeepStr: '14'))
	}
	stages {
		stage("test: baseline (jdk8)") {
			options { timeout(time: 30, unit: 'MINUTES') }
			steps {
				sh 'test/run.sh'
			}
		}
		stage("test: SonarQube") {
			options { timeout(time: 30, unit: 'MINUTES')}
			steps {
				sh 'chmod +x /complete/gradlew && /complete/gradlew sonarqube'
			}
		}
		stage('Docker Build') {
            steps {
                sh 'docker build -t complete/interview-app:$BUILD_NUMBER .'
            }
        }
		stage('Deployment') {
			steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'kubectl apply -f interview-deployment.yml'
				}
			}
		}

	}

	post {
		changed {
			script {
				emailext(
						subject: "[${currentBuild.fullDisplayName}] ${currentBuild.currentResult}",
						mimeType: 'text/html',
						recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']],
						body: "<a href=\"${env.BUILD_URL}\">${currentBuild.fullDisplayName} is reported as ${currentBuild.currentResult}</a>")
			}
		}
	}
}
 