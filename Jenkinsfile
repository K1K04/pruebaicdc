pipeline {
    environment {
        IMAGEN = "alehache/flaskapp"
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
                        git branch:'main',url:'https://github.com/Alehache97/flaskapp.git'
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
                        sshagent(credentials : ['SSH_KEY']) {
                        sh 'ssh -o StrictHostKeyChecking=no debian@pibetis.macale.es "cd flaskapp && git pull && docker-compose down -v && docker pull alehache/flaskapp:latest && docker-compose up -d"'
                        }
                    }
                }
            }
        }        
    }    
    post {
        always {
        mail to: 'alejandroherrera140697@gmail.com',
        subject: "Estado del pipeline: ${currentBuild.fullDisplayName}",
        body: "El despliegue ${env.BUILD_URL} ha tenido como resultado: ${currentBuild.result}"
        }
    }
}
