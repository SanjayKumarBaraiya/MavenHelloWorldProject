pipeline {
    agent any
   // agent {
    //    any
        // Provide Agent name if you Jenkin Node
        //label <#NAME>
    //}
    environment {
        GIT_URL="https://github.com/SanjayKumarBaraiya/MavenHelloWorldProject.git"
        BRANCH_NAME="*/master"
        SONAR_SCANNER_CLI_HOME ="/home/devops/sonarqube/sonar-scanner-5.0.1.3006-linux"
    }
    stages {
	stage('ReadEnv')
        {
            steps {
                readProperties charset: '', file: 'jenkins.properties', text: ''
                
            }
        }
        stage('Checkout')
        {
            steps {
                checkout scmGit(branches: [[name: "${BRANCH_NAME}"]], extensions: [], userRemoteConfigs: [[url: "${GIT_URL}"]])
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
        stage('Sonarqube-Analysis') {
            steps {
                echo "SonarQube Analysis Start"
                withSonarQubeEnv('Sonarqube') {
                    echo "${SONAR_SCANNER_CLI_HOME}"
                    sh '${SONAR_SCANNER_CLI_HOME}/bin/sonar-scanner -Dsonar.projectBaseDir=${WORKSPACE} \
					-Dsonar.projectKey="Demo_DevOps" \
					-Dsonar.projectName="Demo_DevOps"  \
					-Dsonar.projectVersion=1.0 \
                    -Dsonar.coverage.jacoco.xmlReportPaths=${WORKSPACE}/build/reports/jacoco/test/jacocoTestReport.xml \
                    -Dsonar.exclusions=**/build/**,**/reports/** \
                    -Dsonar.coverage.exclusions=**/*Test*.java,**/agent/models/** \
                    -Dsonar.sources=. \
					-Dsonar.java.binaries=${WORKSPACE} \
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
        stage('CleanUP') {
            steps {
                echo "Clean UP"
                cleanWs()
            }
            post {
                success {
                    echo "Clean Up Task ran  successfully"
                }
            }
        }
    }
}
