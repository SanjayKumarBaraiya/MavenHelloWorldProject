pipeline {
    agent any
   // agent {
    //    any
        // Provide Agent name if you Jenkin Node
        //label <#NAME>
    //}
    parameters {
        string defaultValue: 'https://github.com/SanjayKumarBaraiya/', description: 'git repository url', name: 'GIT_URL', trim: true
        string defaultValue: 'MavenHelloWorldProject', description: 'git repository project name', name: 'GIT_PROJECT_NAME', trim: true
        string defaultValue: 'master', description: 'git repository project branch name', name: 'BRANCH_NAME', trim: true
        activeChoice choiceType: 'PT_MULTI_SELECT', description: 'security scan type list', filterLength: 1, filterable: false, name: 'SCAN_TYPE', randomName: 'choice-parameter-8495413156634390', script: groovyScript(fallbackScript: [classpath: [], oldScript: '', sandbox: false, script: ''], script: [classpath: [], oldScript: '', sandbox: true, script: 'return [\'OSS\',\'DAST\',\'SAST\']'])
        reactiveChoice choiceType: 'PT_MULTI_SELECT', filterLength: 1, filterable: false, name: 'DAST_SCAN_TOOL_NAME', randomName: 'choice-parameter-8495557759170690', referencedParameters: 'SCAN_TYPE', script: groovyScript(fallbackScript: [classpath: [], oldScript: '', sandbox: false, script: ''], script: [classpath: [], oldScript: '', sandbox: true, script: '''// Define the list of tools for DAST
                    def dastTools = [\'zap\', \'test\']
                    // Check if OSS is selected in SCAN_TYPE
                    if (SCAN_TYPE && SCAN_TYPE.contains(\'DAST\')) {
                        return dastTools
                    } else {
                        return []  // Return an empty list if DAST is not selected
                    }'''])
        reactiveChoice choiceType: 'PT_MULTI_SELECT', filterLength: 1, filterable: false, name: 'SAST_SCAN_TOOL_NAME', randomName: 'choice-parameter-8495686277490213', referencedParameters: 'SCAN_TYPE', script: groovyScript(fallbackScript: [classpath: [], oldScript: '', sandbox: false, script: ''], script: [classpath: [], oldScript: '', sandbox: true, script: '''// Define the list of tools for SAST
                    def sastTools = [\'sonarqube\', \'checkmarx\']

                    // Check if OSS is selected in SCAN_TYPE
                    if (SCAN_TYPE && SCAN_TYPE.contains(\'SAST\')) {
                        return sastTools
                    } else {
                        return []  // Return an empty list if SAST is not selected
                    }'''])
        reactiveChoice choiceType: 'PT_MULTI_SELECT', filterLength: 1, filterable: false, name: 'OSS_SCAN_TOOL_NAME', randomName: 'choice-parameter-8495836941896038', referencedParameters: 'SCAN_TYPE', script: groovyScript(fallbackScript: [classpath: [], oldScript: '', sandbox: false, script: ''], script: [classpath: [], oldScript: '', sandbox: true, script: '''// Define the list of tools for OSS
                    def ossTools = [\'dependency_check\', \'test\']

                    // Check if OSS is selected in SCAN_TYPE
                    if (SCAN_TYPE && SCAN_TYPE.contains(\'OSS\')) {
                        return ossTools
                    } else {
                        return []  // Return an empty list if OSS is not selected
                    }'''])
    }
    environment {
        GIT_URL="${params.GIT_URL}"
        BRANCH_NAME="${params.BRANCH_NAME}"
        GIT_PROJECT_NAME="${params.GIT_PROJECT_NAME}"
        SUCCESS_MAIL_BODY="email_body_contents-for_success_mail"
        FAILURE_MAIL_BODY="email_body_contents_for_failure_mail"
        SUCCESS_MAIL_SUB="email_subject_for_success"
        FAILURE_MAIL_SUB="email_subject_failure"
        RECIPIENTS_ID="recipients_list"
    }
    options {
        // Add other options as per your requirements
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '180', numToKeepStr: '500')
    }
    stages {
        stage('Checkout')
        {
            steps {
                //checkout scmGit(branches: [[name: "${BRANCH_NAME}"]], extensions: [], userRemoteConfigs: [[url: "${GIT_URL}"]])
                checkout([$class: 'GitSCM', branches: [[name: "${BRANCH_NAME}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '${BRANCH_NAME}'], [$class: 'CloneOption', depth: 1, noTags: true, reference: '', shallow: true, timeout: 60]], submoduleCfg: [], userRemoteConfigs: [[url: "${GIT_URL}/${GIT_PROJECT_NAME}"]]]) 
                }
            post {
                success {
                    echo "Checkout Stage Successfully Completed"
                }
                failure {
                    echo "Checkout Stage Failed"
                }
            }
        }
        stage('Build') {
            steps {
                echo "build"
            }

           post {
                success {
                    echo "Build Stage Successfully Completed"
                }
                failure {
                    echo "Build Stage Failed"
                }
            }
        }
        stage('Load-tools-properties') {
            steps{
            script {
                    // Read the properties file
                    def toolsProps = readProperties file: 'scan_tools.properties'
                    
                    // Access properties and set them as environment variables
                    env.SONAR_PROJECT_KEY = toolsProps['sonar.projectKey']
                    env.SONAR_PROJECT_NAME = toolsProps['sonar.projectName']
                    env.SONAR_PROJECT_VERSION = toolsProps['sonar.projectVersion']
                    env.SONAR_SOURCE_ENCODING = toolsProps['sonar.sourceEncoding']
                    env.SONAR_SCANNER_CLI_HOME = toolsProps['sonar.sonar_scanner_cli_home']
                     env.DEPENDENCY_PROJECT = toolsProps['dependency.project']
                    env.DEPENDENCY_FORMAT = toolsProps['dependency.format']
                    env.DEPENDENCY_CHECK_CLI_HOME = toolsProps['dependency.dependency_check_cli_home']
                }
            }
        }
        stage('Sonarqube-Analysis') {
            when {
                expression {
                    // Convert the parameter value to lowercase and check if it contains 'sonarqube'
                    def toolName = params.SAST_SCAN_TOOL_NAME?.toLowerCase()
                    return toolName?.contains('sonarqube')
                }
            }
            steps {
                
                echo "SonarQube Analysis Start"
                withSonarQubeEnv('Sonarqube') {
                    echo "${SONAR_SCANNER_CLI_HOME}"
                    sh '${SONAR_SCANNER_CLI_HOME}/bin/sonar-scanner -Dsonar.projectBaseDir=$WORKSPACE \
					-Dsonar.projectKey="$SONAR_PROJECT_KEY" \
					-Dsonar.projectName="$SONAR_PROJECT_NAME"  \
					-Dsonar.projectVersion="$SONAR_PROJECT_VERSION" \
                    -Dsonar.sourceEncoding="$SONAR_SOURCE_ENCODING" \
                    -Dsonar.coverage.jacoco.xmlReportPaths=${WORKSPACE}/build/reports/jacoco/test/jacocoTestReport.xml \
                    -Dsonar.exclusions=**/build/**,**/reports/** \
                    -Dsonar.coverage.exclusions=**/*Test*.java,**/agent/models/** \
                    -Dsonar.sources=. \
					-Dsonar.java.binaries=$WORKSPACE \
					-Dsonar.ws.timeout=8000'
                }
            }
            post {
                success {
                    script{
                        timeout(time: 10) { // Pipeline will be killed after a timeout
                            def qg = waitForQualityGate() // Get status from Sonar Server
                            if (qg.status != 'OK') {
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            }
                        }
                    }
                }
            }
        }
        stage('Dependency-Check Analysis') {
            when {
                expression {
                    // Convert the parameter value to lowercase and check if it contains 'sonarqube'
                    def toolName = params.OSS_SCAN_TOOL_NAME?.toLowerCase()
                    return toolName?.contains('dependency_check')
                }
            }
            steps {
                
                echo "Dependency-Check Analysis Start"
                sh '''${DEPENDENCY_CHECK_CLI_HOME}/bin/dependency-check.sh \
                    --project "$DEPENDENCY_PROJECT" \
                    --scan "$WORKSPACE" \
                    --noupdate \
                    --out $WORKSPACE/dependency-check-report \
                    --format $DEPENDENCY_FORMAT
                    '''
            }
            post {
                success {
                    echo "OWASP Depenendcy Check Scan is completed"
                    // Archive the OWASP Dependency-Check report
                    archiveArtifacts artifacts: "dependency-check-report/*.html", allowEmptyArchive: true
 
                // Publish the Dependency-Check results
                    dependencyCheckPublisher pattern: "dependency-check-report/*.html"
                }
            }
        }
        stage('CleanUP') {
            steps {
                echo "Clean UP"
                //cleanWs()
            }
            post {
                success {
                    echo "Clean Up Task ran  successfully"
                }
            }
        }
    }
    post {
        //Sending Email on failure or success status
        success {
            echo "success"
            //emailext body: "${SUCCESS_MAIL_BODY}", subject: "${SUCCESS_MAIL_SUB}", to: "${RECIPIENTS_ID}"
        }
        failure {
            echo "failed"
            //emailext attachLog: true, body: "${FAILURE_MAIL_BODY}", recipientProviders: [culprits()], subject: "${FAILURE_MAIL_SUB}", to: "${RECIPIENTS_ID}"
        }
    }
}
