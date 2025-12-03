pipeline {
    agent any

    stages {
        stage('Build Image') {
            steps {
                // 1. 도커 이미지 만들기 (이름: my-web-site)
                // source 폴더 안에 있는 Dockerfile을 사용
                sh 'docker build -t my-web-site:latest ./source'
            }
        }

        stage('Deploy') {
            steps {
                // 2. 기존에 떠 있던 컨테이너가 있다면 삭제 (에러 방지용 try-catch)
                sh 'docker rm -f my-web-server || true'

                // 3. 새로운 컨테이너 실행
                // -d: 백그라운드 실행
                // -p 8081:80 -> 내 컴퓨터 8081포트로 접속하면 컨테이너 80포트(Nginx)로 연결
                sh 'docker run -d -p 8081:80 --name my-web-server my-web-site:latest'
            }
        }
    }
}
