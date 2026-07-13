pipeline {
    agent any

    environment {
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'natashajg/mi-app'
        IMAGE_TAG = "${REGISTRY}/${IMAGE_NAME}:latest"
        
        // Inicializamos las variables vacías para que estén disponibles en todo el flujo
        COMMIT_TAG = ""
        TIMESTAMP_TAG = ""
    }

    stages {
        stage('Prepare') {
            steps {
                echo '⚙️ Preparando entorno...'
                // Validamos únicamente Docker, que es lo que sí está instalado y corriendo en Windows
                bat 'docker --version'
            }
        }

        stage('Generate Tags') {
          steps {
                script {
                    // Usamos la ruta exacta a git.exe entre comillas dobles y con barras invertidas dobles
                    def commit = bat(
                        script: '@"C:\\Program Files\\Git\\bin\\git.exe" rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    // Formato de fecha estable para el tag de compilación
                    def timestamp = new Date().format("yyyyMMdd-HHmm")

                    // Asignación explícita a las variables de entorno globales
                    env.COMMIT_TAG = "${REGISTRY}/${IMAGE_NAME}:${commit}"
                    env.TIMESTAMP_TAG = "${REGISTRY}/${IMAGE_NAME}:${timestamp}"

                    echo "🏷️ Tag Commit SHA: ${env.COMMIT_TAG}"
                    echo "🏷️ Tag Timestamp: ${env.TIMESTAMP_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Construyendo imagen empaquetada con Docker...'
                // Dockerfile ejecuta internamente el npm install y los entornos aislados de Node
                bat "docker build -t ${IMAGE_TAG} ."
                bat "docker tag ${IMAGE_TAG} ${env.COMMIT_TAG}"
                bat "docker tag ${IMAGE_TAG} ${env.TIMESTAMP_TAG}"
            }
        }

        stage('Push Image') {
            steps {
                echo '📤 Publicando imagen en GitHub Container Registry...'
                withCredentials([
                    string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')
                ]) {
                    // Login y push utilizando la sintaxis de variables para el Batch (cmd) de Windows
                    bat """
                    echo %GITHUB_TOKEN% | docker login ghcr.io -u Jordy268 --password-stdin
                    docker push ${IMAGE_TAG}
                    docker push ${env.COMMIT_TAG}
                    docker push ${env.TIMESTAMP_TAG}
                    """
                }
            }
        }

        stage('Verify') {
            steps {
                echo '✅ Listando imágenes locales en el servidor...'
                bat "docker images"
            }
        }
    }

    post {
        success {
            echo '🎉 ¡Pipeline completado exitosamente!'
        }
        failure {
            echo '❌ ¡El pipeline falló! Revisa los logs superiores en Jenkins.'
        }
        cleanup {
            echo '🧹 Limpiando recursos e imágenes intermedias de Docker...'
            bat 'docker image prune -f'
        }
    }
}