pipeline {
    agent any
    tools{
        nodejs "nodeJs14.5"
    }
	triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage('Prepare Environment') {
            steps {
				echo "Building ${BRANCH_NAME}"
				sh 'npm install -g jest-cli'
                sh 'npm install'
            }
        }
		stage('Build') {
			steps {
				script {
				sh 'npm start:watch'
				sh 'npm pack'
				}
			}
		}
        stage('Test') { 
            steps {
                echo "Running Test..."
                sh "jest test/unit/*.js --coverage --coverageDirectory=output/coverage/jest"
            }
            post {
                always {
                  step([$class: 'CoberturaPublisher', coberturaReportFile: 'output/coverage/jest/cobertura-coverage.xml'])
                }
            }
        }
		stage('Publish Artifacts') { 
            steps {
                echo "Publishing Artifacts..."
            }
        }
		stage('Deploy To Dev') { 
            steps {
                echo "Deploying To Development..."
            }
        }
    }
	post{
		success {
		    script{
				echo "Success"
			}
		}
	}
}