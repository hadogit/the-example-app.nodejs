pipeline {
    agent {
        docker {
            image 'node:6-alpine'
            args '-p 3000:3000'
        }
    }
    stages {
        stage('Build') {
            steps {
				echo "Building ${BRANCH_NAME}"
                sh 'npm install'
            }
        }
        stage('Test') { 
            steps {
                echo "Running Test..."
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