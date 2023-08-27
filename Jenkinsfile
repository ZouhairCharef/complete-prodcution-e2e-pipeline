pipeline{
    agent any

    tools{
        jdk 'java17'
        maven 'Maven3'
    }
    
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }
    }

    stages{
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/ZouhairCharef/complete-prodcution-e2e-pipeline'
            }
        }
    }
}