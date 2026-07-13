pipeline {
    agent any

    environment {
        REGISTRY = 'ghcr.io'
        // Cambiado a tu usuario real de GitHub para evitar el error 403 Forbidden
        IMAGE_NAME = 'jordy268/mi-app' 
        IMAGE_TAG = "${REGISTRY}/${IMAGE_NAME}:latest"
    }

    stages {
        stage('Prepare') {
            steps {
                echo '⚙️ Preparando entorno...'
                bat 'docker --version'
            }
        }

        stage('Generate Tags') {
            steps {
                script {
                    // Obtenemos el SHA corto de manera segura y lo inyectamos directamente al mapa env
                    def commitSha = bat(
                        script: '@"C:\\Program Files\\Git\\bin\\git.exe" rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    def timestamp = new Date().format("yyyyMMdd-HHmm")

                    // Asignación correcta en Jenkins para evitar valores 'null'
                    env.COMMIT_TAG = "${REGISTRY}/${IMAGE_NAME}:${commitSha}"
                    env.TIMESTAMP_TAG = "${REGISTRY}/${IMAGE_NAME}:${timestamp}"

                    echo "🏷️ Tag Commit SHA definitivo: ${env.COMMIT_TAG}"
                    echo "🏷️ Tag Timestamp definitivo: ${env.TIMESTAMP_TAG}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Construyendo imagen empaquetada con Docker...'
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
                    // Usamos variables de entorno robustas y limpias de Jenkins
                    bat """
                    echo %GITHUB_TOKEN% | docker login ghcr.io -u jordy268 --password-stdin
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
            echo '🎉 ¡Pipeline completado exitosamente sin errores!'
        }
        failure {
            echo '❌ ¡El pipeline falló! Revisa los logs de los tags.'
        }
        cleanup {
            echo '🧹 Limpiando recursos e imágenes intermedias de Docker...'
            bat 'docker image prune -f'
        }
    }
}