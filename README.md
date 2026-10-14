# B3-1 · AWS 기술용어사전 배포

상태: **설계·빌드 검증 완료 / AWS 생성·접속·정리 미검증**. 이 문서는 성공 증빙을 대신하지 않는다.

- 기준선: `main`, `5ada699c1c462d957295c3e5d1b44567a9cc565b`
- 평가 자료 저장소: `hyun02063185-AX-beginner/Codyssey_B3-1`
- 배포 소스 저장소: https://github.com/hyun02063185-AX-beginner/codyssey-mission-tech-glossary
- 이 저장소에는 평가 자료와 배포 설정만 관리한다. 용어사전 기능·학습 콘텐츠는 별도 요청 전까지 반영하지 않는다.
- 기존 GitHub Pages는 유지. 기능·콘텐츠·Mission Learning 추가는 동결.
- 리전: 서울 `ap-northeast-2`. 대상: Building Things 계정/프로젝트.
- Management Account 및 Identity Delegated Admin은 변경하지 않는다.

## 빌드 확인

2026-10-14 격리된 체크아웃에서 Node v24.19.0, npm 11.9.0으로 `npm ci`, `npm run build` 성공. `dist/index.html` 및 정적 파일 생성 확인. 추적 파일 변경 없음. [로그](docs/evidence/build.log).
Vite `base: './'`, React `HashRouter` 사용. AWS에서는 `http://<PUBLIC_IP>/#/terms` 형태로 접근한다.
큰 JS 청크 경고는 기존 상태로 기록하며 이번 실습에서 기능 최적화를 하지 않는다.

## 설계

[아키텍처 초안](docs/architecture.png). 리소스 Name 접두사는 `b3-1-`, 가능하면 `Mission=B3-1` 태그 적용.

| 항목 | 계획 | 실제 ID/확인 |
|---|---|---|
| VPC | 10.0.0.0/16 | 미생성 |
| Public Subnet | 10.0.1.0/24, AZ 1개 | 미생성 |
| IGW | VPC에 연결 | 미생성 |
| Route Table | local + 0.0.0.0/0 → IGW, Subnet 명시 연결 | 미생성 |
| EC2 | Amazon Linux 2023, x86_64 소형 1대, 자동 Public IPv4 | 유형·가격 확인 후 결정 |
| EBS | gp3 최소 부팅 용량, 종료 시 삭제 | 미생성 |
| SG inbound | TCP 80: 0.0.0.0/0 / TCP 22: 작업자 현재 공인 IP/32 | 미생성 |
| SG outbound | 초기 기본 허용, 패키지 설치·DNS 등 용도 기록 | 미생성 |
| Nginx | TCP 80, /var/www/b3-1, dist 내용 제공 | 미설치 |

Public Subnet이라는 분류는 IGW 직접 경로에 따른다. EC2의 실제 인터넷 통신에는 Public IP, SG, NACL, 서비스 listen 상태도 필요하다. 기본 NACL 유지.
SSH /32는 실제 SSH를 실행하는 PC의 공인 IP를 사용한다. 이 채팅 실행 환경의 IP를 대신 넣지 않는다. 교육장과 집 이동 시 갱신한다.

## 비용 선행 조건

1. Building Things의 대상 계정과 결제 범위를 확인한다. 크레딧 잔액·만료일은 콘솔에서 재확인한다.
2. 제안: 월 USD 5 비용 예산, 실제 USD 1/3/5 알림. 크레딧을 제외한 사용 비용도 감지하도록 비용 집계 설정을 확인한다. 수신 이메일은 사용자가 지정한다.
3. 알림 저장과 수신 주소를 확인한 뒤 리소스를 생성한다. 조직 결제 권한이 없으면 중단하고 가능한 결제 관리 범위를 확인한다.
4. 예산은 자동 지출 차단 장치가 아니며 알림은 지연될 수 있다. 실습 직후 종료·삭제한다.
5. EC2, Public IPv4, EBS 비용을 확인한다. NAT Gateway, ELB, RDS, EIP, Snapshot은 만들지 않는다.

## IAM 최소권한

