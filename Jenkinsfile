pipeline {
    agent any

    environment {

        // Declares two pipeline-wide variables

        // 1- NAME is the fully-qualified Docker Hub repository
        // 2- SERVICE is the service's name ofc
        
        // Referencing SERVICE inside NAME avoids duplicating the service name

        SERVICE = 'gateway-service'
        NAME = "marcelovta/${env.SERVICE}"
    }
    
    stages {
        stage('Build') {

            // Compiles the project and produces the runnable JAR

            // Tests are intentionally skipped here
            // add a separate `Test` stage with `mvn test` if enforcing the pipeline test result is needed

            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Build & Push Image') {
            steps {
                withCredentials([usernamePassword(

                    // Injects the Docker Hub token stored in step 3 into the build environment
                    // The values are masked in all Jenkins logs

                    credentialsId: 'dockerhub-credential',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'TOKEN'

                )]) {

                    // Logs into dockerhub
                    sh 'echo "$TOKEN" | docker login -u "$USERNAME" --password-stdin'

                    
                    // docker buildx
                    // Creates a BuildKit builder that cross-complies images for both `linux/arm64` and `linux/amd64`
                    sh "docker buildx create --use \
                          --platform=linux/arm64,linux/amd64 \
                          --node multi-platform-builder-${env.SERVICE} \
                          --name multi-platform-builder-${env.SERVICE}"

                    // Dual tags
                    // :latest provides a stable reference
                    // :<BUILD_ID> pins a specific Jenkins build so any image can be traced back tp ots source commit
                    sh "docker buildx build \
                          --platform=linux/arm64,linux/amd64 \
                          --push \
                          --tag ${env.NAME}:latest \
                          --tag ${env.NAME}:${env.BUILD_ID} \
                          -f Dockerfile ."

                    // The builder instance is removed after the push to free up system resources
                    sh "docker buildx rm \
                          --force multi-platform-builder-${env.SERVICE}"
                
                }
            }
        }
    }
}