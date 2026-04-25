pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
    }


    environment {
        registryCredential = 'ecr:ap-south-1:awscreds'
        imageName = "758497006478.dkr.ecr.ap-south-1.amazonaws.com/vprofile-app-image"
        vprofileRegistry = "https://758497006478.dkr.ecr.ap-south-1.amazonaws.com"
        cluster = "vprofile-cluster"
        service = "vproapp-service"
    }
  stages {
   
        stage('Fetch code') {
            steps {
               git branch: 'main', url: 'https://github.com/Swapnil2298/vprofile-app.git'
            }

        }


        stage('Build'){
            steps{
               sh 'mvn install -DskipTests'
            }
        }

        stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage("Sonar Code Analysis") {
            environment {
                scannerHome = tool 'sonar8.0'
            }
            steps {
              withSonarQubeEnv('sonarserver') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }

        stage('Build App Image') {
          steps {
       
            script {
                dockerImage = docker.build( imageName + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
          }
    
        }

        stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }

        stage('Remove Container Images'){
            steps{
                sh 'docker rmi -f $(docker images -a -q)'
            }
        }

        stage('Deploy to ECS'){
            steps{
                withAWS(credentials: 'awscreds', region: 'ap-south-1'){
                    sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                }
            }
        }

  }
}
