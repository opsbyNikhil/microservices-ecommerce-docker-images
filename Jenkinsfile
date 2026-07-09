pipeline {
    
    agent {label "Nikhil-Shop"}
    
    triggers {
        pollSCM("* * * * *")
    }

    stages {
        stage ("Git Checkout") {
            steps {
                git url: "https://github.com/opsbyNikhil/microservices-ecommerce-docker-images.git",
                    branch: "main"
            }
        }

        stage ("Install Dependencies") {
            steps {
                sh "npm install"
            }
        }

        stage('Fix Permissions') {
            steps {
                sh '''
                    chmod +x node_modules/.bin/vite
                    chmod +x node_modules/vite/bin/vite.js
                '''
            }
        }

        stage ("Build Application") {
            steps {
                sh "npm run build"
            }
        }

        // stage ("OWASP Dependency Check") {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan .',
        //                         odcInstallation: 'OWASP-DC'
        //     }
        // }

        // stage ("Publish OWASP Report") {
        //     steps {
        //         dependencyCheckPublisher pattern: "**/dependency-check-report.xml"
        //     }
        // }

        stage ("Sonar-scan") {
            steps {
                withCredentials ([string(credentialsId: "SONAR_ID", variable: "SONAR_TOKEN")]){
                withSonarQubeEnv ("SONAR") {
                    sh """
                        npx sonar:sonar \
                        -Dsonar.projectKey=opsbyNikhil_microservices-ecommerce-docker-images \
                        -Dsonar.organization=opsbynikhil \
                        -Dsonar.host.url=https://sonarcloud.io/ \
                        -Dsonar.login=${SONAR_TOKEN}
                        """
                }    
                }
            }
        }

        stage ("Upload dist into S3") {
            steps {
                sh "aws s3 sync dist/ s3://amaz-s3-nikhil-shop/build-${BUILD_NUMBER}/ --delete"       
            }
        }
    }
}