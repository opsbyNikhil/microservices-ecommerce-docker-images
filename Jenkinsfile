pipeline {
    
    agent {label "Nikhil-Shop"}
    

    // parameters {
    //     booleanParam (
    //         name: 'SKIP_OWASP',
    //         defaultValue: true,
    //         description: "Skip OWASP Dependenct Check"
    //     )

    //     booleanParam (
    //         name: 'SKIP_OWASP_REPORT',
    //         defaultValue: true,
    //         description: "Skip OWASP Dependenct Check"
    //     )
        
    //     booleanParam (
    //         name: 'SKIP_DOCKER_COMPOSE',
    //         defaultValue: true,
    //         description: "Skip OWASP Dependenct Check"
    //     )


    // }
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
        //     when {
        //         expression {
        //             currentBuild.currentResult == null || currentBuild.currentResult == "SUCCESS"
        //         }
        //     }
        //     steps {
        //         dependencyCheck additionalArguments: '--scan .',
        //                         odcInstallation: 'OWASP-DC'
        //     }
        // }

        // stage ("Publish OWASP Report") {
        //     when {
        //         expression {
        //             currentBuild.currentResult == null || currentBuild.currentResult == "SUCCESS"
        //         }
        //     }
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

        stage ("Docker Image") {
            when {
                expression {
                    currentBuild.currentResult == null || currentBuild.currentResult == "SUCCESS"
                }
            }
            steps {
                sh "docker compose build"
            }
        }


        stage ("Trivy-Scan") {
            
            environment {
                SERVICES = "cart-service main-service  order-service product-service user-service"
            }

            steps {
                sh """
                    set +e
                    curl -sSL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/junit.tpl \
                    -o junit.tpl

                    for SERVICE in \$SERVICES
                    do

                        IMAGE=nikhil-shop-\$SERVICE

                        echo "===================================="
                        echo "Scanning \$IMAGE"
                        echo "===================================="

                        if ! docker image inspect \$IMAGE:latest >/dev/null 2>&1; then
                            echo "\$IMAGE does not exist. Skipping..."
                            continue
                        fi
                       
                    
                            # Generate readable xml report
                            trivy image \
                                --scanners vuln \
                                --severity HIGH,CRITICAL \
                                --format template \
                                --template "@junit.tpl" \
                                --exit-code 0 \
                                -o trivy-report-\$SERVICE.xml \
                                \$IMAGE:latest

                            # Generate readable text report
                            trivy image \
                                --scanners vuln \
                                --severity HIGH,CRITICAL \
                                --format table \
                                --exit-code 0 \
                                -o trivy-report-\$SERVICE.txt \
                                \$IMAGE:latest
                    done

                    echo "Generated Reports:"
                    ls -lh trivy-report-* || true
                """
            }
        }
        
        stage ("Docker Image push to ECR") {
            environment {
                SERVICES = "cart-service main-service  order-service product-service user-service"
            }
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_REGION', variable: 'AWS_REGION'),
                    string(credentialsId: 'AWS_ACCOUNT_ID', variable: 'AWS_ACCOUNT_ID')
                ]){
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.ap-south-1.amazonaws.com


                    for SERVICE in \$SERVICES
                    do 
                        IMAGE=nikhil-shop-\$SERVICE

                        echo "Tagging \$IMAGE eith build-${BUILD_NUMBER}"
                        docker tag \$IMAGE:latest \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/\$IMAGE:build-${BUILD_NUMBER}

                        echo "Pushing \$IMAGE:build-${BUILD_NUMBER}"
                        docker push \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/\$IMAGE:build-${BUILD_NUMBER}
                    done
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy-report-*.*', fingerprint: true
            junit allowEmptyResults: true, 
                  keepLongStdio: true,
                  testResults: 'trivy-report-*.xml'
        }
    }
}