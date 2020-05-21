// Create hosted npm repo in NXRM (npm-develope)
// enable "npm token bearer realm"


pipeline {
    agent any
    
    environment {
        CI = 'true'
	ARTEFACT_NAME = "${WORKSPACE}/node_modules/"
        DEV_REPO = 'npm-development'
        TAG_FILE = "${WORKSPACE}/tag.json"
        IQ_SCAN_URL = ""
	NPM_USER = 'admin'
	NPM_PASS = 'admin123'
	NPM_EMAIL = 'user@domain.com'
	NPM_REGISTRY = 'http://10.0.0.150:8081/repository/npm-development' 
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install -g npm-cli-login'
            }
        }
        stage('Nexus IQ Scan') {
      		steps {
				script {
					// def policyEvaluation = nexusPolicyEvaluation failBuildOnNetworkError: false, iqApplication: 'juice-shop', iqStage: 'build', jobCredentialsId: 'nodeuser'
                    nexusPolicyEvaluation failBuildOnNetworkError: true, iqApplication: selectedApplication('js_juiceshop'), iqScanPatterns: [[scanPattern: '**/node_modules/*']], iqStage: ${iqStage}, jobCredentialsId: 'nodeuser'
                    // SCAN_URL = "${policyEvaluation.applicationCompositionReportUrl}"
                }
      		}
    	}
        stage('Scan result'){
            steps {
                // echo "scan url: ${SCAN_URL}"
                echo "user: ${USER}"
                echo "job name: ${JOB_NAME}"
                echo "build id: ${BUILD_ID}"
                echo "build url: ${BUILD_URL}"
                echo "build tag: ${BUILD_TAG}"
            }
        }
    }

       stage('Create tag'){
            steps {
                script {
    
                    // Git data (Git plugin)
                    echo "${GIT_URL}"
                    echo "${GIT_BRANCH}"
                    echo "${GIT_COMMIT}"
                    echo "${WORKSPACE}"

                    // construct the meta data (Pipeline Utility Steps plugin)
                    def tagdata = readJSON text: '{}' 
                    tagdata.buildNumber = "${BUILD_NUMBER}" as String
                    tagdata.buildId = "${BUILD_ID}" as String
                    tagdata.buildJob = "${JOB_NAME}" as String
                    tagdata.buildTag = "${BUILD_TAG}" as String
                    tagdata.appVersion = "${BUILD_VERSION}" as String
                    tagdata.buildUrl = "${BUILD_URL}" as String
                    tagdata.iqScanUrl = "${IQ_SCAN_URL}" as String
                    tagdata.gitUrl = "${GIT_BRANCH}" as String
                    //tagData.promote = "no" as String

                    writeJSON(file: "${TAG_FILE}", json: tagdata, pretty: 4)
                    sh 'cat ${TAG_FILE}'

                    createTag nexusInstanceId: 'nexus', tagAttributesPath: "${TAG_FILE}", tagName: "${BUILD_TAG}"

                    // write the tag name to the build page (Rich Text Publisher plugin)
                    rtp abortedAsStable: false, failedAsStable: false, parserName: 'Confluence', stableText: "Nexus Repository Tag: ${BUILD_TAG}", unstableAsStable: true 
                }
            }
        }

        stage('Upload to Nexus Repository'){
            steps {
                script {
		    sh 'npm-cli-login -u ${NPM_USER} -p ${NPM_PASS} -r ${NPM_REGISTRY}'
                    nexusPublisher nexusInstanceId: 'nexus', nexusRepositoryId: "${DEV_REPO}", packages: [[$class: 'NPM', mavenAssetList: [[classifier: '', extension: 'tar', filePath: "${ARTEFACT_NAME}"]], mavenCoordinate: [artifactId: 'Juiceshop', groupId: 'Juiceshop', packaging: 'tar', version: "${BUILD_VERSION}"]]], tagName: "${BUILD_TAG}"
                }
            }
        }

}