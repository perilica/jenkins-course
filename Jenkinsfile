
def Dajanaoutput

def sendEmail(job_name, job_id, job_status, Dajanaoutput)
{
    echo "Saljem email, ${job_name}, ${job_id},${job_status}, ${Dajanaoutput}"
    
}
pipeline {
    agent any
	parameters{
		string (
			name: "ARTIFACT_NAME",
			defaultValue: "def_value.zip",
			description: "description"
		)
		booleanParam (
			name: "FAIL_PIPELINE",
			description: "fail pipeline"
		)
        booleanParam defaultValue: true, name: 'RUN_TEST'
		
	}
    stages {
		stage ('Dynamic')
		{
			echo 'Dynamic'
		}
        stage('Download') {
            steps {
                cleanWs()
                withCredentials (
                    [usernamePassword(credentialsId: "ID1", passwordVariable: "psw", usernameVariable: "usr")]
					)
                {
                    echo "${psw}, ${usr}"
                }
                rtDownload (
                    serverId: 'DajanaID', 
                    spec: ''' {
                        "files": [
							{
							"pattern": "generic-local/libraries/printer.zip",
							"target": "printer.zip",
							"flat": "true"
							}
                        ]
                    }'''
				)
                dir('pipeline') {
                    git branch: 'pipeline', url: 'https://github.com/KLevon/jenkins-course'
                }
				unzip (
					zipFile: "printer.zip",
					dir: "pipeline"
				)
                echo 'Download'
            }
        }
        stage('Build') {
            steps {
                echo 'Build'
				bat (
					script: """
						cd pipeline
						Makefile.bat
					"""
				)
            }
        }
		stage('Tests') {
			when {
				equals expected: true,
				actual: params.RUN_TEST
			}
			steps {
				echo 'Tests'
				script {
					def array = ["printer","scanner", "main"]
					for (element in array)
					{
					Dajanaoutput += bat (
						script: """
							cd pipeline
							Tests.bat ${element}
						""", returnStdout: true
						).trim()
					}		
			
				}
			}
			

        }
        stage('Publish') {
            steps {
                echo 'Publish'
				script {
					zip (
						zipFile: "${params.ARTIFACT_NAME}",
						archive: true,
						dir: "pipeline",
						glob: ""
					)
					}
				rtUpload (
					serverId: 'DajanaID', 
                    spec: """ {
                        "files": [
							{
							"pattern": "${params.ARTIFACT_NAME}",
							"target": "generic-local/release/Dajana/${env.BUILD_ID}/",
							"flat": "true"
							}
                        ]
                    }""" 
				)
				script {
					if (params.FAIL_PIPELINE==true) {
						bat {
						    script: """
        						exit 1
        					"""
						}
						
					}
					
				}
            }
        }
		

		
    }
	post {
			failure {				
				script {
					sendEmail(env.JOB_NAME,env.BUILD_ID, currentBuild.currentResult, Dajanaoutput)
					}
			}
		}
}
