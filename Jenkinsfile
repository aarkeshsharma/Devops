pipeline {
    agent any
    
    tools{
        maven "MAVEN"
        jfrog "jfrog_cli"
    }
    
    environment{
        TOMCAT_USER= 'admin'
        TOMCAT_PASSWORD= 'admin'
        TOMCAT_PORT= '8500'
        TOMCAT_HOST = 'localhost'
        ARTIFACTORY_SERVER_ID='jfrog'
        ARTIFACTORY_REPO='maven-snapshots'
        WAR_FILE='target/web-application-war-file.war'
    }

    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/aarkeshsharma/Devops.git'
            }
        }
        stage('compile'){
            steps{
                bat "mvn compile"
            }
        }
        stage('test'){
            steps{
                bat "mvn test"
            }
        }
        stage('build'){
            steps{
                bat "mvn package"
            }
        }
        stage('sonarqube analysis'){
            steps{
                withSonarQubeEnv('sonarqube'){
                    bat "mvn clean package sonar:sonar"
                }
            }
        }

 stage('upload to artifactory') {
    steps {
        script {
           // def server = Artifactory.server("${env.ARTIFACTORY_SERVER_ID}")

            // Define the upload spec for JFrog CLI
            def uploadSpec = """{
                "files": [{
                    "pattern": "${env.WAR_FILE}",
                    "target": "${env.ARTIFACTORY_REPO}/"
                }]
            }"""

            // Write the upload spec to a file
            writeFile file: 'uploadSpec.json', text: uploadSpec

            // Use JFrog CLI to upload the WAR file to Artifactory
            bat """
            jf rt u --url="http://localhost:8082/artifactory/" ^
                      --user="admin" ^
                      --password="Password123" ^
                      --spec=uploadSpec.json
            """
        }
    }
}
        stage('deploy'){
            steps{
                script{
                    def warFile = 'target\\web-application-war-file.war'
                    def deployUrl = "http://${env.TOMCAT_HOST}:${env.TOMCAT_PORT}/manager/text/deploy?path=/web-application-war-file"
                    bat """curl --upload-file ${warFile} ^ --user ${env.TOMCAT_USER}:${env.TOMCAT_PASSWORD} ^ ${deployUrl}"""
                }
            }
        }
    }
   
}
