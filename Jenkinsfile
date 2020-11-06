def appName = "jweb"
def applicationProcessName = "deploy"
def environmentName = "acaternberg_DEV_basicTraining"
def cdconfiguration = "cb-cd"
def projectName = "Training_acaternberg"
def pipelineName = "deployPL"

pipeline {
    agent {
        kubernetes {
            label 'maven_build'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
  - name: maven
    image: maven:alpine
    command:
    - cat
    tty: true
"""
        }
    }
//    parameters{
//        //choice(choices: ['dev', 'prd', 'ist'], description: 'What environment ?', name: 'envtarget')
//        string(defaultValue: "foo", description: 'App Name ?', name: 'p1')
//        string(defaultValue: "bar", description: 'Component Name ?', name: 'p2')
//
//    }

    environment {
        runProc = 'false'
        runDeploy = 'false'
        deployArtifact = 'true'
        runPipe = "true"
    }
    stages {
        stage('Run maven build') {
            steps {
                container('maven'){
                    configFileProvider([configFile(fileId: 'globl-maven-settins', variable: 'MAVEN_SETTINGS_XML')]) {
                        sh "mvn -s $MAVEN_SETTINGS_XML deploy"
                        //sh "mvn -X -Dmaven.wagon.http.ssl.insecure=true -s $MAVEN_SETTINGS_XML deploy"
                    }
                }
            }
        }
      /*  stage('call cd proceedure'){
            when {
                environment name: 'runProc', value: 'true'
            }
            steps{
               step([$class: 'ElectricFlowRunProcedure',
                      configuration: "${configuration}",
                      projectName : "${projectName}",
                      procedureName : 'chkCreds',
                      procedureParameters : """{"procedure":{"procedureName":"chkCreds",
                      "parameters":[
                            {"actualParameterName":"p1","value":"${params.p1}"},
                            {"actualParameterName":"p2","value":"${params.p2}"}
                      ]}}"""
                ])
            }
        }*/
        //stage('new call'){
        //    steps{
        //        cloudBeesFlowCallRestApi body: '', configuration: 'CdConfiguration', envVarNameForResult: 'CALL_REST_API_CREATE_PROJECT_RESULT', httpMethod: 'POST', parameters: [[key: 'projectName', value: 'EC-TEST-Jenkins-1.00.00.02'], [key: 'description', value: 'Native Jenkins Test Project']], urlPath: '/projects'
        //       
        //    }
        //}

        //stage('Call a cd procedure'){
        //    steps{
        //        //cloudBeesFlowRunProcedure configuration: 'CdConfiguration', overrideCredential: [credentialId: 'CREDS_PARAM'], procedureName: 'TomcatCheckServer', procedureParameters: '{"procedure":{"procedureName":"TomcatCheckServer","parameters":[{"actualParameterName":"max_time","value":"10"},{"actualParameterName":"tomcat_config_name","value":"Tomcat configuration"}]}}', projectName: 'CloudBees'
        //        cloudBeesFlowRunProcedure configuration: 'CdConfiguration', procedureName: 'chkCreds', procedureParameters: '{"procedure":{"procedureName":"chkCreds","parameters":[{"actualParameterName":"p1","value":"${params.p1}"},{"actualParameterName":"p2","value":"${params.p2}"}]}}', projectName: 'Honey'
        //
        //
        //    }
        //}
        
        
        stage('Publish an artifact to CD'){
            steps{
                cloudBeesFlowPublishArtifact {
                     onfiguration cdconfiguration
                     repositoryName 'default'
                     artifactName 'de.caternberg:jweb'
                     artifactVersion "${env.BUILD_NUMBER}"
                     filePath 'target/jweb.war'
                }
            }
        }
      

        stage('Deploy Application') {
            when {
                environment name: 'runDeploy', value: 'true'
            }
            steps {

                echo ""
                /*cloudBeesFlowDeployApplication applicationName: 'honey',
                                              configuration: 'CdConfiguration',
                                              applicationProcessName: 'InstallHoney',
                                              environmentName: 'dev', projectName: 'nectar'
                */
                cloudBeesFlowDeployApplication applicationName: "${appName}",
                        applicationProcessName: "${applicationProcessName}",
                        configuration: "${cdconfiguration}",
                        deployParameters: '{"runProcess":{"applicationName":"honey","applicationProcessName":"InstallHoney","parameter":[' +
                                '{"actualParameterName":"JENKINS_BUILD_NUMBER","value":"${BUILD_NUMBER}"},' +
                                '{"actualParameterName":"JENKINS_BUILD_ID","value":"${BUILD_ID}"},' +
                                '{"actualParameterName":"JENKINS_BUILD_DISPLAY_NAME","value":"${BUILD_DISPLAY_NAME}"},' +
                                '{"actualParameterName":"JENKINS_BUILD_URL","value":"${BUILD_URL}"},' +
                                '{"actualParameterName":"JENKINS_JOB_NAME","value":"${JOB_NAME}"},' +
                                '{"actualParameterName":"JENKINS_JOB_BASE_NAME","value":"${JOB_BASE_NAME}"},' +
                                ']}}',
                        environmentName: "${environmentName}",
                        projectName: "${projectName}"

            }




        }
        stage("DeployArtifact") {
            when {
                environment name: 'deployArtifact', value: 'true'
            }

            steps {
                cloudBeesFlowCreateAndDeployAppFromJenkinsPackage configuration: "${cdconfiguration}", filePath: 'target/*.jar'
                cloudBeesFlowPublishArtifact artifactName: 'de.caternberg.example:maven-executable-jar', artifactVersion: '1.0', configuration: ${cdconfiguration}, filePath: 'target/*.jar', repositoryName: 'default'
            }
            post {
                success {
                    echo "DeployArtifact Successfully"
                    //Slack notification...
                    //Jira update
                }
                failure {
                    echo "DeployArtifact Failed"
                    //Slack notification....
                    //Jira update
                }
            }
        }

        stage('RunPipeline') {
            when {
                environment name: 'runPipe', value: 'true'
            }

            steps {
               cloudBeesFlowRunPipeline addParam: '{"pipeline":{"pipelineName":"${pipelineName}","parameters":[{"parameterName":"JENKINS_BUILD_URL","parameterValue":"${BUILD_URL}"}]}}', configuration: "${cdconfiguration}", pipelineName: "${pipelineName}", projectName: "${projectName}"
            }
        }
            
    }
}
