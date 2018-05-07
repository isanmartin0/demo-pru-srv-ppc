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
    def isPPCApplicationProperties = false
    def isPPCApplicationDevProperties = false
    def isPPCApplicationUatProperties = false
    def isPPCApplicationProdProperties = false
    def jenkinsFilePathPPC = relativeTargetDirPPC + 'Jenkinsfile'
    def jenkinsYamlPathPPC = relativeTargetDirPPC + 'Jenkins.yml'
    def openshiftTemplatePathPPC = relativeTargetDirPPC + 'kube/template.yaml'
    def applicationPropertiesPathPPC = relativeTargetDirPPC + 'configuration_profiles/application.properties'
    def applicationDevPropertiesPathPPC = relativeTargetDirPPC + 'configuration_profiles/dev/application-dev.properties'
    def applicationUatPropertiesPathPPC = relativeTargetDirPPC + 'configuration_profiles/uat/application-uat.properties'
    def applicationProdPropertiesPathPPC = relativeTargetDirPPC + 'configuration_profiles/prod/application-prod.properties'
    def configMapsVolumePersistDefaultPath = '/usr/local/tomcat/conf'
    def jenknsFilePipelinePPC


    def pom
    def projectURL
    def artifactId
    def groupId

    int maxOldBuildsToKeep = 0

    //Taurus parameters
    def taurus_test_base_path = 'src/test/taurus'
    def acceptance_test_path = '/acceptance_test/'
    def performance_test_path = '/performance_test/'
    def smoke_test_path = '/smoke_test/'
    def security_test_path = '/security_test/'


    def openshift_route_hostname = ''
    def openshift_route_hostname_with_protocol = ''

    //AppDynamics parameters
    Boolean creationAppDynamicsConfigMap = false
    def isPPCAppDynamicsTemplate = false
    def appDynamicsTemplatePathPPC = relativeTargetDirPPC + 'appDynamics/appDynamics_template.yaml'
    def appDynamicsConfigMapsVolumePersistDefaultPath = '/opt/appdynamics/conf'



    echo "BEGIN PARALLEL PROJECT CONFIGURATION (PPC)"


    node('maven') {

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

                //application.properties
                isPPCApplicationProperties = fileExists applicationPropertiesPathPPC

                if (isPPCApplicationProperties) {
                    echo "Parallel configuration project profile application.properties found"
                } else {
                    echo "Parallel configuration project profile application.properties not found"
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

                //appDynamics template
                isPPCAppDynamicsTemplate = fileExists appDynamicsTemplatePathPPC

                if (isPPCAppDynamicsTemplate) {
                    echo "Parallel configuration project AppDynamics template found"
                } else {
                    echo "Parallel configuration project AppDynamics template not found"
                }


                echo "isPPCJenkinsFile : ${isPPCJenkinsFile}"
                echo "isPPCJenkinsYaml : ${isPPCJenkinsYaml}"
                echo "isPPCOpenshiftTemplate : ${isPPCOpenshiftTemplate}"
                echo "isPPCApplicationProperties : ${isPPCApplicationProperties}"
                echo "isPPCApplicationDevProperties : ${isPPCApplicationDevProperties}"
                echo "isPPCApplicationUatProperties : ${isPPCApplicationUatProperties}"
                echo "isPPCApplicationProdProperties : ${isPPCApplicationProdProperties}"
                echo "isPPCAppDynamicsTemplate : ${isPPCAppDynamicsTemplate}"

            }
            catch (exc) {
                echo 'There is an error on retrieving parallel project configuration'
                def exc_message = exc.message
                echo "${exc_message}"
            }
        }

        stage('Load pipeline configuration') {


            if (!isPPCJenkinsFile || !isPPCJenkinsYaml || !isPPCOpenshiftTemplate) {
                currentBuild.result = Constants.FAILURE_BUILD_RESULT
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

            //Detect creation of AppDynamics agent and existence of appdynamics template
            echo "params.appDynamics.creationAppDynamicsConfigMap: ${params.appDynamics.creationAppDynamicsConfigMap}"

            if (params.appDynamics.creationAppDynamicsConfigMap) {
                creationAppDynamicsConfigMap = params.appDynamics.creationAppDynamicsConfigMap.toBoolean()
            }

            if (creationAppDynamicsConfigMap  && !isPPCAppDynamicsTemplate) {
                currentBuild.result = Constants.FAILURE_BUILD_RESULT
                throw new hudson.AbortException('The parallel project configuration has not appdynamics template')
            }


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
                currentBuild.result = Constants.FAILURE_BUILD_RESULT
                throw new hudson.AbortException('Version of artifact in pom is not allowed for this type of branch')
            }

        }



        stage ('Prepare profiles') {
            switch (branchType) {
                case 'feature':
                    echo "Detect feature type branch"
                    envLabel="dev"
                    if (params.maven.profileFeature) {
                        mavenProfile = "-P${params.maven.profileFeature}"
                    }
                    if (params.spring.profileFeature) {
                        springProfile = params.spring.profileFeature
                    }
                    break
                case 'develop':
                    echo "Detect develop type branch"
                    envLabel="dev"
                    if (params.maven.profileDevelop) {
                        mavenProfile = "-P${params.maven.profileDevelop}"
                    }
                    if (params.spring.profileDevelop) {
                        springProfile = params.spring.profileDevelop
                    }
                    break
                case 'release':
                    echo "Detect release type branch"
                    envLabel="uat"
                    if (params.maven.profileRelease) {
                        mavenProfile = "-P${params.maven.profileRelease}"
                    }
                    if (params.spring.profileRelease) {
                        springProfile = params.spring.profileRelease
                    }
                    break
                case 'master':
                    echo "Detect master type branch"
                    envLabel="pro"
                    if (params.maven.profileMaster) {
                        mavenProfile = "-P${params.maven.profileMaster}"
                    }
                    if (params.spring.profileMaster) {
                        springProfile = params.spring.profileMaster
                    }
                    break
                case 'hotfix':
                    echo "Detect hotfix type branch"
                    envLabel="uat"
                    if (params.maven.profileHotfix) {
                        mavenProfile = "-P${params.maven.profileHotfix}"
                    }
                    if (params.spring.profileHotfix) {
                        springProfile = params.spring.profileHotfix
                    }
                    break
            }

            echo "Maven profile selected: ${mavenProfile}"
            echo "Spring profile selected: ${springProfile}"
        }


        if (branchName != 'master')
        {
            if (branchType in params.testing.predeploy.checkstyle) {
                stage('Checkstyle') {
                    echo "Running Checkstyle artifact..."
                    sh "${mavenCmd} checkstyle:check -DskipTests=true ${mavenProfile}"
                }
            } else {
                echo "Skipping Checkstyle..."
            }

            stage('Build') {
                echo "Building artifact..."
                sh "${mavenCmd} package -DskipTests=true -Dcheckstyle.skip=true ${mavenProfile}"
            }

            if (branchType in params.testing.predeploy.unitTesting) {
                stage('Unit Tests') {
                    echo "Running unit tests..."
                    sh "${mavenCmd} verify -Dcheckstyle.skip=true ${mavenProfile}"
                }
            } else {
                echo "Skipping unit tests..."
            }

            if (branchType in params.testing.predeploy.sonarQube) {
                stage('SonarQube') {
                    echo "Running SonarQube..."

                    def sonar_project_key = groupId + ":" + artifactId + "-" + branchNameHY
                    def sonar_project_name = artifactId + "-" + branchNameHY

                    echo "sonar_project_key: ${sonar_project_key}"
                    echo "sonar_project_name: ${sonar_project_name}"

                    sh "${mavenCmd} sonar:sonar -Dsonar.host.url=${sonarQube} ${mavenProfile} -Dsonar.projectKey=${sonar_project_key} -Dsonar.projectName=${sonar_project_name}"

                }
            } else {
                echo "Skipping Running SonarQube..."
            }

            stage('Artifact Deploy') {
                echo "Deploying artifact to Artifactory..."
                sh "${mavenCmd} deploy -DskipTests=true -Dcheckstyle.skip=true ${mavenProfile}"
            }
        } else {
            // Is the master branch

            stage('Check release version on Artifactory') {
                def artifactoryResponseCode = checkArtifactoryReleaseVersion {
                    artCredential = artifactoryCredential
                    repoUrl = artifactoryRepoURL
                }

                echo "Artifactory response status code: ${artifactoryResponseCode}"

                if (artifactoryResponseCode != null && Constants.HTTP_STATUS_CODE_OK.equals(artifactoryResponseCode)) {
                    echo "Artifact is avalaible for the pipeline on Artifactory"
                } else {
                    currentBuild.result = Constants.FAILURE_BUILD_RESULT
                    throw new hudson.AbortException('The artifact on Artifactory is not avalaible for the pipeline')
                }

            }

        }

        //Creation Config Maps
        Boolean useConfigurationProfilesFiles = false
        Boolean persistConfigurationProfilesFiles = false
        def configMapsVolumePersistPath = ''
        echo "params.spring.useConfigurationProfilesFiles: ${params.spring.useConfigurationProfilesFiles}"
        echo "params.spring.persistConfigurationProfilesFiles: ${params.spring.persistConfigurationProfilesFiles}"
        echo "params.spring.configMapsVolumePersistPath: ${params.spring.configMapsVolumePersistPath}"

        if (params.spring.useConfigurationProfilesFiles) {
            useConfigurationProfilesFiles = params.spring.useConfigurationProfilesFiles.toBoolean()
        }

        if (useConfigurationProfilesFiles) {
            if (params.spring.persistConfigurationProfilesFiles) {
                persistConfigurationProfilesFiles = params.spring.persistConfigurationProfilesFiles.toBoolean()
            }

            if (persistConfigurationProfilesFiles) {
                if (params.spring.configMapsVolumePersistPath) {
                    configMapsVolumePersistPath = params.spring.configMapsVolumePersistPath
                } else {
                    configMapsVolumePersistPath = configMapsVolumePersistDefaultPath
                }
            }
        }

        echo "useConfigurationProfilesFiles value: ${useConfigurationProfilesFiles}"
        echo "persistConfigurationProfilesFiles value: ${persistConfigurationProfilesFiles}"
        echo "configMapsVolumePersistPath value: ${configMapsVolumePersistPath}"





        stage('OpenShift Build') {
            echo "Building image on OpenShift..."

            openshiftCheckAndCreateProject {
                oseCredential = openshiftCredential
                cloudURL = openshiftURL
                environment = envLabel
                jenkinsNS = jenkinsNamespace
                artCredential = artifactoryCredential
                template = params.openshift.templatePath
                branchHY = branchNameHY
                branch_type = branchType
                dockerRegistry = registry
            }


            boolean configMapPersisted = false

            if (useConfigurationProfilesFiles) {
                def configMapCreated = openshiftConfigMapsCreation {
                    springProfileActive = springProfile
                    isPPCApplicationPropertiesOpenshift =  isPPCApplicationProperties
                    isPPCApplicationDevPropertiesOpenshift = isPPCApplicationDevProperties
                    isPPCApplicationUatPropertiesOpenshift = isPPCApplicationUatProperties
                    isPPCApplicationProdPropertiesOpenshift = isPPCApplicationProdProperties
                    applicationPropertiesPathPPCOpenshift = applicationPropertiesPathPPC
                    applicationDevPropertiesPathPPCOpenshift = applicationDevPropertiesPathPPC
                    applicationUatPropertiesPathPPCOpenshift = applicationUatPropertiesPathPPC
                    applicationProdPropertiesPathPPCOpenshift = applicationProdPropertiesPathPPC
                    branchHY = branchNameHY
                    branch_type = branchType
                }

                if (configMapCreated && persistConfigurationProfilesFiles) {
                    configMapPersisted = openshiftConfigMapsPersistence {
                        configMapsVolumePersistPathOpenshift = configMapsVolumePersistPath
                        branchHY = branchNameHY
                        branch_type = branchType
                    }
                }

            }

            //Creation AppDynamics config map


            echo "creationAppDynamicsConfigMap value: ${creationAppDynamicsConfigMap}"

            boolean appDynamicsConfigMapPersisted = false

            if (creationAppDynamicsConfigMap && (branchType == 'release' || branchType == 'hotfix' || branchType == 'master')) {
            //Only the release,hotfix and master branch types will have Appdynamics agent

                def appDynamicsConfigMapCreated = false
                def appDynamicsConfigMapsVolumePersistPath = ''
                echo "params.appDynamics.appDynamicsConfigMapsVolumePersistPath: ${params.appDynamics.appDynamicsConfigMapsVolumePersistPath}"

                //The appdynamics template is provided by parallel project configuration (PPC)
                params.appDynamics.appDynamicsTemplatePath = relativeTargetDirPPC + params.appDynamics.appDynamicsTemplatePath
                echo "Appdynamics template provided by parallel project configuration (PPC)"

                assert params.appDynamics.appDynamicsTemplatePath?.trim()

                if (params.appDynamics.appDynamicsConfigMapsVolumePersistPath) {
                    appDynamicsConfigMapsVolumePersistPath = params.appDynamics.appDynamicsConfigMapsVolumePersistPath
                } else {
                    appDynamicsConfigMapsVolumePersistPath = appDynamicsConfigMapsVolumePersistDefaultPath
                }



                echo "params.appDynamics.appDynamicsTemplatePath: ${params.appDynamics.appDynamicsTemplatePath}"
                echo "appDynamicsConfigMapsVolumePersistPath value: ${appDynamicsConfigMapsVolumePersistPath}"

                if (branchType == 'release' || branchType == 'hotfix') {

                    //Detect existence and show parameters of agent for UAT environment
                    if (!params.appDynamics.controllerHostnameUAT || !params.appDynamics.controllerPortUAT || !params.appDynamics.controllerSSLEnabledUAT
                        || !params.appDynamics.agentAccountNameUAT || !params.appDynamics.agentAccountAccessKeyUAT || !params.appDynamics.appDynamicsTemplatePath) {
                            currentBuild.result = Constants.FAILURE_BUILD_RESULT
                            throw new hudson.AbortException('There are mandatory AppDynamics parameters without value for UAT environment. The mandatory parameters are: controllerHostnameUAT, controllerPortUAT, controllerSSLEnabledUAT, agentAccountNameUAT, agentAccountAccessKeyUAT and appDynamicsTemplatePath')
                    }

                    echo "params.appDynamics.controllerHostnameUAT: ${params.appDynamics.controllerHostnameUAT}"
                    echo "params.appDynamics.controllerPortUAT: ${params.appDynamics.controllerPortUAT}"
                    echo "params.appDynamics.controllerSSLEnabledUAT: ${params.appDynamics.controllerSSLEnabledUAT}"
                    echo "params.appDynamics.agentApplicationNamePrefixUAT: ${params.appDynamics.agentApplicationNamePrefixUAT}"
                    echo "params.appDynamics.agentApplicationNameSufixUAT: ${params.appDynamics.agentApplicationNameSufixUAT}"
                    echo "params.appDynamics.agentTierNamePrefixUAT: ${params.appDynamics.agentTierNamePrefixUAT}"
                    echo "params.appDynamics.agentTierNameSufixUAT: ${params.appDynamics.agentTierNameSufixUAT}"
                    echo "params.appDynamics.agentAccountNameUAT: ${params.appDynamics.agentAccountNameUAT}"
                    echo "params.appDynamics.agentAccountAccessKeyUAT: ${params.appDynamics.agentAccountAccessKeyUAT}"


                    appDynamicsConfigMapCreated = openshiftAppDynamicsConfigMapsCreation {
                        appDynamicsTemplate = params.appDynamics.appDynamicsTemplatePath
                        appDynamics_controller_hostname = params.appDynamics.controllerHostnameUAT
                        appDynamics_controller_port = params.appDynamics.controllerPortUAT
                        appDynamics_controller_ssl_enabled = params.appDynamics.controllerSSLEnabledUAT
                        appDynamics_agent_application_name_prefix = params.appDynamics.agentApplicationNamePrefixUAT
                        appDynamics_agent_application_name_sufix = params.appDynamics.agentApplicationNameSufixUAT
                        appDynamics_agent_tier_name_prefix = params.appDynamics.agentTierNamePrefixUAT
                        appDynamics_agent_tier_name_sufix = params.appDynamics.agentTierNameSufixUAT
                        appDynamics_agent_account_name = params.appDynamics.agentAccountNameUAT
                        appDynamics_agent_account_access_key = params.appDynamics.agentAccountAccessKeyUAT
                        branchHY = branchNameHY
                        branch_type = branchType
                    }

                } else if (branchType == 'master') {

                    //Detect existence and show parameters of agent for PRO environment
                    if (!params.appDynamics.controllerHostnamePRO || !params.appDynamics.controllerPortPRO || !params.appDynamics.controllerSSLEnabledPRO
                        || !params.appDynamics.agentAccountNamePRO || !params.appDynamics.agentAccountAccessKeyPRO || !params.appDynamics.appDynamicsTemplatePath) {
                            currentBuild.result = Constants.FAILURE_BUILD_RESULT
                            throw new hudson.AbortException('There are mandatory AppDynamics parameters without value for PRO environment. The mandatory parameters are: controllerHostnamePRO, controllerPortPRO, controllerSSLEnabledPRO, agentAccountNamePRO, agentAccountAccessKeyPRO and appDynamicsTemplatePath')
                    }

                    echo "params.appDynamics.controllerHostnamePRO: ${params.appDynamics.controllerHostnamePRO}"
                    echo "params.appDynamics.controllerPortPRO: ${params.appDynamics.controllerPortPRO}"
                    echo "params.appDynamics.controllerSSLEnabledPRO: ${params.appDynamics.controllerSSLEnabledPRO}"
                    echo "params.appDynamics.agentApplicationNamePrefixPRO: ${params.appDynamics.agentApplicationNamePrefixUAT}"
                    echo "params.appDynamics.agentApplicationNameSufixPRO: ${params.appDynamics.agentApplicationNameSufixPRO}"
                    echo "params.appDynamics.agentTierNamePrefixPRO: ${params.appDynamics.agentTierNamePrefixPRO}"
                    echo "params.appDynamics.agentTierNameSufixPRO: ${params.appDynamics.agentTierNameSufixPRO}"
                    echo "params.appDynamics.agentAccountNamePRO: ${params.appDynamics.agentAccountNamePRO}"
                    echo "params.appDynamics.agentAccountAccessKeyPRO: ${params.appDynamics.agentAccountAccessKeyPRO}"


                    appDynamicsConfigMapCreated = openshiftAppDynamicsConfigMapsCreation {
                        appDynamicsTemplate = params.appDynamics.appDynamicsTemplatePath
                        appDynamics_controller_hostname = params.appDynamics.controllerHostnamePRO
                        appDynamics_controller_port = params.appDynamics.controllerPortPRO
                        appDynamics_controller_ssl_enabled = params.appDynamics.controllerSSLEnabledPRO
                        appDynamics_agent_application_name_prefix = params.appDynamics.agentApplicationNamePrefixPRO
                        appDynamics_agent_application_name_sufix = params.appDynamics.agentApplicationNameSufixPRO
                        appDynamics_agent_tier_name_prefix = params.appDynamics.agentTierNamePrefixPRO
                        appDynamics_agent_tier_name_sufix = params.appDynamics.agentTierNameSufixPRO
                        appDynamics_agent_account_name = params.appDynamics.agentAccountNamePRO
                        appDynamics_agent_account_access_key = params.appDynamics.agentAccountAccessKeyPRO
                        branchHY = branchNameHY
                        branch_type = branchType
                    }

                }


            }

            openshiftEnvironmentVariables {
                springProfileActive = springProfile
                branchHY = branchNameHY
                branch_type = branchType
                configMapPersistedOpenshift = configMapPersisted
                configMapsVolumePersistPathOpenshift = configMapsVolumePersistPath
            }

            openshiftBuildProject {
                artCredential = artifactoryCredential
                snapshotRepoUrl = artifactorySnapshotsURL
                repoUrl = artifactoryRepoURL
                javaOpts = ''
                springProfileActive = springProfile
                bc = params.openshift.buildConfigName
                is = params.openshift.imageStreamName
                branchHY = branchNameHY
                branch_type = branchType
            }


        }

    } // end of node

    def deploy = 'Yes'


    if (branchType in params.confirmDeploy) {
        try {
            stage('Decide on Deploying') {

                //Parameters timeout deploy answer

                Boolean timeoutConfirmDeploy = false
                int timeoutConfirmDeployTime = 0
                String timeoutConfirmDeployUnit = ''
                boolean isTimeoutConfirmDeployUnitValid = false

                echo "params.timeoutConfirmDeploy: ${params.timeoutConfirmDeploy}"

                if (params.timeoutConfirmDeploy != null) {
                    timeoutConfirmDeploy = params.timeoutConfirmDeploy.toBoolean()
                }

                if (timeoutConfirmDeploy) {
                    echo "params.timeoutConfirmDeployTime: ${params.timeoutConfirmDeployTime}"
                    echo "params.timeoutConfirmDeployUnit: ${params.timeoutConfirmDeployUnit}"

                    String timeoutConfirmDeployTimeParam = params.timeoutConfirmDeployTime
                    if (timeoutConfirmDeployTimeParam != null && timeoutConfirmDeployTimeParam.isInteger()) {
                        timeoutConfirmDeployTime = timeoutConfirmDeployTimeParam as Integer
                    }

                    if (params.timeoutConfirmDeployUnit != null && ("NANOSECONDS".equals(params.timeoutConfirmDeployUnit.toUpperCase())
                    || "MICROSECONDS".equals(params.timeoutConfirmDeployUnit.toUpperCase())
                    || "MILLISECONDS".equals(params.timeoutConfirmDeployUnit.toUpperCase())
                    || "SECONDS".equals(params.timeoutConfirmDeployUnit.toUpperCase())
                    || "MINUTES".equals(params.timeoutConfirmDeployUnit.toUpperCase())
                    || "HOURS".equals(params.timeoutConfirmDeployUnit.toUpperCase())
                    || "DAYS".equals(params.timeoutConfirmDeployUnit.toUpperCase()))) {
                        isTimeoutConfirmDeployUnitValid = true
                        timeoutConfirmDeployUnit = params.timeoutConfirmDeployUnit.toUpperCase()
                    }
                }

                echo "timeoutConfirmDeploy value: ${timeoutConfirmDeploy}"

                if (timeoutConfirmDeploy) {
                    echo "timeoutConfirmDeployTime value: ${timeoutConfirmDeployTime}"
                    echo "timeoutConfirmDeployUnit value: ${timeoutConfirmDeployUnit}"
                }


                if (timeoutConfirmDeploy && timeoutConfirmDeployTime > 0 && isTimeoutConfirmDeployUnitValid) {
                    //Wrap input with timeout
                    timeout(time:timeoutConfirmDeployTime, unit:"${timeoutConfirmDeployUnit}") {
                        deploy = input message: 'Waiting for user approval',
                            parameters: [choice(name: 'Continue and deploy?', choices: 'No\nYes', description: 'Choose "Yes" if you want to deploy this build')]
                    }
                } else {
                    //Input without timeout
                    deploy = input message: 'Waiting for user approval',
                        parameters: [choice(name: 'Continue and deploy?', choices: 'No\nYes', description: 'Choose "Yes" if you want to deploy this build')]

                }
            }
        } catch (err) {
            def user = err.getCauses()[0].getUser()
            if('SYSTEM'.equals(user.toString())) { //timeout
                currentBuild.result = "FAILED"
                throw new hudson.AbortException("Timeout on confirm deploy")
            }
        }
    }

    if (deploy == 'Yes') {
        node {
            checkout scm
            stage('OpenShift Deploy') {
                echo "Deploying on OpenShift..."

                openshift_route_hostname = openshiftDeployProject {
                    branchHY = branchNameHY
                    branch_type = branchType
                }

                openshift_route_hostname_with_protocol = utils.getRouteHostnameWithProtocol(openshift_route_hostname, false)

            }
        }


        echo "Openshift route hostname: ${openshift_route_hostname}"
        echo "Openshift route hostname (with protocol): ${openshift_route_hostname_with_protocol}"

        echo "params.jenkins.errorOnPostDeployTestsUnstableResult: ${params.jenkins.errorOnPostDeployTestsUnstableResult}"
        Boolean errorOnPostDeployTestsUnstableResult = false

        if (params.jenkins.errorOnPostDeployTestsUnstableResult != null) {
            errorOnPostDeployTestsUnstableResult = params.jenkins.errorOnPostDeployTestsUnstableResult.toBoolean()
        }

        echo "errorOnPostDeployTestsUnstableResult value: ${errorOnPostDeployTestsUnstableResult}"

        def tasks = [:]

        //Smoke tests
        if (branchType in params.testing.postdeploy.smokeTesting) {
            tasks["${Constants.SMOKE_TEST_TYPE}"] = {
                node('taurus') { //taurus
                    try {
                        stage("${Constants.SMOKE_TEST_TYPE} Tests") {
                            executePerformanceTest {
                                pts_taurus_test_base_path = taurus_test_base_path
                                pts_acceptance_test_path = smoke_test_path
                                pts_openshift_route_hostname_with_protocol = openshift_route_hostname_with_protocol
                                pts_performance_test_type = Constants.SMOKE_TEST_TYPE
                            }
                        }
                    } catch (exc) {
                        def exc_message = exc.message
                        echo "${exc_message}"
                        if (errorOnPostDeployTestsUnstableResult) {
                            currentBuild.result = Constants.UNSTABLE_BUILD_RESULT
                        } else {
                            //Failed status
                            currentBuild.result = Constants.FAILURE_BUILD_RESULT
                            throw new hudson.AbortException("The ${Constants.SMOKE_TEST_TYPE} tests stage has failures")
                        }
                    }
                }
            }
        } else {
            echo "Skipping ${Constants.SMOKE_TEST_TYPE} tests..."
        }

        //Acceptance tests
        if (branchType in params.testing.postdeploy.acceptanceTesting) {
            tasks["${Constants.ACCEPTANCE_TEST_TYPE}"] = {
                node('taurus') { //taurus
                    try {
                        stage("${Constants.ACCEPTANCE_TEST_TYPE} Tests") {
                            executePerformanceTest {
                                pts_taurus_test_base_path = taurus_test_base_path
                                pts_acceptance_test_path = acceptance_test_path
                                pts_openshift_route_hostname_with_protocol = openshift_route_hostname_with_protocol
                                pts_performance_test_type = Constants.ACCEPTANCE_TEST_TYPE
                            }
                        }
                    } catch (exc) {
                        def exc_message = exc.message
                        echo "${exc_message}"
                        if (errorOnPostDeployTestsUnstableResult) {
                            currentBuild.result = Constants.UNSTABLE_BUILD_RESULT
                        } else {
                            //Failed status
                            currentBuild.result = Constants.FAILURE_BUILD_RESULT
                            throw new hudson.AbortException("The ${Constants.ACCEPTANCE_TEST_TYPE} tests stage has failures")
                        }
                    }
                }
            }
        } else {
            echo "Skipping ${Constants.ACCEPTANCE_TEST_TYPE} tests..."
        }

        //Security tests
        if (branchType in params.testing.postdeploy.securityTesting) {
            tasks["${Constants.SECURITY_TEST_TYPE}"] = {
                node('taurus') { //taurus
                    try {
                        stage("${Constants.SECURITY_TEST_TYPE} Tests") {
                            executePerformanceTest {
                                pts_taurus_test_base_path = taurus_test_base_path
                                pts_acceptance_test_path = security_test_path
                                pts_openshift_route_hostname_with_protocol = openshift_route_hostname_with_protocol
                                pts_performance_test_type = Constants.SECURITY_TEST_TYPE
                            }
                        }
                    } catch (exc) {
                        def exc_message = exc.message
                        echo "${exc_message}"
                        if (errorOnPostDeployTestsUnstableResult) {
                            currentBuild.result = Constants.UNSTABLE_BUILD_RESULT
                        } else {
                            //Failed status
                            currentBuild.result = Constants.FAILURE_BUILD_RESULT
                            throw new hudson.AbortException("The ${Constants.SECURITY_TEST_TYPE} tests stage has failures")
                        }
                    }
                }
            }
        } else {
            echo "Skipping ${Constants.SECURITY_TEST_TYPE} tests..."
        }


        //Executing smoke, acceptance and security tests in parallel
        parallel tasks


        //Performance tests
        if (branchType in params.testing.postdeploy.performanceTesting) {
            node('taurus') { //taurus
                try {
                    stage("${Constants.PERFORMANCE_TEST_TYPE} Tests") {
                        executePerformanceTest {
                            pts_taurus_test_base_path = taurus_test_base_path
                            pts_acceptance_test_path = performance_test_path
                            pts_openshift_route_hostname_with_protocol = openshift_route_hostname_with_protocol
                            pts_performance_test_type = Constants.PERFORMANCE_TEST_TYPE
                        }
                    }
                } catch (exc) {
                    def exc_message = exc.message
                    echo "${exc_message}"
                    if (errorOnPostDeployTestsUnstableResult) {
                        currentBuild.result = Constants.UNSTABLE_BUILD_RESULT
                    } else {
                        //Failed status
                        currentBuild.result = Constants.FAILURE_BUILD_RESULT
                        throw new hudson.AbortException("The ${Constants.PERFORMANCE_TEST_TYPE} tests stage has failures")
                    }
                }
            }
        } else {
            echo "Skipping ${Constants.PERFORMANCE_TEST_TYPE} tests..."
        }


    } else {
        //User doesn't want to deploy
        //Failed status
        currentBuild.result = Constants.FAILURE_BUILD_RESULT
        throw new hudson.AbortException("The deploy on Openshift hasn't been confirmed")
    }



    stage('Notification') {
        echo "Sending Notifications..."

     /*
        if (!Constants.SUCCESS_BUILD_RESULT.equals(currentBuild.result)) {
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