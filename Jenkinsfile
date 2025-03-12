pipeline {
    environment {
        IMAGEN = "kiko4/flask-app"
        USUARIO = 'USER_DOCKERHUB'
    }
    agent none
    stages {
        stage("test the project") {
            agent {
                docker {
                image "python:3"
                args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch:'master',url:'https://github.com/K1K04/pruebaicdc.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r app/requirements.txt'
                    }
                }
                stage('Test')
                {
                    steps {
                        sh 'cd app && pytest test_app.py'
                    }
                }
            }
        }
        stage('Creación de la imagen') {
            agent any
            stages {
                stage('Build') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:$BUILD_NUMBER"
                        }
                    }
                }
                stage('Deploy') {
                    steps {
                        script {
                            docker.withRegistry( '', USUARIO ) {
                                newApp.push()
                                newApp.push("latest")
                            }
                        }
                    }
                }
                stage('Clean Up') {
                    steps {
                        sh "docker rmi $IMAGEN:$BUILD_NUMBER"
                        }
                }
            }
        }
        stage ('Despliegue') {
            agent any
            stages {
                stage ('Despliegue flaskapp'){
                    steps{
                        sshagent(credentials : ['VPS_SSH']) {
                        sh 'ssh -p 4444-o StrictHostKeyChecking=no debian@popeye.kiko4da.fun "pruebaicdc && git pull && docker-compose down -v && docker pull kiko4/flask-app:latest && docker-compose up -d"'
                        }
                    }
                }
            }
        }        
    }    
    post {
        always {
        mail to: 'kiko4da4@gmail.com',
        subject: "Estado del pipeline: ${currentBuild.fullDisplayName}",
        body: "El despliegue ${env.BUILD_URL} ha tenido como resultado: ${currentBuild.result}"
        }
    }
}
