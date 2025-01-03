pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'redrayn/wargame'
        SONAR_HOST_URL = 'http://sonarqube:9000'
    }
    
    triggers {
        pollSCM('*/5 * * * *')  // 5분마다 GitHub 확인
    }
    
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/kimbeomjun90/web_wargamer.git',
                    credentialsId: 'Jenkins-GitHub'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    // SonarQube 스캐너 실행
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=wargame \
                        -Dsonar.sources=Web/src \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=admin \
                        -Dsonar.password=admin
                    """
                }
            }
        }
        
        stage('Build & Deploy') {
            steps {
                script {
                    // 도커 이미지 빌드
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} Web/
                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                    """
                    
                    // 쿠버네티스 배포
                    sh """
                        kubectl apply -f k8s/deployment.yaml -n devops
                        kubectl set image deployment/web-service web-service=${DOCKER_IMAGE}:${BUILD_NUMBER} -n devops
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    // 배포 상태 확인
                    sh """
                        kubectl get pods -n devops | grep web-service
                        kubectl get deployment web-service -n devops
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '파이프라인이 성공적으로 완료되었습니다.'
        }
        failure {
            echo '파이프라인이 실패했습니다. 롤백을 시작합니다.'
            sh 'kubectl rollout undo deployment/web-service -n devops'
        }
    }
}
