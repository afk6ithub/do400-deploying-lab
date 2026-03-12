pipeline {
    agent {
        node { label "maven" }
    }

    environment {
        QUAY = credentials('QUAY_USER')
    }

    parameters {
        // Optional parameter for frontend tests (not used here)
        booleanParam(name: "RUN_FRONTEND_TESTS", defaultValue: true)
    }

    stages {

        // Run Maven tests
        stage("Test") {
            steps {
                sh "./mvnw verify"
            }
        }

        // Build and push container image to Quay.io
        stage('Build & Push Image') {
            steps {
                // Add Jib extension to Quarkus project
                sh './mvnw quarkus:add-extension -Dextensions="container-image-jib"'

                // Build and push image
                sh '''
                ./mvnw package -DskipTests \
                  -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                  -Dquarkus.container-image.build=true \
                  -Dquarkus.container-image.registry=quay.io \
                  -Dquarkus.container-image.group=$QUAY_USR \
                  -Dquarkus.container-image.name=do400-deploying-lab \
                  -Dquarkus.container-image.username=$QUAY_USR \
                  -Dquarkus.container-image.password="$QUAY_PSW" \
                  -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                  -Dquarkus.container-image.additional-tags=latest \
                  -Dquarkus.container-image.push=true
                '''
            }
        }

        // Deploy stage with input
        stage('Deploy') {
            when {
                expression { env.GIT_BRANCH == 'origin/main' }
                beforeInput true
            }
            input {
                message 'Deploy the application?'
            }
            steps {
                echo 'Deploying...'
            }
        }

    }
}
