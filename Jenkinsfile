pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M2_HOME" and add it to the path.
        maven "M2_HOME"
    }

    environment {
        SCANNER_HOME= tool 'sonar-scanner'

    }

    stages {
        stage('Git Checkout') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/NikhilLara/Healthcare.git'

                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
                    }
                }

        stage('Generate Test Reports') {
            steps {
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/Healthcare/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }

            stage('Trivy FS Scan') { 
                steps {
                    sh 'trivy fs --format table -o fs.html .' 
                } 
            }

            stage('SonarQube Analysis') { 
                steps {
                    withSonarQubeEnv('sonar-server') { 
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Healthcare -Dsonar.projectKey=Healthcare\
                                -Dsonar.java.binaries=target'''
                            } 
                        }
                    }
            stage('Build') {
                    steps {
                        sh "mvn package"
                        }
                    }
                
            stage('Publish Artifacts') {
                    steps {
                        withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'JAVA_HOME', maven: 'M2_HOME', mavenSettingsConfig: '', traceability: true) {
                            sh "mvn deploy"
                            }
                        }
                    }

            stage('Create Docker Image') {
                    steps {
                        sh 'docker build -t nikhillara1989/healthcare:1.0 .'
                        sh "docker buildx build --builder my-builder -t nikhillara1989/healthcare:1.0 ."
                        }
                    }

            stage('Docker-Login') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-login', passwordVariable: 'dockerpassword', usernameVariable: 'Dockerlogin')]) {
                            sh 'docker-login -u ${dockerhub-login} -p {dockerpassword}'
                            }
                        }
                    }

            stage('Docker Push') {
                    steps {
                        sh 'docker push nikhillara1989/healthcare:1.0'
                        }
                    }
            
       
}
}
