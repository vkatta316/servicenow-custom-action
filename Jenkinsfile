pipeline {
    agent any
    
    tools {
        maven 'rama01Maven'
    }

    environment {
        SCANNER_HOME = tool 'sonarScanner'
        DOCKER_IMAGE_NAME = "ramadevops.jfrog.io/docker-local/hello-frog:1.0.0"
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
                sleep(5);
            }
            post {
                always {
                    junit "**/target/surefire-reports/*.xml"
                }
            }
        }

        stage('Sonar Summaries') {
            steps {
                withSonarQubeEnv(installationName: 'ramasonarcloud'){
                    sh 'mvn clean verify sonar:sonar \
                        -Pcoverage \
                        -Dsonar.token=1926d793f92181dd0ac406335d4d4bea392a3023 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.organization=pramaraju96 \
                        -Dsonar.projectKey=pramaraju96_ServiceNow-DevOps-Change-Sample'
                }
            }       
        }

        stage('Upload to JFrog') {
            steps {
                sh 'mvn package'
                sh "ls -ltr target/"
                echo "Uploading the artifacts to JFrog"
            }
            post{
                always{
                    script{
                        def server = Artifactory.server 'ramadevops'
                        def uploadSpec = """{
                        "files": [{
                        "pattern": "target/devops-e2e-1.40.0-SNAPSHOT.jar",
                        "target": "devops-jfrog-local-repo/"
                        }]
                        }"""

                        def buildInfo = server.upload(uploadSpec) 
                        server.publishBuildInfo buildInfo
                        sleep(5);
                    }
                }
            }  
        }

        stage('Download from JFrog') {
            steps {
                echo "Download the JFrog artifacts"
                echo "/var/jenkins_home/workspace/rama folder/CD Pipeline/${env.BUILD_NUMBER}/"
            }
            post{
                always{
                    script{
                        def server = Artifactory.server 'ramadevops'
                        def downSpec= """{
                        "files": [{
                        "pattern": "devops-jfrog-local-repo/devops-e2e-1.40.0-SNAPSHOT.jar",
                        "target":"/var/jenkins_home/workspace/rama folder/CD Pipeline/${BUILD_NUMBER}/"
                        }]
                        }"""

                        def buildInfo = server.download(downSpec) 
                        server.publishBuildInfo buildInfo
                        sleep(5);
                    }
                }
            } 
        }

        stage('Change') {
            steps {
                snDevOpsChange(changeRequestDetails:"""
                    {
                    "attributes":
                    {
                        "short_description": "Test description",
                        "priority": "1",
                        "start_date": "2021-02-05 08:00:00",
                        "end_date": "2022-04-05 08:00:00",
                        "justification": "test justification",
                        "description": "test description",
                        "cab_required": true,
                        "comments": "This update for work notes is from jenkins file",
                        "work_notes": "test work notes"
                        },
                    "setCloseCode": true
                    }
                """)
            }
        }
    }
}
