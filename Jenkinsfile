pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        ARGOCD_SERVER = '34.47.186.224:32000'
        APP_NAME = 'ekart'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Ajinkya-A3/Ekart.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('Sonarqube Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=.
                    '''
                }
            }
        }

   //     stage('OWASP DP check') {
   //       steps {
   //             dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
   //                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
   //       }
   //     }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                script {
                    def imageTag = "at1asflame/ekart:${BUILD_NUMBER}"
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t ${imageTag} -f docker/Dockerfile ."
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image at1asflame/ekart:${BUILD_NUMBER} > trivy-report.txt"
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def imageTag = "at1asflame/ekart:${BUILD_NUMBER}"
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push ${imageTag}"
                    }
                }
            }
        }

        stage('GitOps: Update Image Tag & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git-credentials', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    script {
                        def imageTag = "at1asflame/ekart:${BUILD_NUMBER}"
                        sh """
                            git config user.name "jenkins"
                            git config user.email "jenkins@yourdomain.com"
                            git checkout main
                            sed -i 's|image: .*|image: ${imageTag}|' k8s/deploymentservice.yml
                            git add k8s/deploymentservice.yml
                            git commit -m "Update image to ${imageTag} via Jenkins"
                            git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Ajinkya-A3/Ekart.git
                            git push origin main
                        """
                    }
                }
            }
        }
       stage('ArgoCD Sync') {
            steps {
                withCredentials([usernamePassword(
                credentialsId: 'argocd-cred',
                usernameVariable: 'ARGOCD_USERNAME',
                passwordVariable: 'ARGOCD_PASSWORD'
                )]) {
        sh """
        export HOME=\$(mktemp -d)
        argocd login ${ARGOCD_SERVER} --username \$ARGOCD_USERNAME --password \$ARGOCD_PASSWORD --insecure
        argocd app sync ${APP_NAME}
        argocd app wait ${APP_NAME} --health --sync
    """
}

    }
}

    }
}
