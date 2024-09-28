pipeline {
    agent any

    tools {
        maven 'Maven_3'
    }
    
    stages {

        stage ("informing") {
            steps {
                slackSend channel: 'aug-2024-weekend-batch', message: 'pipeline execution is starting now..'
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: '4e106a4d-293e-49d2-ac7a-b7852703aad8', url: 'https://github.com/akannan1087/myAug2024WeekendRepo'
            }
        }
        
        stage ("Build") {
            steps {
                sh "mvn clean install -f MyWebApp/pom.xml"
            }
        }
        
        stage ("code coverage") {
            steps {
                jacoco()
            }
        }
        
        stage ("Code analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'c6e025d1-c44b-40a1-ac02-7911a9d17d48') {
                    sh "mvn sonar:sonar -f MyWebApp/pom.xml"
                    }
                }
            }
        }
        
        stage ("Nexus upload") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '77378c22-bf54-4bc7-b88e-0ad6cd6d53dd', groupId: 'com.gcp.as', nexusUrl: 'ec2-34-229-1-243.compute-1.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        
        stage ("DEV deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: '9efb6a41-bf73-49c8-9f17-0c05e56a9e22', path: '', url: 'http://ec2-18-212-80-124.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ("Slack notify") {
            steps {
                slackSend channel: 'aug-2024-weekend-batch', message: 'DEV deployment was done, please start your testing..'
            }
        }
        //CI ends

        //CD starts here..

        stage ("DEV approve") {
            steps {
                    echo "Taking approval from DEV Manager for QA Deployment"     
                    timeout(time: 1, unit: 'HOURS') {
                    input message: 'Do you approve QA Deployment?', submitter: 'admin'
                }    
            }
        }

        stage ("QA Deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: '9efb6a41-bf73-49c8-9f17-0c05e56a9e22', path: '', url: 'http://ec2-18-212-80-124.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }

        stage ("QA notify") {
            steps {
                slackSend channel: 'aug-2024-weekend-batch, qa-testing-team', message: 'QA deployment was done, please start your functional testing..'
            }
        }

        stage ("QA approve") {
            steps {
                    echo "Taking approval from QA Manager for PROD Deployment"     
                    timeout(time: 1, unit: 'HOURS') {
                    input message: 'Do you approve PROD Deployment?', submitter: 'admin'
                }    
            }
        }

        stage ("PROD Deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: '9efb6a41-bf73-49c8-9f17-0c05e56a9e22', path: '', url: 'http://ec2-18-212-80-124.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }

        stage ("final notify") {
            steps {
                slackSend channel: 'product-owners-teams,qa-managers', message: 'PROD deployment was done, please inform end customers to use the app..'
            }
        }
    }

    post {
        always {
            // Clean up workspace
            cleanWs()
        }
        success {
            // Notify success (you can add email or Slack notifications here)
                slackSend channel: 'aug-2024-weekend-batch', message: 'pipeline was successful....'
        }
        failure {
            // Notify failure
            echo "Build or deployment failed."
        }
    }
}
