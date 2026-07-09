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
                withSonarQubeEnv ("SONAR_ID") {
                    sh """
                        npx sonar-scanner \
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

        // stage ("Docker Image") {
        //     steps {
        //         sh "docker compose up -d"
        //     }
        // }

        stage ("Docker Image push to ECR") {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_REGION', variable: 'AWS_REGION'),
                    string(credentialsId: 'AWS_ACCOUNT_ID', variable: 'AWS_ACCOUNT_ID')
                ]){
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.ap-south-1.amazonaws.com

                    IMAGES="
                    nikhil-shop-cart-service  
                    nikhil-shop-main-service   
                    nikhil-shop-order-service
                    nikhil-shop-product-service
                    nikhil-shop-user-service
                    "
                    for IMAGE in \$IMAGES
                    do 
                        TAG=\$(echo \$IMAGE | sed 's/nikhil-shop-//')
                        
                        docker tag \$IMAGE:latest \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/nikhil-shop-docker-images:\$TAG

                        echo "Pushing \$IMAGE..."
                        docker push \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/nikhil-shop-docker-images:\$TAG
                    done
                    """
                }
            }
        }
    }
}