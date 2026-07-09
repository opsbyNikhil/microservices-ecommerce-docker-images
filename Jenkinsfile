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

        stage ("OWASP Dependency Check") {
            steps {
                dependencyCheck additionlArguments: '--scan .',
                                odcinstallation: 'OWASP-DC'
            }
        }

        stage ("Publish OWASP Report") {
            steps {
                dependencyCheckPublisher pattern: "**/dependency-check-report.xml"
            }
        }
    }
}