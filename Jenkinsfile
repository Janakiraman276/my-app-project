pipeline {
    agent any

    stages {
        stage('SCM Checkout') {
            steps {
                git 'https://github.com/manojsb24/my-app.git'
            }
        }

        stage('Compile-Package') {
            steps {
                script {
                    def mvnHome = tool name: 'maven5', type: 'maven'
                    sh "${mvnHome}/bin/mvn clean package"
                    sh 'mv target/myweb*.war target/newapp.war'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def mvnHome = tool name: 'maven5', type: 'maven'
                    withSonarQubeEnv('sonar') {
                        sh "${mvnHome}/bin/mvn sonar:sonar"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t janakiraman276/myweb:0.0.2 .'
            }
        }

        stage('Docker Image Push') {
            steps {
                withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
                    sh "docker login -u janakiraman276 -p \$dockerPassword"
                }
                sh 'docker push janakiraman276/myweb:0.0.2'
            }
        }

        stage('Nexus Image Push') {
            steps {
                withCredentials([string(credentialsId: 'nexuspass', variable: 'nexusPassword')]) {
                    sh 'echo "${nexusPassword}" | docker login -u admin --password-stdin 15.206.170.67:8083'
                }
                sh 'docker tag janakiraman276/myweb:0.0.2 15.206.170.67:8083/jani:1.0.0'
                sh 'docker push 15.206.170.67:8083/jani:1.0.0'
            }
        }

        stage('Remove Previous Container') {
            steps {
                script {
                    try {
                        sh 'docker rm -f tomcattest'
                    } catch (error) {
                        // do nothing if there is an exception
                    }
                }
            }
        }

        stage('Docker Deployment') {
            steps {
                sh 'docker run -d -p 8090:8080 --name tomcattest janakiraman276/myweb:0.0.2'
            }
        }
    }

    post {
        always {
            script {
                emailext subject: "Build Notification - ${currentBuild.result}",
                body: "The build status is: ${currentBuild.result}",
                to: "janakiraman276@gmail.com",
                attachLog: true
            }
        }
    }
}
