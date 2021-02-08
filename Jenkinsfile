pipeline {
    environment {
        REGISTRY_URL = "http://harbor2.coe.com/"
        SONARQUBE_PROJECT_TITLE = "Mobile_Web_eCom_App"
        REGISTRY_NAME = "harbor2.coe.com/pipeline/mobile-web-ecommerce-app"
        KUBERNETESserver_URL = "https://lb.coe.com/k8s/clusters/c-v84ch"
        GIT_URL = "http://git.coe.com:3000/rakesh/Mobile_Web_eCom_App.git"
        SONARQUBE_URL = "http://10.0.100.16"
    }
    agent { 
        label 'master'  
    }
    stages {
        stage('Git checkout on jenkins master VM') {
            steps {
                git "${env.GIT_URL}"
                sh 'ls -al'
            }
        }
        stage('Code Quality Check via SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner';
                    withSonarQubeEnv("sonarqube") {
                        sh "${tool("sonar-scanner")}/bin/sonar-scanner \
                        -Dsonar.projectKey=$SONARQUBE_PROJECT_TITLE \
                        -Dsonar.java.binaries=$WORKSPACE \
                        -Dsonar.host.url='$SONARQUBE_URL' "
                    }
                }
            }
        }
        stage('Application_Services_ContainerImage_Build_&_Push_to_Private_Registry') {
            steps {
                script {
                    docker.withRegistry(REGISTRY_URL , 'harbor') {
                    sh "docker build -t $REGISTRY_NAME:adservice$BUILD_NUMBER ./src/adservice"
                    sh "docker push $REGISTRY_NAME:adservice$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:emailservice$BUILD_NUMBER ./src/emailservice"
                    sh "docker push $REGISTRY_NAME:emailservice$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:productcatalogservice$BUILD_NUMBER ./src/productcatalogservice"
                    sh "docker push $REGISTRY_NAME:productcatalogservice$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:shippingservice$BUILD_NUMBER ./src/shippingservice"
                    sh "docker push $REGISTRY_NAME:shippingservice$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:checkoutservice$BUILD_NUMBER ./src/checkoutservice"
                    sh "docker push $REGISTRY_NAME:checkoutservice$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:paymentservice$BUILD_NUMBER ./src/paymentservice"
                    sh "docker push $REGISTRY_NAME:paymentservice$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:currencyservice$BUILD_NUMBER ./src/currencyservice"
                    sh "docker push $REGISTRY_NAME:currencyservice$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:cartservice$BUILD_NUMBER ./src/cartservice"
                    sh "docker push $REGISTRY_NAME:cartservice$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:frontend$BUILD_NUMBER ./src/frontend"
                    sh "docker push $REGISTRY_NAME:frontend$BUILD_NUMBER"

                    sh "docker build -t $REGISTRY_NAME:loadgenerator$BUILD_NUMBER ./src/loadgenerator"
                    sh "docker push $REGISTRY_NAME:loadgenerator$BUILD_NUMBER"                                                                                                                                                                                  
                    } 
                }
            }
        }
        stage('App Deployment and Istio Mesh Services') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'APILoginRkUser', serverUrl: 'https://lb.coe.com/k8s/clusters/c-v84ch']) {
                        sh 'kubectl version --short'
                        sh "BUILD_NUMBER=${BUILD_NUMBER} && REGISTRY_NAME=${REGISTRY_NAME} && envsubst < ./release/kubernetes-manifests.yaml > ./release/kubernetes-ecomm.yaml"
                        sh 'kubectl apply -f ./release/kubernetes-ecomm.yaml'
                        sh 'kubectl apply -f ./release/istio-manifests.yaml'
                    }
                }
            }
        }
        stage ('Send Email Notification') {
            steps {
                // send to email
                emailext subject: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                    <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
                    to: '$DEFAULT_RECIPIENTS'
            }
        }    
    }
    post {
        success {
        emailext subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
                to: "priyanka.jare@os3infotech.com"
        }
        failure {
            emailext to: 'priyanka.jare@os3infotech.com',
                subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}