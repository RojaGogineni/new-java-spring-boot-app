pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        S3_BUCKET = 'java-app-release'
        APP_NAME = 'rose-app'
        ENV_NAME = 'rose-env'
        SOLUTION_STACK = '64bit Amazon Linux 2023 v4.2.6 running Corretto 17'
        GRADLE_HOME = '/usr/local/gradle'
        M2 = '/usr/local/apache-maven/bin'
        M2_HOME = '/usr/local/apache-maven'
        PORT = '8080'
    }

    stages {
	    stage('Checkout') {
            steps {
                // Checkout your source code from GitHub
                git 'https://github.com/classes-101/new-java-spring-boot-app.git'
            }
        }
		
        stage('Build') {
            steps {
                script {
                    // Your build steps here
                    sh 'mvn install'
                    sh 'mvn package'
                }
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'jenkins-aws-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    // Use AWS credentials stored in Jenkins
                    //withCredentials([usernamePassword(credentialsId: 'jenkins-aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        // Assuming the JAR file is generated at build/libs/app.jar
                        def jarFile = 'target/database_service_project-0.0.2.jar'
                        def s3Key = "app-${env.BUILD_ID}.jar"
                        sh """
                            export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                            export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                            aws s3 cp ${jarFile} s3://${S3_BUCKET}/${s3Key}
                            
                            
                        """
                        echo 'ddddddddddddddddddtttttttttttyyyyyyyyyyyy'
                    }
                }
            }
        }

        stage('Deploy with Terraform') {
            steps {
                script {
                    // Ensure terraform is installed and initialized
                    sh 'terraform init'

                    // Use AWS credentials stored in Jenkins
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'jenkins-aws-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    // withCredentials([usernamePassword(credentialsId: 'jenkins-aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh """
                            export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                            export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                            terraform plan \
                            -var="app_version=v${env.BUILD_ID}" \
                            -var="s3_key=app-${env.BUILD_ID}.jar"
                            terraform apply -auto-approve \
                            -var="app_version=v${env.BUILD_ID}" \
                            -var="s3_key=app-${env.BUILD_ID}.jar"
                        """
                    }
                }
            }
        }
    }


}
