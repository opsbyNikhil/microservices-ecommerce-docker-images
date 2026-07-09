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

        stage ("Upload dist into S3") {
            steps {
                withCredentials ([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "Jenkins-Dist"
                ]]) {
                        sh """
                            zip -r build-${BUILD_NUMBER}.zip dist
                            aws s3 cp build-${BUILD_NUMBER}.zip s3://amaz-s3-nikhil-shop"""
                        
                }
            }
        }
    }
}