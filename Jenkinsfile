pipeline {
    agent any

    environment {
        // 하버 주소 (설정한 도메인)
        REGISTRY = 'harbor.local.net'
        // 미리 만들어두신 프로젝트 이름
        PROJECT = 'test'
        // 이미지 이름
        IMAGE_NAME = 'my-web-site'
        // 1단계에서 만든 자격증명 ID
        CREDENTIAL_ID = 'harbor-login'
    }

    stages {
        stage('Calculate Version') {
            steps {
                script {
                    // 1. 현재 빌드 번호 가져오기 (예: 1, 2, 10...)
                    def buildNum = currentBuild.number.toInteger()
                    
                    // 2. 0.1을 곱해서 버전 계산 (예: 1 -> 0.1, 12 -> 1.2)
                    // String.format을 써서 소수점 첫째 자리까지 깔끔하게 자름
                    def verCalc = String.format("%.1f", buildNum * 0.1)
                    
                    // 3. 환경 변수에 저장 (v0.1, v0.2 형식)
                    env.IMAGE_TAG = "v${verCalc}"
                    
                    echo "🎉 이번 빌드 버전은 [ ${env.IMAGE_TAG} ] 입니다."
                }
            }
        }

        stage('Build & Push') {
            steps {
                script {
                    // 전체 이미지 주소 조합
                    def fullImageName = "${REGISTRY}/${PROJECT}/${IMAGE_NAME}:${env.IMAGE_TAG}"

                    // 1. 도커 이미지 빌드
                    sh "docker build -t ${fullImageName} ./source"

                    // 2. 하버 로그인 및 푸쉬
                    withCredentials([usernamePassword(credentialsId: CREDENTIAL_ID, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "docker login ${REGISTRY} -u $USER -p $PASS"
                        sh "docker push ${fullImageName}"
                    }
                    
                    echo "✅ 하버 푸쉬 완료: ${fullImageName}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // 배포 시에도 방금 만든 태그를 사용
                    def fullImageName = "${REGISTRY}/${PROJECT}/${IMAGE_NAME}:${env.IMAGE_TAG}"

                    // 기존 컨테이너 삭제 후 재실행
                    sh "docker rm -f my-web-server || true"
                    sh "docker run -d -p 8081:80 --name my-web-server ${fullImageName}"
                }
            }
        }
    }
}
