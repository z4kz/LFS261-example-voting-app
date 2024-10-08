pipeline {
    agent none

    stages {
        stage("build") {
            agent {
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }

        stage("test") {
            agent {
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Running Unit Tests on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }

        stage("package") {
            when {
                branch 'master'
            }
            agent {
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('docker-package') {
            agent any

            when {
                branch 'master'
            }

            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/',
                    'dockerlogin') { def workerImage =
                        docker.build("zacts/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("latest")
                    }
                }
            }

            stage('deploy to dev') {
                agent any

                when {
                    branch 'master'
                }

                steps {
                    echo 'Deploy instavote app with docker compose'
                    sh 'docker compose up -d'
                }
            }
        }
    }

    post {
        always {
            echo 'Building multibranch pipeline for worker is completed..'
        }
    }
}
