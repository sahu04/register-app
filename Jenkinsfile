pipeline {
    agent { label 'jenkins-agent' }

    // Define environment variables
     environment {
           SONAR_TOKEN = credentials('jenkins-sonarqube-token1')
    //     APP_NAME = "register-app-pipeline"
    //     RELEASE = "1.0.0"
    //     DOCKER_USER = "ashfaque9x"
    //     DOCKER_PASS = 'dockerhub'
    //     IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
    //     IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    //     JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
     }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/sahu04/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        // SonarQube Analysis
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token1') { 
                        // sh "mvn sonar:sonar"
                        sh '''
                         mvn clean verify sonar:sonar \
                         -Dsonar.projectKey=jenkins-sonar \
                         -Dsonar.projectName='jenkins-sonar' \
                         -Dsonar.host.url=http://15.207.16.61:9000/
                         -Dsonar.login=${SONAR_TOKEN}
                        '''
                    }
                }	
            }
        }

        // Quality Gate
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonar-token'
                }	
            }
        }

        // Uncomment the following stage to build and push Docker Image
        // stage("Build & Push Docker Image") {
        //     steps {
        //         script {
        //             docker.withRegistry('', DOCKER_PASS) {
        //                 docker_image = docker.build "${IMAGE_NAME}"
        //             }

        //             docker.withRegistry('', DOCKER_PASS) {
        //                 docker_image.push("${IMAGE_TAG}")
        //                 docker_image.push('latest')
        //             }
        //         }
        //     }
        // }

        // Uncomment the following stage to run a Trivy Scan
        // stage("Trivy Scan") {
        //     steps {
        //         script {
        //             sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ashfaque9x/register-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
        //         }
        //     }
        // }

        // Uncomment the following stage to cleanup artifacts
        // stage ('Cleanup Artifacts') {
        //     steps {
        //         script {
        //             sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
        //             sh "docker rmi ${IMAGE_NAME}:latest"
        //         }
        //     }
        // }

        // Uncomment the following stage to trigger the CD pipeline
        // stage("Trigger CD Pipeline") {
        //     steps {
        //         script {
        //             sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-13-232-128-192.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
        //         }
        //     }
        // }
    }

    // Uncomment the following post actions for notifications
    // post {
    //     failure {
    //         emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                  subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
    //                  mimeType: 'text/html', to: "ashfaque.s510@gmail.com"
    //     }
    //     success {
    //         emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
    //                  subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
    //                  mimeType: 'text/html', to: "ashfaque.s510@gmail.com"
    //     }      
    // }
}
