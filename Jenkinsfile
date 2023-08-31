pipeline{
    agent any

    tools{
        jdk 'java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "zouhaircharef"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = "${JENKINS_API_TOKEN}"
    }
    
    stages{

        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }

        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/ZouhairCharef/complete-prodcution-e2e-pipeline'
            }
        }

        stage("Build Application"){
            steps{
                sh "mvn clean package"
            }
        }

        stage("Test Application"){
            steps{
                sh "mvn test"
            }
        }

        stage("Sonarqube Analysis"){
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate"){
            steps{
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    // Try with TLSv1.2
                    def ret = sh(script: "curl -v -k --tlsv1.2 --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://192.168.50.10:8080/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'", returnStatus: true)
                    
                    // If TLSv1.2 fails, try with TLSv1.3
                    if (ret != 0) {
                        ret = sh(script: "curl -v -k --tlsv1.3 --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://192.168.50.10:8080/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'", returnStatus: true)
                    }

                    // If both attempts fail, throw an error
                    if (ret != 0) {
                        error("Failed to trigger CD Pipeline!")
                    }
                }
            }

        }

    }
}