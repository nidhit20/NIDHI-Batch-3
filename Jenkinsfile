// pipeline {
//     agent any

//     environment {
//         dockerRegistry = 'https://index.docker.io/v1/'
//         DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
//         frontendImage = 'fullstack-frontend'
//         backendImage = 'fullstack-backend'
//     }

//     stages {
//         stage('Check .env File') {
//             steps {
//                 script {
//                     def envFilePath = 'C:/ProgramData/Jenkins/.jenkins/workspace/user-auth pipeline/backend/.env'
//                     if (fileExists(envFilePath)) {
//                         echo "The .env file exists. Reading file..."
//                     } else {
//                         error "The .env file does not exist at path: ${envFilePath}"
//                     }
//                 }
//             }
//         }

//         stage('Load Environment Variables') {
//             steps {
//                 script {
//                     def envFilePath = 'C:/ProgramData/Jenkins/.jenkins/workspace/user-auth pipeline/backend/.env'
//                     if (fileExists(envFilePath)) {
//                         def envContent = readFile(envFilePath)
//                         def envVars = envContent.split('\n').collect { line ->
//                             if (line.trim()) {
//                                 def parts = line.split('=')
//                                 def key = parts[0].trim()
//                                 def value = parts[1].trim()
//                                 return "${key}=${value}"
//                             }
//                         }.findAll { it != null }
//                         withEnv(envVars) {
//                             echo "Environment variables loaded: ${envVars}"
//                         }
//                     } else {
//                         error "The .env file does not exist at path: ${envFilePath}"
//                     }
//                 }
//             }
//         }

//         stage('Build Backend') {
//             steps {
//                 script {
//                     def backendPath = 'backend'
//                     if (fileExists(backendPath)) {
//                         echo "Building backend image"
//                         bat "docker build -t ${backendImage}:latest ${backendPath}" // Build the image
//                         bat "docker tag ${backendImage} nidhit225/fullstack-backend:fullstack-backend" // Tag image
//                     } else {
//                         error "Backend directory not found"
//                     }
//                 }
//             }
//         }

//         stage('Build Frontend') {
//             steps {
//                 script {
//                     def frontendPath = 'frontend'
//                     if (fileExists(frontendPath)) {
//                         echo "Building frontend image"
//                         bat "docker build -t ${frontendImage}:latest ${frontendPath}" // Build the image
//                         bat "docker tag ${frontendImage} nidhit225/fullstack-frontend:fullstack-frontend" // Tag image
//                     } else {
//                         error "Frontend directory not found"
//                     }
//                 }
//             }
//         }

//         stage('Push Backend') {
//             steps {
//                 script {
//                     echo "Preparing to push backend image"
//                     docker.withRegistry(dockerRegistry, 'dockerhub-creds') {
//                         bat "docker push nidhit225/fullstack-backend:${backendImage}"
//                     }
//                 }
//             }
//         }

//         stage('Push Frontend') {
//             steps {
//                 script {
//                     echo "Preparing to push frontend image"
//                     docker.withRegistry(dockerRegistry, 'dockerhub-creds') {
//                         bat "docker push nidhit225/fullstack-frontend:${frontendImage}"
//                     }
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             echo "PIPELINE SUCCESS"
//         }
//         failure {
//             echo "PIPELINE FAILED"
//         }
//     }
// }
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        BACKEND_IMAGE_NAME = 'fullstack-backend'
        FRONTEND_IMAGE_NAME = 'fullstack-frontend'
        DOCKER_TAG = "latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/nidhit20/NIDHI-Batch-3.git'
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                script {
                    dockerImageBackend = docker.build("${env.BACKEND_IMAGE_NAME}:${env.DOCKER_TAG}", "./backend")
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                script {
                    dockerImageFrontend = docker.build("${env.FRONTEND_IMAGE_NAME}:${env.DOCKER_TAG}", "./client")
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${env.DOCKER_CREDENTIALS_ID}") {
                        echo "Logged in to Docker Hub"
                    }
                }
            }
        }

        stage('Push Backend Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${env.DOCKER_CREDENTIALS_ID}") {
                        dockerImageBackend.push()
                    }
                }
            }
        }

        stage('Push Frontend Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${env.DOCKER_CREDENTIALS_ID}") {
                        dockerImageFrontend.push()
                    }
                }
            }
        }

        stage('Post-build Cleanup') {
            steps {
                script {
                    // Clean up local Docker images to save space
                    if (isUnix()) {
                        sh "docker rmi ${env.BACKEND_IMAGE_NAME}:${env.DOCKER_TAG} || true"
                        sh "docker rmi ${env.FRONTEND_IMAGE_NAME}:${env.DOCKER_TAG} || true"
                    } else {
                        bat "docker rmi ${env.BACKEND_IMAGE_NAME}:${env.DOCKER_TAG} || ver > nul"
                        bat "docker rmi ${env.FRONTEND_IMAGE_NAME}:${env.DOCKER_TAG} || ver > nul"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Docker images for both backend and frontend successfully built and pushed to Docker Hub!'
        }
        failure {
            echo 'Build failed. Please check the logs for more details.'
        }
    }
}