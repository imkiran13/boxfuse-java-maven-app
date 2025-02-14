pipeline {
    agent any
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Specify the branch name to deploy')
        string(name: 'WAR_VERSION', defaultValue: '1.0', description: 'Specify the WAR file version')
    }
    stages {
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
                deployToTomcat('13.233.140.75', 'tomcat', 'tomcat', 'http://13.233.140.75:8080/manager/text', "/boxfuse-java-maven-app-${params.WAR_VERSION}", 'Dev', params.WAR_VERSION)
            }
        }
        stage("Deploy to QA") {
            when {
                expression { params.BRANCH_NAME == 'qa' }
            }
            steps {
                deployToTomcat('35.154.248.74', 'tomcat', 'tomcat', 'http://35.154.248.74:8080/manager/text', "/boxfuse-java-maven-app-${params.WAR_VERSION}", 'QA', params.WAR_VERSION)
            }
        }
        stage("Deploy to Prod") {
            when {
                expression { params.BRANCH_NAME == 'master' }
            }
            steps {
                input(message: "Do you want to proceed to PROD?", ok: "Proceed") // Approval step

                // Deploy to PROD server after approval
                deployToTomcat('43.205.203.47', 'tomcat', 'tomcat', 'http://43.205.203.47:8080/manager/text', "/boxfuse-java-maven-app-${params.WAR_VERSION}", 'Prod', params.WAR_VERSION)

                // Send Slack notification
                slackSend(channel: 'devopsrocks9am', message: "Deployment to PROD has been approved by manager with version ${params.WAR_VERSION}.")
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
