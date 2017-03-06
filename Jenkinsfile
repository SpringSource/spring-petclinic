pipeline {
    
    agent any

    stages {
        
        stage("build") {

            agent { label "build" }

            tools {
                maven "M3"
            }

            steps {
                sh "mvn clean package"
            }

            post {
                always {
                    archive "target/*.jar"
                    junit 'target/surefire-reports/*.xml'
                    stash includes: "**", name: "sources"
                    stash includes: "target/*.jar", name: "binary"
                }
            } 
        }
        
        stage("tests") {
            steps {

                parallel (
                    "static-analysis" : {
                        node("build") {
                            unstash "sources"
                            withMaven(maven:"M3") {
                                withSonarQubeEnv('sonarqube') {
                                    sh 'mvn sonar:sonar'
                                }
                            }
                        }
                    },
                    "performance-tests": {
                        node("test") {
                            echo "performance tests"
                            sh "ls -rtl"
                            sleep 20
                        }
                    }
                )
            }
        }

        stage("user-interaction") {
            agent none
            steps {
                input "Deploy ?"
            }
        }

        stage("deploy") {
            agent { label "ssh" }
            steps {
                unstash name:"binary"
                sh "ls -rtl target/"
            }
        }
    }
}
