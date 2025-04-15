pipeline {
    agent any
    environment {
        DOKCER_IMAGE = 'shudhoo/sampleapp'
        DOCKER_TAG = 'latest'
    }
    stages {
        stage('Validate Stage') {
            steps {
                git 'https://github.com/Shudhoo/samplejavaapp.git'
                sh '/opt/maven/bin/mvn validate'
            }
        }
        stage('Compile Stage') {
            steps {
                sh '/opt/maven/bin/mvn compile'
            }
        }
        stage('Code Review') {
            steps {
                sh '/opt/maven/bin/mvn -P metrics pmd:pmd'
            }
            post {
                success {
                    recordIssues sourceCodeRetention: 'LAST_BUILD', tools: [pmdParser(pattern: '**/pmd.xml')]
                }
            }
        }
        stage ('Unit Testing'){
            steps {
                sh '/opt/maven/bin/mvn test'
            }
            post {
                success {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage ('Code Coverage') {
            steps {
                sh '/opt/maven/bin/mvn verify'
            }
            post {
                success {
                    jacoco buildOverBuild: true, changeBuildStatus: true, runAlways: true, skipCopyOfSrcFiles: true
                }
            }
        }
        stage ('Package') {
            steps {
                sh '/opt/maven/bin/mvn package'
            }
        }
        stage ('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOKCER_IMAGE}:${DOCKER_TAG} .'
            }
        }
        stage ('Push Docker Image') {
            steps {
                script {
            withDockerRegistry(credentialsId: 'dockerhub-creds', url: 'https://index.docker.io/v1/') {
                def img = docker.image("${DOKCER_IMAGE}:${DOCKER_TAG}")
                img.push()
            }
                }
        }
        }
    }
}

