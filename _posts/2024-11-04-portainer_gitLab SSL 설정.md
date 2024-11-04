## gitlab SSL 적용하기


2. Local Certificate Authority (CA) 생성 및 인증서 서명
로컬 CA를 생성하여 자체 서명된 인증서보다 안전하게 설정할 수 있습니다. 로컬 네트워크에 설치된 다른 장치에서도 인증서를 신뢰하도록 설정할 수 있습니다.

CA 키 및 인증서 생성
로컬 CA 키 및 인증서를 생성합니다.

bash
코드 복사
openssl genpkey -algorithm RSA -out /etc/gitlab/ssl/myCA.key
openssl req -x509 -new -nodes -key /etc/gitlab/ssl/myCA.key -sha256 -days 1024 -out /etc/gitlab/ssl/myCA.crt
GitLab SSL 키 및 인증서 요청 파일 생성

bash
코드 복사
openssl req -new -newkey rsa:2048 -nodes -keyout /etc/gitlab/ssl/gitlab.key -out /etc/gitlab/ssl/gitlab.csr
CSR 파일을 사용하여 CA로 인증서 서명

bash
코드 복사
openssl x509 -req -in /etc/gitlab/ssl/gitlab.csr -CA /etc/gitlab/ssl/myCA.crt -CAkey /etc/gitlab/ssl/myCA.key -CAcreateserial -out /etc/gitlab/ssl/gitlab.crt -days 500 -sha256
GitLab 설정 파일 수정 및 적용
위에서 설명한 대로 /etc/gitlab/gitlab.rb 파일을 수정하여 SSL 인증서를 적용합니다.

클라이언트 시스템에서 로컬 CA 신뢰 추가
각 클라이언트 (PC, 브라우저 등)에서 myCA.crt 파일을 설치하여 신뢰할 수 있도록 설정합니다.




## Portainer SSL 적용하기

Portainer에서 SSL을 설정하여 HTTPS를 통해 안전하게 접속하려면, SSL 인증서를 준비하고 Portainer 컨테이너에 설정을 추가해야 합니다. 아래는 SSL을 설정하는 주요 방법입니다.

1. SSL 인증서 준비하기
Portainer에서 사용할 SSL 인증서를 준비해야 합니다. 다음 중 하나를 선택할 수 있습니다:

Let's Encrypt 인증서: 공인 인증서를 무료로 발급받을 수 있습니다.
자체 서명된 인증서(Self-signed certificate): 내부 네트워크에서만 사용할 경우 적합합니다.
기업에서 발급받은 SSL 인증서: 사내에서 발급받은 인증서를 사용할 수도 있습니다.
자체 서명된 SSL 인증서 생성
자체 서명된 인증서를 생성하려면 OpenSSL을 사용할 수 있습니다. 아래 명령어로 인증서를 생성하세요:

bash
코드 복사
sudo mkdir -p /opt/portainer/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /opt/portainer/ssl/portainer.key \
  -out /opt/portainer/ssl/portainer.crt
portainer.key: SSL 개인 키
portainer.crt: SSL 인증서
OpenSSL 명령어 실행 중 Common Name (CN) 항목에 도메인 이름 또는 IP 주소를 입력하세요.

2. Portainer를 HTTPS로 실행하도록 설정
Portainer 실행 시 HTTPS를 활성화하고, 생성한 SSL 인증서와 키 파일을 컨테이너에 마운트해야 합니다.

기존의 Portainer 컨테이너가 실행 중이라면 먼저 중지하고 제거합니다:

bash
코드 복사
docker stop portainer
docker rm portainer
이제 HTTPS를 활성화하여 Portainer를 다시 실행합니다.

bash
코드 복사
docker run -d -p 9443:9443 --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  -v /opt/portainer/ssl/portainer.crt:/certs/portainer.crt \
  -v /opt/portainer/ssl/portainer.key:/certs/portainer.key \
  portainer/portainer-ce --ssl --sslcert /certs/portainer.crt --sslkey /certs/portainer.key
9443:9443: Portainer에 HTTPS로 접근하기 위해 외부 9443 포트를 사용합니다. 필요에 따라 다른 포트를 사용할 수도 있습니다.
/certs/portainer.crt와 /certs/portainer.key: SSL 인증서와 키 파일의 경로입니다.
--ssl, --sslcert, --sslkey: Portainer에 SSL을 활성화하는 플래그입니다.
3. Portainer 접속 확인
컨테이너가 정상적으로 실행되면, 웹 브라우저에서 다음 주소로 Portainer에 접속할 수 있습니다:

arduino
코드 복사
https://<IP 주소 또는 도메인>:9443
자체 서명된 인증서를 사용하는 경우, 브라우저에서 SSL 경고가 나타날 수 있습니다. 신뢰할 수 있는 SSL 인증서를 사용하면 이 경고가 사라집니다.
