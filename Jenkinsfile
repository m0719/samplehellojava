pipeline {
    agent any // Executes on any available executor (including the Jenkins master)

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main',
                credentialsId: 'gitdev',
                url: 'https://github.com/m0719/samplehellojava.git'
            }
        }

        stage('Maven Build') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Code Review') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    script {
                        def mavenHome = tool name: "mavenbuild", type: "maven"
                        def mavenCMD = "${mavenHome}\\bin\\mvn" // Using double backslashes in the path for Windows

                        bat "${mavenCMD} sonar:sonar"
                    }
                }
            }
        }

        stage('Artifact Upload') {
            steps {
                nexusArtifactUploader(
                    artifacts: [
                        [
                            artifactId: 'scribe-java-webapp',
                            classifier: '',
                            file: '\\target\\scribe-java-webapp.war',
                            type: 'war'
                        ]
                    ],
                    credentialsId: 'sonanexusnew',
                    groupId: 'org.helloworld121',
                    nexusUrl: '65.0.78.57:8082',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'maniit-repo',
                    version: '1.0-SNAPSHOT'
                )
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFileName = 'scribe-java-webapp-1.0-20231201.101749-1.war'
                    def warPath = "target\\${warFileName}"

                    // Download the WAR file from Nexus using PowerShell
                    bat "powershell -Command \"Invoke-WebRequest -Uri 'http://65.0.78.57:8082/repository/maniit-repo/org/helloworld121/scribe-java-webapp/1.0-SNAPSHOT/${warFileName}' -OutFile '${warPath}'\""

                    // Deploy the WAR file to Tomcat
                    def username = "admin" // Tomcat manager username
                    def password = "admin" // Tomcat manager password

                    // Use curl for Windows or PowerShell's Invoke-RestMethod
                    bat "curl -T ${warPath} http://${username}:${password}@localhost:8085/manager/text/deploy?path=/scribe-java-webapp"
                }
            }
        }
    }
}
