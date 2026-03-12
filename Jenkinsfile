pipeline {
    agent {
        node { label "maven" }
    }

    environment {
        QUAY = credentials('QUAY_USER')
    }

    parameters {
        booleanParam(name: "RUN_FRONTEND_TESTS", defaultValue: true)
    }

    stages {

        // Prepare environment (install Node.js once)
        stage('Prepare Environment') {
            steps {
                sh '''
                # Install Node.js once
                curl -fsSL https://rpm.nodesource.com/setup_18.x | bash -
                yum install -y nodejs
                node -v
                npm -v
                '''
            }
        }

        // Run backend and frontend tests in parallel
        stage('Run Tests') {
            parallel {

                stage('Backend Tests') {
                    steps {
                        sh 'node ./backend/test.js'
                    }
                }

                stage('Frontend Tests') {
                    when { 
                        expression { params.RUN_FRONTEND_TESTS } 
                    }
                    steps {
                        sh 'node ./frontend/test.js'
                    }
                }

            }
        }

        // Build and push container image to Quay.io
        stage('Build & Push Image') {
            steps {
                // Add Jib extension
                sh './mvnw quarkus:add-extension -Dextensions="container-image-jib"'

                // Build and push image
                sh '''
                ./mvnw package -DskipTests \
                  -Dquarkus.jib.base-jvm-imagee=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                  -Dquarkus.container-image.build=true \
                  -Dquarkus.container-image.registry=quay.io \
                  -Dquarkus.container-image.group=$QUAY_USR \
                  -Dquarkus.container-image.name=do400-deploying-lab \
                  -Dquarkus.container-image.username=$QUAY_USR \
                  -Dquarkus.container-image.password="$QUAY_PSW" \
                  -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                  -Dquarkus.container-imaaaaaaaaaaaage.additional-tags=latest \
                  -Dquarkus.container-image.push=true
                '''
            }
        }

        // Deploy stage with user input
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
