pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "lex1725/react-app" // Update to your own repo
        DOCKER_CREDENTIALS_ID = "dockerhub_login" // Ensure this matches the ID in Jenkins Credentials
    }
    
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'chmod +x gradlew'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/reactApp'
            }
        }
        
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    app = docker.build("${DOCKER_IMAGE}")
                    app.inside {
                        sh 'echo $(curl localhost:1233)'
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }

        stage('DeployToStaging') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh "docker pull ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    try {
                        sh "docker stop react-app"
                        sh "docker rm react-app"
                    } catch (err) {
                        echo "Caught error: $err"
                    }
                    sh "docker run --restart always --name react-app -p 1233:80 -d ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                }
            }
        }
        
        stage("Check HTTP Response") {
            steps {
                script {
                    final String url = "http://localhost:1233"
                    
                    final String response = sh(script: "curl -o /dev/null -s -w '%{http_code}\\n' $url", returnStdout: true).trim()
                    
                    if (response == "200") {
                        echo response
                        println "Successful Response Code" 
                    } else {
                        echo response
                        println "Error Response Code" 
                    }
                }
            }
        }
        
        stage('DeployToProduction') {
            when {
                branch 'main'
            }
            steps {
                input 'Does the staging environment look OK? Did you get a 200 response?'
                milestone(1)
                script {
                    sh "docker pull ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    try {
                        sh "docker stop react-app"
                        sh "docker rm react-app"
                    } catch (err) {
                        echo "Caught error: $err"
                    }
                    sh "docker run --restart always --name react-app -p 1233:80 -d ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}

