@Library('Poc1SharedLib')_
pipeline {
    agent any 
 	environment
	{
	ant_build = "C:\\Apurva\\JenkinsWorkspace\\ant1\\build.xml"
	}
	stages {
        stage('Git Checkout') { 
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/apurva1j/POC1.git']]])
            }
        }
        stage('Test') { 
            steps {
                bat "echo Test"
            }
        }

        stage ('Invoke_pipeline1') {
            steps {
                build job: 'Pipeline1'
            }
        }		
		
		stage('Wait') {
			   steps {
		timeout(time: 2, unit: "HOURS") {
		input message: 'Approve Deploy?', ok: 'Yes'
		}

			   }
		  }
	 stage('Sonarqube') {
		environment {
        scannerHome = tool 'SonarScannerLocal'
		}
		steps {
        withSonarQubeEnv('SonarLocal') {
            bat "${scannerHome}/bin/sonar-scanner"
        }
		}
		}
		stage('Build') {
		steps
		{
		withAnt(installation: 'Ant 1.10.7', jdk: 'jdk-11.0.4') {
		// some block
		bat "ant -f ${ant_build}"
		}
}
		}
		
	stage('upload') {
           steps {
              script { 
                 def server = Artifactory.server 'frogArtifactory'
                 def uploadSpec = """{
                    "files": [{
                       "pattern": "C:/Apurva/Bars/",
                       "target": "SampleRepo"
                    }]
                 }"""

                 server.upload(uploadSpec) 
               }
            }
        }
	stage('sharedLib') {
	steps {
    	HelloWorld('hellooooooooooooooooooooo world')
   	 }

	}	

	stage('Artifactory download') {
	steps {		
	script {
			def server = Artifactory.server 'frogArtifactory'
			def downloadSpec = """{
			"files": [
			{
				"pattern": "SampleRepo/*.bar",
				"target": "C:/Apurva/ArtifactoryDl/"
			}]
			}"""
	//server.download(downloadSpec)
	server.download spec: downloadSpec
	}
	}
	}
	
    stage('UCD deploy')
    {
	steps{
	step([$class: 'UCDeployPublisher',
	component: [componentName: 'IIBComp', componentTag: '',
	delivery: [$class: 'Push', baseDir: 'C:\\Apurva\\ArtifactoryDl', fileExcludePatterns: '', fileIncludePatterns: '*.bar', pushDescription: '', pushIncremental: false, pushProperties: '', pushVersion: '${BUILD_NUMBER}']], 
	deploy: [deployApp: 'IIBDeploy', deployDesc: 'Requested from Jenkins', deployEnv: 'IIBEnv', deployOnlyChanged: false, deployProc: 'IIBDeployAppProcess', deployReqProps: '', deployVersions: 'IIBComp:${BUILD_NUMBER}', skipWait: false], siteName: 'UcdServer'])
	
	}
}	
		
	stage('Deploy') { 
            steps {
                bat "echo Deploy"
            }
        }
    }
}
