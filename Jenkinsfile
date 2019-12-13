
pipeline {
    agent any

    environment {
        TAG_FILE = "${WORKSPACE}/tag.json"
        IQ_SCAN_URL = ""
        PLUGIN_PATH = "${WORKSPACE}/${PLUGIN_NAME}.hpi"
        PLUGIN_INDEX = "https://updates.jenkins-ci.org/download/plugins"
        PLUGIN_REPO = 'jenkins-plugins'
    }

    stages {
        stage('Download plugin file') {
            steps {
                sh "wget ${PLUGIN_INDEX}/${PLUGIN_NAME}/${PLUGIN_VERSION}/${PLUGIN_NAME}.hpi"
            }
            post {
                success {
                    echo 'Got the plugin file...'
                    sh 'ls -l'
                }
            }
        }

        stage('Nexus IQ Scan'){
            steps {
                script{
                
                    try {
                        def policyEvaluation = nexusPolicyEvaluation failBuildOnNetworkError: true, iqApplication: "${PLUGIN_NAME}", iqScanPatterns: [[scanPattern: '**/*.hpi']], iqStage: 'release', jobCredentialsId: 'admin'
                        echo "Nexus IQ scan succeeded"
                        IQ_SCAN_URL = "${policyEvaluation.applicationCompositionReportUrl}"
                    } 
                    catch (error) {
                        def policyEvaluation = error.policyEvaluation
                        echo "Nexus IQ scan vulnerabilities detected"
                    }
                }
            }
        }

        stage('Create tag'){
            steps {
                script {
                        
                    // construct the meta data (Pipeline Utility Steps plugin)
                    def tagdata = readJSON text: '{}' 
                    tagdata.buildUser = "${USER}" as String
                    tagdata.buildNumber = "${BUILD_NUMBER}" as String
                    tagdata.buildId = "${BUILD_ID}" as String
                    tagdata.buildJob = "${JOB_NAME}" as String
                    tagdata.buildTag = "${BUILD_TAG}" as String
                    tagdata.appVersion = "${BUILD_VERSION}" as String
                    tagdata.buildUrl = "${BUILD_URL}" as String
                    tagdata.iqScanUrl = "${IQ_SCAN_URL}" as String

                    writeJSON(file: "${TAG_FILE}", json: tagdata, pretty: 4)
                    sh 'cat ${TAG_FILE}'

                    createTag nexusInstanceId: 'nxrm3', tagAttributesPath: "${TAG_FILE}", tagName: "${BUILD_TAG}"

                    write the tag name to the build page (Rich Text Publisher plugin)
                    rtp abortedAsStable: false, failedAsStable: false, parserName: 'HTML', stableText: "Nexus Repository Tag (Stable): ${BUILD_TAG}", unstableText: "Nexus Repository Tag (Unstable): ${BUILD_TAG}", unstableAsStable: false 
                }
            }
        }

        // stage('Upload to Nexus Repository'){
        //     steps {
        //         script {
        //             nexusPublisher nexusInstanceId: 'nxrm3', nexusRepositoryId: "${DEV_REPO}", packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: 'war', filePath: "${ARTEFACT_NAME}"]], mavenCoordinate: [artifactId: 'WebGoat', groupId: 'org.demo', packaging: 'war', version: "${BUILD_VERSION}"]]], tagName: "${BUILD_TAG}"
        //         }
        //     }
        // }
    }
}

