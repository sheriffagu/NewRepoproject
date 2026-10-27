pipeline {
    agent any
    tools {
        maven'maven3'
    }

    stages {
        stage('checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '45c083c7-73b4-4340-bbfc-fd9af0e191a7', url: 'https://github.com/sheriffagu/NewRepoproject']])
            }
        }
        stage('build') {
            steps {
                sh " mvn clean install -f MyWebApp/pom.xml"
            }
        }
        stage('code coverage') {
            steps {
                jacoco()
            }
        }
        stage('code scanning') {
            steps {
                withSonarQubeEnv(installationName: 'sonarQube',credentialsId: 'a26c06be-9ee0-45ee-81cb-2a6ef480b48a') {
                    sh'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -f MyWebApp/pom.xml'
                 }
            }
        }
        stage('SAST SCANNING') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('binary storage') {
            steps {
               nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '44c66aa2-affc-447c-b96b-c51ff5b016ef', groupId: 'com.dept.app', nexusUrl: 'ec2-77-113-208-163.us-east-2.compute.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT' 
            }
        }
        stage('deploy to container') {
            steps {
               deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: '61fd6ad2-3d74-46e8-98ae-7dfc704277f3', path: '', url: 'http://ec2-3-23-102-89.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war' 
            }
        }
        stage('slack notify') {
            steps {
                echo " send message to manager"
                slackSend channel: 'devops', message: 'deployment complete'
            }
        }
        stage('QA approval') {
            steps {
                echo 'waiting for manager approval'
                timeout(time: 7, unit: 'DAYS'){
                    input message:"do you approve?",submitter: 'admin'
                }
            }
        }
        stage('deploy to QA') {
            steps {
               deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: '61fd6ad2-3d74-46e8-98ae-7dfc704277f3', path: '', url: 'http://ec2-3-23-102-89.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war' 
            }
        }       
    }
}
