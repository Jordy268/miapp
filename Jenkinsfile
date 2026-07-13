pipeline {

    agent any


    environment {

        REGISTRY = 'ghcr.io'

        IMAGE_NAME = 'natashajg/mi-app'

        IMAGE_TAG = "${REGISTRY}/${IMAGE_NAME}:latest"

    }


    stages {


        stage('Prepare') {

            steps {

                echo 'Preparando entorno...'

                bat 'docker --version'

                bat 'node --version'

                bat 'npm --version'

            }

        }



        stage('Install Dependencies') {

            steps {

                echo 'Instalando dependencias...'

                bat 'npm install'

            }

        }



        stage('Test') {

            steps {

                echo 'Ejecutando pruebas...'

                bat 'npm test'

            }

        }

        stage('Verificar Reporte') {

    steps {

        bat 'dir'

        bat 'dir test-results'

    }

}

        stage('JUnit Report') {

    steps {

        junit 'test-results/results.xml'

    }

}


        stage('Generate Tags') {

            steps {

                script {


                    def commit = bat(
                        script: '@git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()


                    def timestamp = new Date().format("yyyyMMdd-HHmm")


                    env.COMMIT_TAG = "${REGISTRY}/${IMAGE_NAME}:${commit}"

                    env.TIMESTAMP_TAG = "${REGISTRY}/${IMAGE_NAME}:${timestamp}"


                    echo "Tag Commit SHA: ${env.COMMIT_TAG}"

                    echo "Tag Timestamp: ${env.TIMESTAMP_TAG}"


                }

            }

        }




        stage('Build Docker Image') {

            steps {


                echo 'Construyendo imagen Docker...'


                bat "docker build -t ${IMAGE_TAG} ."


                bat "docker tag ${IMAGE_TAG} ${COMMIT_TAG}"


                bat "docker tag ${IMAGE_TAG} ${TIMESTAMP_TAG}"


            }

        }




        stage('Push Image') {


            steps {


                echo 'Publicando imagen en GitHub Container Registry...'


                withCredentials([

                    string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')

                ]) {



                    bat """

                    echo %GITHUB_TOKEN% | docker login ghcr.io -u Jordy268 --password-stdin


                    docker push ${IMAGE_TAG}


                    docker push ${COMMIT_TAG}


                    docker push ${TIMESTAMP_TAG}


                    """

                }

            }

        }




        stage('Verify') {


            steps {


                echo 'Imagen generada correctamente'


                bat "docker images"


            }

        }


    }




    post {



        success {


            echo 'Pipeline completado exitosamente'



           /* emailext(

                subject: "Pipeline exitoso: ${JOB_NAME} #${BUILD_NUMBER}",


                body: """

                El pipeline se ejecutó correctamente.


                Proyecto:

                ${JOB_NAME}


                Build:

                ${BUILD_NUMBER}


                Estado:

                SUCCESS



                Imagen latest:

                ${IMAGE_TAG}



                Imagen Commit SHA:

                ${COMMIT_TAG}



                Imagen Build Timestamp:

                ${TIMESTAMP_TAG}


                """,


                to: 'correo@gmail.com'


            )*/

        }





        failure {


            echo 'El pipeline fallo'



           /* emailext(

                subject: "Pipeline fallido: ${JOB_NAME} #${BUILD_NUMBER}",



                body: """

                El pipeline presentó errores.



                Proyecto:

                ${JOB_NAME}



                Build:

                ${BUILD_NUMBER}



                Estado:

                FAILURE



                Revisar consola de Jenkins para más detalles.


                """,



                to: 'correo@gmail.com'


            )*/


        }




        cleanup {


            echo 'Limpiando recursos Docker'


            bat 'docker image prune -f'


        }


    }


}