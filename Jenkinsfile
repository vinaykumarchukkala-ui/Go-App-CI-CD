pipeline {
   agent any

   environment {
    DOCKER_USERBNAME = credentials('dockerhub-username') // jenkins credentials ID
    DOCKER_PASSWORD = credentials('dockerhub-password') // jenkins credentials ID
    GIT_TOKEN  = credentials('git-token') // jenkins credentials ID
    GO_VERSION = '1.22'
    IMAGE_TAG = "${env.BUILD_ID}"
   }

   options{
        skipStagesAfterUnstable()
    }

    stages {
        stage('Checkout') {
            steps{
                checkout scm
            }
        }
    
    }

        stage('Set uo Go') {
            steps {
                sh '''
                    wget htts://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz
                    sudo tar -c /usr/local -xzf go${GO_VERSION}.linux-amd64.tar.gz
                    export PATH=$PATH:/usr/local/go/bin
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'go build -o go-web-app'
            }
        }

        stage('Test') {
            steps {
                sh 'go test ./...'
            }
        }

        stage('Code Quality') {
            steps {
                sh '''
                    curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s v1.56.2
                    ./bin/golangci-lint run
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            docker build -t ${DOCKER_USERNAME}/go-web-app:${IMAGE_TAG} .
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            docker push ${DOCKER_USERNAME}/go-web-app:${IMAGE_TAG}
                        '''
                    }
                }
            }
        }

        stage('Update Helm Chart Tag') {
            steps {
                sh '''
                    git config --global user.email "vinaykumarchukkala-ui"
                    git config --global user.email "vinaykumarchukkala@gmail.com"
                    sed -i "s/tag:.*/tag: ${IMAGE_TAG}/" helm/go-web-app-chart/values.yaml
                    git add helm/go-web-app-chart/values.yaml
                    git commit -m "Update image tag to ${IMAGE_TAG}"
                    git push https://$GIT_TOKEN@github.com/${env.GIT_REPO}
                '''
            }
        }
    post {
        always {
            cleanWs()
        }
    }
    
}