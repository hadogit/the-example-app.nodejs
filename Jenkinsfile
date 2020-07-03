ARTIFACTORY_URL = "http://157.175.86.184:8081/artifactory"
ARTIFACT_NAME = "artifacts-${BRANCH_NAME}-${BUILD_ID}.tar.gz"
WORKSPACE = "${JOB_NAME}/${BUILD_ID}"

def DEV_ENVIRONMENT = [:]
DEV_ENVIRONMENT.name = 'dev'
DEV_ENVIRONMENT.host = '15.185.40.243'
DEV_ENVIRONMENT.allowAnyHosts = true
CONTAINER_NAME = "mynodejs"
DeployToStagingJobPath = "Deploy_To_Staging"

pipeline {
    agent any
    tools{
        nodejs "nodeJs14.5"
    }
	options {
        timeout(time: 3, unit: 'HOURS')
        timestamps()
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
				        DEV_ENVIRONMENT.user = "ubuntu"
                        DEV_ENVIRONMENT.identityFile = identity
				    	sshCommand remote: DEV_ENVIRONMENT, command: "wget --user interview --password interview123! ${ARTIFACTORY_URL}/example-repo-local/${BRANCH_NAME}/${ARTIFACT_NAME}"				
				        sshCommand remote: DEV_ENVIRONMENT, command: "tar xvf ${ARTIFACT_NAME}"
						sshCommand remote: DEV_ENVIRONMENT, command: "sudo docker build -t nodejs/${BUILD_ID} ."
				        sshCommand remote: DEV_ENVIRONMENT, command: "sudo docker stop $(sudo docker container ls -aq)"
				        sshCommand remote: DEV_ENVIRONMENT, command: "sudo docker run -d --name=${CONTAINER_NAME} nodejs"

				    }
					echo "Running on ${DEV_ENVIRONMENT.host}:3000"
					currentBuild.description = "${DEV_ENVIRONMENT.host}:3000"
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
		        script{
		    		input message: "Are You Sure?"
                    echo "Deploying To Staging..."
		    		result = build(job: DeployToStagingJobPath,
		    			parameters: [
		    				string(name: "Build_Number", value: BUILD_ID),
		    				string(name: "Branch", value: BRANCH_NAME) ],
		    			wait: true)
                }
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
