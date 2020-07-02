ARTIFACTORY_URL = "http://157.175.86.184:8081/artifactory"
ARTIFACT_NAME = "artifacts-${BRANCH_NAME}-${BUILD_ID}.tar.gz"
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
                        sh "tar -zcvf ../${ARTIFACT_NAME} ."
                        def server = Artifactory.server 'ART'
                        
                        def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "../${ARTIFACT_NAME}",
                              "target": "example-repo-local/${BRANCH_NAME}/"
                            }
                         ]
                        }"""
                        server.upload spec: uploadSpec                        
                    }
                    
                }
        }
		stage('Deploy To Dev') { 
            steps {
                script{
                    echo "Deploying To Development..."
				    withCredentials([sshUserPrivateKey(credentialsId: 'environment-user', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'ubuntu')]) {
				        devEnvironment.user = "ubuntu"
                        devEnvironment.identityFile = identity
				    	sshCommand remote: devEnvironment, command: "wget --user interview --password interview123! ${ARTIFACTORY_URL}/example-repo-local/${BRANCH_NAME}/${ARTIFACT_NAME}"				
				        sshCommand remote: devEnvironment, command: "tar xvf ${ARTIFACT_NAME} && npm run start:dev"
				    }  
                }
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
