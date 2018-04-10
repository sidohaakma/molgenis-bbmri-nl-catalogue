pipeline {
    agent {
        label 'devtools'
    }

    tools {
        // Has to be configured on the host with this name : [ mvn-3.5.3 ]
        maven 'mvn-3.5.3'
    }

    parameters {
        string(name: 'VERSION', defaultValue: 'x.x.x')
    }

    environment {
        MOLGENIS_ARTIFACT_ID=
        MOLGENIS_REPOSITORY = "https://spacewalk.hpc.rug.nl"
        MOLGENIS_GIT_SETTINGS='git-settings'
    }

    stages {
        stage('Preparation') {
            steps {
                // Clean workspace
                cleanWs()
                // Get code from git.webhosting.rug.nl
                checkout scm
                // read pom
                def pom = readMavenPom file: 'pom.xml'
                MOLGENIS_ARTIFACT_ID=pom.artifactId
                // Generate the release.properties
                sh ".release/generate_release_properties.bash ${MOLGENIS_ARTIFACT_ID} org.molgenis ${params.VERSION}"
            }
        }
        stage('Prepare release of package') {
            steps {
                echo "Prepare release package [ ${MOLGENIS_ARTIFACT_ID} - ${params.VERSION} ]"
                configFileProvider(
                        [configFile(fileId: '${MOLGENIS_GIT_SETTINGS}', variable: 'MAVEN_SETTINGS')]) {
                    sh "mvn -s ${MAVEN_SETTINGS} clean release:prepare -B"
                }
            }
        }
        stage('Perform release of package') {
            steps {
                echo "Perform release package [ ${MOLGENIS_ARTIFACT_ID} - ${params.VERSION} ]"
                configFileProvider(
                        [configFile(fileId: '${MOLGENIS_GIT_SETTINGS}', variable: 'MAVEN_SETTINGS')]) {
                    sh "mvn -s ${MAVEN_SETTINGS} release:perform"
                }
            }
        }
    }
    post {
        // [ slackSend ]; has to be configured on the host, it is the "Slack Notification Plugin" that has to be installed
        success {
            notifySuccess()
        }
        failure {
            notifyFailed()
        }
    }
}
