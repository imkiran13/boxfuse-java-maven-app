pipeline {
    agent any
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Specify the branch name to deploy')
        string(name: 'WAR_VERSION', defaultValue: '1.0', description: 'Specify the WAR file version')
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs() // Cleans the workspace before starting the pipeline
            }
        }
        stage("GIT checkout") {
            steps {
                script {
                    // Checkout the specified branch
                    def branchName = params.BRANCH_NAME
                    checkout([$class: 'GitSCM', branches: [[name: branchName]], userRemoteConfigs: [[url: 'https://github.com/imkiran13/boxfuse-java-maven-app.git']]])
                }
            }
        }
        stage("Testing") {
            steps {
                echo "Test Runs"
            }
        }
        stage("Build") {
            steps {
                script {
                    def warVersion = params.WAR_VERSION
                    sh "mvn clean package"
                    sh "mv target/hello-${warVersion}.war target/boxfuse-java-maven-app-${warVersion}.war"
                }
            }
        }
        stage("Deploy to Dev") {
            when {
                expression { params.BRANCH_NAME == 'dev' }
            }
            steps {
                deployToTomcat('65.2.161.227', 'admin', 'admin', 'http://65.2.161.227:8080/manager/text', "/boxfuse-java-maven-app-${params.WAR_VERSION}", 'Dev', params.WAR_VERSION)
            }
        }
        stage("Deploy to QA") {
            when {
                expression { params.BRANCH_NAME == 'qa' }
            }
            steps {
                deployToTomcat('15.207.16.116', 'admin', 'admin', 'http://15.207.16.116:8080/manager/text', "/boxfuse-java-maven-app-${params.WAR_VERSION}", 'QA', params.WAR_VERSION)
            }
        }
        stage("Deploy to Prod") {
            when {
                expression { params.BRANCH_NAME == 'main' }
            }
            steps {
                input(message: "Do you want to proceed to PROD?", ok: "Proceed") // Approval step

                // Deploy to PROD server after approval
                deployToTomcat('43.204.103.54', 'admin', 'admin', 'http://43.204.103.54:8080/manager/text', "/boxfuse-java-maven-app-${params.WAR_VERSION}", 'Prod', params.WAR_VERSION)

                // Send Slack notification
                slackSend(channel: 'cricket', message: "Deployment to PROD has been approved by manager with version ${params.WAR_VERSION}.")
            }
        }
    }
}

def deployToTomcat(tomcatIP, username, password, tomcatURL, contextPath, environment, warVersion) {
    def warFileName = "target/boxfuse-java-maven-app-${warVersion}.war"

    // Deploy the WAR file using curl
    sh """
        curl -v -u ${username}:${password} --upload-file ${warFileName} ${tomcatURL}/deploy?path=${contextPath}&update=true
    """
    echo "Deployment to ${environment} server with version ${warVersion} completed."
}
