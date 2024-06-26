pipeline {
    agent any
    tools {
        jdk 'MyJava'
        gradle 'Mygradle'
    }
    //clean workspace
    stages {
        stage('Cleanws') {
            steps {
                cleanWs()
            }
        }
        //checkout the code
        stage('checkout from scm') {
            steps {
                git branch: 'main', url: 'https://github.com/Kachi-Okorie/Java-Web-Build.git'
            }
        }
        //compile every code in our project
        stage('Gradle compile') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew compileJava'
            }
        }
        stage('Test Gradle') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew test'
            }
        }
        //check code and vulnerabilities
        stage('sonarqube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-jenkins') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                    timeout(time: 10, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "pipeline is aborted due to qualitygate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage('build Gradle') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew build'
            }
        }
       
            stage('build and push to docker Hub') {
                steps {
                    script {
                        // withCredentials([usernameColonPassword(credentialsId: 'docker', variable: 'docker_password')])
                        withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'docker_password', usernameVariable: 'docker_name')]) {
                            sh '''
                             docker build -t kachio/test1 .
                             docker login -u $docker_name -p $docker_password
                             docker push kachio/test1
                             '''
                        }
                    }
                }
            }
            stage('deploy to container') {
                steps {
                    script {
                        // withCredentials([usernameColonPassword(credentialsId: 'docker', variable: 'docker_password')])
                        withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'docker_password', usernameVariable: 'docker_name')]) {
                            sh 'docker run -d --name g1 -p 8082:8080 kachio/gradleproject:latest'
                        }
                    }
                }
            }
    }
}