- Builder ID MFA는 AWS 리소스 작업 권한과 구분한다. 대상 계정의 현재 역할/권한을 먼저 확인한다.
- root나 장기 Access key를 일상 작업용으로 만들지 않는다. 임시 역할 세션을 사용한다.
- EC2 런타임은 정적 파일만 제공하므로 IAM instance profile 불필요. AWS API 자격 증명을 서버에 두지 않는다. IMDSv2 required.
- 배포 작업자 권한은 서울 리전의 필요한 EC2/VPC 조회·생성·태깅·연결·삭제 작업으로 제한한다. RunInstances는 AMI·Subnet·SG·EBS 등 종속 리소스 권한도 필요하다.
- 실제 IAM 정책/역할과 권한 근거는 배포 후 기록한다. 아직 정책을 적용하거나 최소권한을 검증하지 않았다. AWS 관리형 AdministratorAccess를 최소권한 증빙으로 표기하지 않는다.

## 실행 순서

예산 → VPC → Subnet → IGW 연결 → Route Table/기본 경로/명시 연결 → SG → Key pair/EC2 → Nginx → dist 업로드 → 검증 → 증빙 → 정리.
EC2 생성 시 공개 IPv4 활성화, EBS 종료 시 삭제, IMDSv2 필수, 불필요한 역할·추가 볼륨 없음 확인.

Amazon Linux 2023 서버에서:

```sh
sudo dnf install -y nginx
sudo install -d -m 755 /var/www/b3-1
```

용어사전 저장소의 지정 기준선을 별도 체크아웃하여 `npm ci` 후 `npm run build`로 dist를 생성한다. 이 평가 자료 저장소 자체에는 npm 프로젝트가 없다.

아래 명령은 용어사전 체크아웃 디렉터리를 기준으로 한다. `B3_EVAL_REPO`에는 이 평가 자료 저장소를 복제한 **절대 경로**를 넣는다. Nginx 설정은 이 평가 자료 저장소의 `deploy/b3-1/nginx.conf`를 사용하며, 로컬에서 빌드한 dist 내용을 전송한다. 꺾쇠 자리표시는 실제 값으로 바꾼다. 개인키는 Git에 저장하지 않는다.

```sh
B3_EVAL_REPO=/absolute/path/to/Codyssey_B3-1
chmod 400 <KEY.pem>
scp -i <KEY.pem> -r dist/. ec2-user@<PUBLIC_IP>:~/b3-1-dist
scp -i <KEY.pem> "$B3_EVAL_REPO/deploy/b3-1/nginx.conf" ec2-user@<PUBLIC_IP>:~/b3-1-nginx.conf
```

서버에서:

```sh
sudo cp -r ~/b3-1-dist/. /var/www/b3-1/
sudo find /var/www/b3-1 -type d -exec chmod 755 {} +
sudo find /var/www/b3-1 -type f -exec chmod 644 {} +
sudo cp ~/b3-1-nginx.conf /etc/nginx/nginx.conf
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
curl -I http://localhost
```

설정은 AL2023의 Nginx 기본 설정을 교체하는 전용 실습 서버용이다. 서버에서 npm 설치/빌드하지 않는다.

## 실제 성공 증빙 — 모두 미확인

- [ ] 예산·크레딧·대상 계정 확인 (개인/결제정보는 제출 화면에서 가림)
- [ ] VPC/Subnet/IGW/Route Table 연결 화면과 리소스 ID
- [ ] SG 80 공개 및 SSH /32, EC2 Public IP·상태·IAM 근거
- [ ] `nginx -t` 성공, `systemctl is-active nginx` active
- [ ] EC2 내부 `curl -I http://localhost` → HTTP 200
- [ ] 외부 PC `curl -I http://<PUBLIC_IP>/` → HTTP 200
- [ ] 외부 브라우저 `http://<PUBLIC_IP>/#/terms` 실제 화면·검색·미션 이동·새로고침
- [ ] 실제 문제 1건 기록, 아키텍처를 실제 구성으로 갱신
- [ ] cleanup 체크리스트와 잔존 리소스 확인

현재 접속 URL/Public IP: **아직 없음**. 정리 후에는 접속 불가와 검증 당시 시각·IP를 구분하여 기록한다.

## 공식 참고

- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html
- https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html
- https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html
- https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-best-practices.html


## 제출 문서

- [아키텍처](docs/architecture.png)
- [문제 해결 기록](docs/troubleshooting.md)
- [정리 체크리스트](docs/cleanup-checklist.md)
- [빌드 로그](docs/evidence/build.log)
