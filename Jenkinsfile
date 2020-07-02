
ARTIFACTS_NAME = "artifacts-${BRANCH_NAME}-${BUILD_ID}.zip"
WORKSPACE = "${JOB_NAME}/${BUILD_ID}"
pipeline {
    agent any
    tools{
        nodejs "nodeJs14.5"
    }
	triggers {
        pollSCM('* * * * *')
    }
	 parameters {
        booleanParam(name: 'Deploy_To_Staging', defaultValue: false, description: 'Do You Want To Deploy To Staging?')
		}
    stages {
        stage('Prepare Environment') {
            steps {
				echo "Building ${BRANCH_NAME}"
				sh 'npm install -g jest-cli'
                sh 'npm install'
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
                    script{
                        echo "Publishing Artifacts..."
                        sh "tar -zcvf ../${ARTIFACTS_NAME} ."
                        def server = Artifactory.server 'ART'
                        
                        def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "../${ARTIFACTS_NAME}",
                              "target": "example-repo-local/${BRANCH_NAME}/"
                            }
                         ]
                        }"""
                        server.upload spec: uploadSpec                        
                    }

 
//                    sh "jfrog rt ping --url=http://157.175.86.184:8081"
//                    sh "jfrog rt u artifacts-${BUILD_ID}.gzip example-repo-local --url=http://157.175.86.184:8081 --user=interview --password=interview123!"
                    
                }
        }
		stage('Deploy To Dev') { 
            steps {
                echo "Deploying To Development..."
            }
        }
		stage('Deploy To Staging'){
		    when {
                expression {
                    params.Deploy_To_Staging
                }
            }
		steps {
                echo "Deploying To Staging..."
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
