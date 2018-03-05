#!/usr/bin/groovy
import com.evobanco.Utils
import com.evobanco.Constants

def runPPCJenkinsfile() {

    def utils = new com.evobanco.Utils()

    def artifactorySnapshotsURL = 'https://digitalservices.evobanco.com/artifactory/libs-snapshot-local'
    def artifactoryReleasesURL = 'https://digitalservices.evobanco.com/artifactory/libs-release-local'
    def sonarQube = 'http://sonarqube:9000'
    def openshiftURL = 'https://openshift.grupoevo.corp:8443'
    def openshiftCredential = 'openshift'
    def registry = '172.20.253.34'
    def artifactoryCredential = 'artifactory-token'
    def jenkinsNamespace = 'cicd'
    def mavenCmd = 'mvn -U -B -s /opt/evo-maven-settings/evo-maven-settings.xml'
    def mavenProfile = ''
    def springProfile = ''
    def params
    def envLabel
    def branchName
    def branchNameHY
    def branchType
    def artifactoryRepoURL

    //Parallet project configuration (PPC) properties
    def branchPPC = 'master'
    def credentialsIdPPC = '4b18ea85-c50b-40f4-9a81-e89e44e20178' //credentials of the parallel configuration project
    def relativeTargetDirPPC = '/tmp/configs/PPC/'
    def isPPCJenkinsFile = false
    def isPPCJenkinsYaml = false
    def isPPCOpenshiftTemplate = false
    def isPPCApplicationDevProperties = false
    def isPPCApplicationUatProperties = false
    def isPPCApplicationProdProperties = false
    def jenkinsFilePathPPC = relativeTargetDirPPC + 'Jenkinsfile'
    def jenkinsYamlPathPPC = relativeTargetDirPPC + 'Jenkins.yml'
    def openshiftTemplatePathPPC = relativeTargetDirPPC + 'kube/template.yaml'
    def applicationDevPropertiesPathPPC = relativeTargetDirPPC + 'configuration_profiles/dev/application-dev.properties'
    def applicationUatPropertiesPathPPC = relativeTargetDirPPC + 'configuration_profiles/uat/application-uat.properties'
    def applicationProdPropertiesPathPPC = relativeTargetDirPPC + 'configuration_profiles/prod/application-prod.properties'
    def jenknsFilePipelinePPC


    def pom
    def projectURL
    def artifactId
    def groupId

    int maxOldBuildsToKeep = 0

    //Taurus parameters
    def taurus_test_base_path = 'src/test/resources/taurus'
    def acceptance_test_path = '/acceptance_test/'
    def performance_test_path = '/'
    def smoke_test_path = '/smoke_test/'
    def security_test_path = '/security_test/'


    def openshift_route_hostname = 'demo-pru-srv-develop.svcsdev.grupoevo.corp'


    echo "BEGIN PARALLEL PROJECT CONFIGURATION (PPC)"


    node {

        //sleep 10
        checkout scm

        stage('Check Parallel project configuration elements (PPC)') {

            pom = readMavenPom()
            projectURL = pom.url
            artifactId = pom.artifactId
            groupId = utils.getProjectGroupId(pom.groupId, pom.parent.groupId, false)

            try {
                def parallelConfigurationProject = utils.getParallelConfigurationProjectURL(projectURL, artifactId)

                echo "Parallel configuration project ${parallelConfigurationProject} searching"

                retry (3)
                        {
                            checkout([$class                           : 'GitSCM',
                                      branches                         : [[name: branchPPC]],
                                      doGenerateSubmoduleConfigurations: false,
                                      extensions                       : [[$class           : 'RelativeTargetDirectory',
                                                                           relativeTargetDir: relativeTargetDirPPC]],
                                      submoduleCfg                     : [],
                                      userRemoteConfigs                : [[credentialsId: credentialsIdPPC,
                                                                           url          : parallelConfigurationProject]]])
                        }
                echo "Parallel configuration project ${parallelConfigurationProject} exits"

                // Jenkinsfile
                isPPCJenkinsFile = fileExists jenkinsFilePathPPC

                if (isPPCJenkinsFile) {
                    echo "Parallel configuration project Jenkinsfile found"
                } else {
                    echo "Parallel configuration project Jenkinsfile not found"
                }


                // Jenkins.yml
                isPPCJenkinsYaml = fileExists jenkinsYamlPathPPC

                if (isPPCJenkinsYaml) {
                    echo "Parallel configuration project Jenkins.yml found"
                } else {
                    echo "Parallel configuration project Jenkins.yml not found"
                }

                // Openshift template (template.yaml)
                isPPCOpenshiftTemplate = fileExists openshiftTemplatePathPPC

                if (isPPCOpenshiftTemplate) {
                    echo "Parallel configuration project Openshift template found"
                } else {
                    echo "Parallel configuration project Openshift template not found"
                }

                //application-dev.properties
                isPPCApplicationDevProperties = fileExists applicationDevPropertiesPathPPC

                if (isPPCApplicationDevProperties) {
                    echo "Parallel configuration project profile application-dev.properties found"
                } else {
                    echo "Parallel configuration project profile application-dev.properties not found"
                }

                //application-uat.properties
                isPPCApplicationUatProperties = fileExists applicationUatPropertiesPathPPC

                if (isPPCApplicationUatProperties) {
                    echo "Parallel configuration project profile application-uat.properties found"
                } else {
                    echo "Parallel configuration project profile application-uat.properties not found"
                }


                //application-prod.properties
                isPPCApplicationProdProperties = fileExists applicationProdPropertiesPathPPC

                if (isPPCApplicationProdProperties) {
                    echo "Parallel configuration project profile application-prod.properties found"
                } else {
                    echo "Parallel configuration project profile application-prod.properties not found"
                }


                echo "isPPCJenkinsFile : ${isPPCJenkinsFile}"
                echo "isPPCJenkinsYaml : ${isPPCJenkinsYaml}"
                echo "isPPCOpenshiftTemplate : ${isPPCOpenshiftTemplate}"
                echo "isPPCApplicationDevProperties : ${isPPCApplicationDevProperties}"
                echo "isPPCApplicationUatProperties : ${isPPCApplicationUatProperties}"
                echo "isPPCApplicationProdProperties : ${isPPCApplicationProdProperties}"

            }
            catch (exc) {
                echo 'There is an error on retrieving parallel project configuration'
                def exc_message = exc.message
                echo "${exc_message}"
            }
        }

        stage('Load pipeline configuration') {


            if (!isPPCJenkinsFile || !isPPCJenkinsYaml || !isPPCOpenshiftTemplate) {
                currentBuild.result = 'FAILURE'
                throw new hudson.AbortException('The parallel project configuration has not mandatory elements')
            }

            //Take parameters of the parallel project configuration (PPC)
            params = readYaml  file: jenkinsYamlPathPPC
            echo "Using Jenkins.yml from parallel project configuration (PPC)"

            //The template is provided by parallel project configuration (PPC)
            params.openshift.templatePath = relativeTargetDirPPC + params.openshift.templatePath
            echo "Template provided by parallel project configuration (PPC)"

            assert params.openshift.templatePath?.trim()

            echo "params.openshift.templatePath: ${params.openshift.templatePath}"


         }


        stage('Prepare') {
            echo "Prepare stage (PGC)"

            setDisplayName()

            echo "${currentBuild.displayName}"

            branchName = utils.getBranch()
            echo "We are on branch ${branchName}"
            branchType = utils.getBranchType(branchName)
            echo "This branch is a ${branchType} branch"
            branchNameHY = branchName.replace("/", "-").replace(".", "-").replace("_","-")
            echo "Branch name processed: ${branchName}"

            artifactoryRepoURL = (branchType == 'master' || branchType == 'release' || branchType == 'hotfix')  ? artifactoryReleasesURL : artifactorySnapshotsURL

            def isValidVersion = utils.isValidBranchPomVersion(pom.version, branchType)

            if (!isValidVersion) {
                //Sufix -SNAPSHOT is required for develop and feature branch types and is forbidden for release,hotfix and master branch types
                currentBuild.result = 'FAILURE'
                throw new hudson.AbortException('Version of artifact in pom is not allowed for this type of branch')
            }

        }


    } // end of node

    def deploy = 'Yes'

    if (branchType in params.confirmDeploy) {
        stage('Decide on Deploying') {
            deploy = input message: 'Waiting for user approval',
                    parameters: [choice(name: 'Continue and deploy?', choices: 'No\nYes', description: 'Choose "Yes" if you want to deploy this build')]
        }
    }

    if (deploy == 'Yes') {

        echo "Openshift route hostname: ${openshift_route_hostname}"

        def tasks = [:]




        if (branchType in params.testing.postdeploy.performanceTesting) {
            node('taurus') { //taurus
                checkout scm
                stage('Performance Tests') {
                    echo "Running performance tests..."

                    def test_files_location = taurus_test_base_path + performance_test_path + '**/*.yml'
                    echo "Searching performance tests with pattern: ${test_files_location}"

                    def files = findFiles(glob: test_files_location)

                    def testFilesNumber = files.length
                    echo "Performance test files found number: ${testFilesNumber}"

                    files.eachWithIndex { file, index ->

                        def isDirectory = files[index].directory

                        if (!isDirectory) {
                            echo "Executing performance test file number #${index}: ${files[index].path}"

                            echo "Setting taurus scenarios.scenario-default.default-address to ${openshift_route_hostname}"
                            echo "Setting taurus modules.gatling.java-opts to ${openshift_route_hostname}"

                            def bztScript = 'bzt -o scenarios.scenario-default.default-address=' + openshift_route_hostname + ' -o modules.gatling.java-opts=-Ddefault-address=' + openshift_route_hostname + ' ' + files[index].path  + ' -report --option=modules.console.disable=true'

                            echo "Executing script ${bztScript}"
                            sh "${bztScript}"
                        }

                    }
                }
            }
        } else {
            echo "Skipping performance tests..."
        }
    }



    stage('Notification') {
        echo "Sending Notifications..."

     /*
        if (currentBuild.result != 'SUCCESS') {
            slackSend channel: '#ops-room', color: '#FF0000', message: "The pipeline ${currentBuild.fullDisplayName} has failed."
            hipchatSend (color: 'RED', notify: true, message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            emailext (
                    subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
      <p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>""",
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
    */

    }

    stage('Remove old builds') {

        echo "params.maxOldBuildsToKeep: ${params.jenkins.maxOldBuildsToKeep}"
        echo "params.daysOldBuildsToKeep: ${params.jenkins.daysOldBuildsToKeep}"

        String maxOldBuildsToKeepParam = params.jenkins.maxOldBuildsToKeep
        String daysOldBuildsToKeepParam = params.jenkins.daysOldBuildsToKeep

        if (maxOldBuildsToKeepParam != null && maxOldBuildsToKeepParam.isInteger()) {
            maxOldBuildsToKeep = maxOldBuildsToKeepParam as Integer
        }

        if (daysOldBuildsToKeepParam != null && daysOldBuildsToKeepParam.isInteger()) {
            daysOldBuildsToKeep = daysOldBuildsToKeepParam as Integer
        }

        echo "maxOldBuildsToKeep: ${maxOldBuildsToKeep}"
        echo "daysOldBuildsToKeep: ${daysOldBuildsToKeep}"

        if (maxOldBuildsToKeep > 0 && daysOldBuildsToKeep > 0) {

            echo "Keeping last ${maxOldBuildsToKeep} builds"
            echo "Keeping builds for  ${daysOldBuildsToKeep} last days"

            properties([[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: "${daysOldBuildsToKeep}", numToKeepStr: "${maxOldBuildsToKeep}"]]]);

        } else if (maxOldBuildsToKeep > 0) {

            echo "Keeping last ${maxOldBuildsToKeep} builds"

            properties([[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: "${maxOldBuildsToKeep}"]]]);

        } else if (daysOldBuildsToKeep > 0) {

            echo "Keeping builds for  ${daysOldBuildsToKeep} last days"

            properties([[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: "${daysOldBuildsToKeep}", numToKeepStr: '']]]);

        } else {

            echo "Not removing old builds."

            properties([[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '']]]);

        }

    }


    echo "END PARALLEL PROJECT CONNFIGURATION (PPC)"

} //end of method

return this;