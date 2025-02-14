pipeline {
    agent any
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Specify the branch name to deploy')
    }
    stages {
        stage("GIT checkout") {
            steps {
                script {
                    // Checkout the specified branch
                    def branchName = params.BRANCH_NAME
                    checkout([$class: 'GitSCM', branches: [[name: branchName]], userRemoteConfigs: [[url: 'https://github.com/imkiran13/sample-java-maven-app.git']]])
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
                sh "mvn clean package"
                sh "mv target/hello-1.0.war target/sample-java-maven-app.war"
            }
        }
        stage("Deploy to Dev") {
            when {
                expression { params.BRANCH_NAME == 'dev' }
            }
            steps {
                deployToTomcat('13.233.140.75', 'tomcat', 'tomcat', 'http://13.233.140.75:8080/manager/text', '/sample-java-maven-app', 'Dev')
            }
        }
        stage("Deploy to QA") {
            when {
                expression { params.BRANCH_NAME == 'qa' }
            }
            steps {
                deployToTomcat('35.154.248.74', 'tomcat', 'tomcat', 'http://35.154.248.74:8080/manager/text', '/sample-java-maven-app', 'QA')
            }
        }
        stage("Deploy to Prod") {
            when {
                expression { params.BRANCH_NAME == 'master' }
            }
            steps {
                input(message: "Do you want to proceed to PROD?", ok: "Proceed") // Approval step

                // Deploy to PROD server after approval
                deployToTomcat('43.205.203.47', 'tomcat', 'tomcat', 'http://43.205.203.47:8080/manager/text', '/sample-java-maven-app', 'Prod')
                
                // Send Slack notification
                slackSend(channel: 'devopsrocks9am', message: "Deployment to PROD has been approved by manager.")
            }
        }
    }
}

def deployToTomcat(tomcatIP, username, password, tomcatURL, contextPath, environment) {
    def warFileName = 'target/sample-java-maven-app.war'

    // Deploy the WAR file using curl
    sh """
        curl -v -u ${username}:${password} --upload-file ${warFileName} ${tomcatURL}/deploy?path=${contextPath}&update=true
    """
    echo "Deployment to ${environment} server completed."
}
