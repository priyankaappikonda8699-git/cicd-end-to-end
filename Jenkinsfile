pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "priyappi/cicd-ete"
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: 'github-creds', 
                url: 'https://github.com/priyankaappikonda8699-git/cicd-end-to-end.git',
                branch: 'main'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    sh '''
                    echo 'Buid Docker Image'
                    docker build -t priyappi/cicd-ete:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Docker Login') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push the artifacts'){
           steps{
                script{
                    sh '''
                    echo 'Push to Repo'
                    docker push priyappi/cicd-ete:${BUILD_NUMBER}
                    '''
                }
            }
        }
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'github-creds', 
                url: 'https://github.com/priyankaappikonda8699-git/cicd-manifests.git',
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'github-creds', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cat deploy.yaml
                        sed -i "s|image:.*|image: priyappi/cicd-ete:${BUILD_NUMBER}|g" deploy.yaml
                        cat deploy.yaml
                        git config user.email "jenkins@example.com"
                        git config user.name "jenkins"
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/priyankaappikonda8699-git/cicd-manifests.git HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}
