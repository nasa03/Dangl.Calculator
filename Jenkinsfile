pipeline {
    options {
        disableConcurrentBuilds()
    }
    agent {
        node {
            label 'master'
            customWorkspace 'workspace/Dangl.Calculator'
        }
    }
    environment {
        KeyVaultBaseUrl = credentials('AzureCiKeyVaultBaseUrl')
        KeyVaultClientId = credentials('AzureCiKeyVaultClientId')
        KeyVaultClientSecret = credentials('AzureCiKeyVaultClientSecret')
    }
    stages {
        stage ('Tests'){
			parallel {
		        stage ('Linux Test') {
					agent {
						node {
							label 'linux'
						}
					}
		        	steps {
		        		sh 'bash build.sh LinuxTest -configuration Debug'
		        	}
		        	post {
                        always {
							xunit(
								testTimeMargin: '3000',
								thresholdMode: 1,
								thresholds: [
									failed(failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0'),
									skipped(failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0')
								],
								tools: [
									xUnitDotNet(deleteOutputFiles: true, failIfNotNew: true, pattern: '**/*testresults.xml', stopProcessingIfError: true)
								])
						}
		        	}
		        }
                stage ('Windows Test') {
                    steps {
                        powershell './build.ps1 Coverage -configuration Debug'
                    }
                    post {
                        always {
                            recordIssues(
								tools: [
									msBuild(), 
									taskScanner(
										excludePattern: '**/*node_modules/**/*', 
										highTags: 'HACK, FIXME', 
										ignoreCase: true, 
										includePattern: '**/*.cs, **/*.g4, **/*.ts, **/*.js', 
										normalTags: 'TODO')
									])
                            xunit(
                                testTimeMargin: '3000',
                                thresholdMode: 1,
                                thresholds: [
                                    failed(failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0'),
                                    skipped(failureNewThreshold: '0', failureThreshold: '0', unstableNewThreshold: '0', unstableThreshold: '0')
                                ],
                                tools: [
                                    xUnitDotNet(deleteOutputFiles: true, failIfNotNew: true, pattern: '**/*testresults.xml', stopProcessingIfError: true)
                                ])
                            cobertura(
                                coberturaReportFile: 'output/Cobertura.xml',
                                failUnhealthy: false,
                                failUnstable: false,
                                maxNumberOfBuilds: 0,
                                onlyStable: false,
                                zoomCoverageChart: false)
                        }
                    }
                }
			}
		}
        stage ('Deploy') {
            steps {
                powershell './build.ps1 UploadDocumentation+PublishGitHubRelease'
            }
        }
    }
    post {
        always {
            step([$class: 'Mailer',
                notifyEveryUnstableBuild: true,
                recipients: "georg@dangl.me",
                sendToIndividuals: true])
            cleanWs()
        }
    }
}
