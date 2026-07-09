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

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                    whoami
                    pwd
                    node -v
                    npm -v
                    ls -l node_modules/.bin/vite
                    ls -l node_modules/vite/bin/vite.js
                '''
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    whoami
                    pwd
                    ls -l node_modules/.bin/vite
                    ./node_modules/.bin/vite --version
                    npm run build
                '''
            }
        }
    }
   
}