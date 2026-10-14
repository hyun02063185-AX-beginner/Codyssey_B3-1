# B3-1 · AWS 기술용어사전 배포

상태: **AWS 배포 및 외부 접속 검증 완료**. 동료 평가 기간에는 서버와 관련 리소스를 보존하고, 평가 종료 뒤 정리할 예정이다.

- 기준선: `main`, `5ada699c1c462d957295c3e5d1b44567a9cc565b`
- 평가 자료 저장소: `hyun02063185-AX-beginner/Codyssey_B3-1`
- 배포 소스 저장소: https://github.com/hyun02063185-AX-beginner/codyssey-mission-tech-glossary
- 이 저장소에는 평가 자료와 배포 설정만 관리한다. 용어사전 기능·학습 콘텐츠는 별도 요청 전까지 반영하지 않는다.
- 기존 GitHub Pages는 유지. 기능·콘텐츠·Mission Learning 추가는 동결.
- 대상 계정: Building Things `345471468876`, 리전: 서울 `ap-northeast-2`.
- 최종 B3-1 리소스는 모두 Building Things 계정에 생성했다. 관리 계정에서 빈 VPC를 한 번 잘못 만들었으나 즉시 삭제했으며, 관리 계정의 리소스 ID는 기록하지 않는다.

## 빌드 확인

2026-10-14 격리된 체크아웃에서 Node v24.19.0, npm 11.9.0으로 `npm ci`, `npm run build` 성공. `dist/index.html` 및 정적 파일 생성 확인. 추적 파일 변경 없음. [로그](docs/evidence/build.log).
Vite `base: './'`, React `HashRouter` 사용. AWS에서는 `http://<PUBLIC_IP>/#/terms` 형태로 접근한다.
큰 JS 청크 경고는 기존 상태로 기록하며 이번 실습에서 기능 최적화를 하지 않는다.

## 실제 배포 상태와 리소스

[아키텍처 초안](docs/architecture.png). 최종 리소스의 Name 접두사는 `b3-1-`이다.

| 항목 | 실제 구성 | ID/검증 |
|---|---|---|
| VPC | `b3-1-vpc`, `10.0.0.0/16` | `vpc-0ff58eb92cae855b5` |
| Public Subnet | `b3-1-public-subnet`, `10.0.1.0/24` | `subnet-0754b3a7e047d243b` |
| Internet Gateway | VPC에 연결한 `b3-1-igw` | `igw-0a76b6e7ce5f7dbba` |
| Public Route Table | `b3-1-public-rt`, `0.0.0.0/0 → IGW`, Subnet 명시 연결 | `rtb-0c43593adab690b32` |
| Security Group | `b3-1-web-sg`: HTTP TCP 80은 `0.0.0.0/0`, 생성 당시 SSH TCP 22는 `121.135.181.46/32`, outbound all traffic `0.0.0.0/0` | `sg-09cbf49fbacde0808` |
| Key pair | 이름 `b3-1-ec2-key`; 개인키 파일·경로·내용은 문서와 Git에 기록하지 않음 | 생성 확인 |
| EC2 | `b3-1-web`, Amazon Linux 2023 x86_64, `t3.micro`; private IP `10.0.1.149`, 자동 할당 public IPv4 `13.125.211.95`; IAM instance profile 없음, IMDSv2 Required | `i-0d3d200089acf1e25` |
| EBS | gp3 8 GiB, 종료 시 삭제 활성화 | `vol-0be705be47a8b5285` |
| Nginx | 설치·enable 완료, `/var/www/b3-1`에서 기준선의 `dist` 정적 파일 제공 | `nginx -t` 성공, localhost HTTP 200 |

Public Subnet은 IGW 직접 경로와 해당 Subnet의 명시 연결로 구성했다. EC2의 실제 인터넷 통신에는 Public IP, SG, NACL, 서비스 listen 상태도 필요하며 기본 NACL을 유지했다.

자동 할당 Public IPv4를 사용하므로 인스턴스를 stop/start 또는 terminate하면 현재 IP가 변경되거나 해제될 수 있다. 동료 평가의 검증 전에는 현재 인스턴스를 유지하며, URL 변경 시 이 문서와 증빙을 함께 갱신한다.

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

## 실제 검증과 동료 평가 URL

- [x] 배포 소스 기준선에서 `npm run build` 실행 후 `dist` 생성
- [x] EC2 내부에서 `nginx -t` 성공 및 `curl -I http://localhost` → HTTP 200
- [x] 외부 브라우저에서 기술용어사전 화면 확인: http://13.125.211.95/#/terms
- [x] VPC, Subnet, IGW, Route Table 명시 연결, SG, EC2, EBS 실구성 기록
- [x] 실제 배포 중 발생한 조직 리전 제한 및 콘솔 표시 문제 해결 기록: [문제 해결 기록](docs/troubleshooting.md)

동료 평가 시 위 URL에서 기술용어사전 화면을 확인한다. 개인키, AWS 결제수단, 주소, 전화번호, MFA 정보와 민감한 콘솔 화면은 증빙 및 Git에 포함하지 않는다. 평가용 캡처 목록과 가림 기준은 [evidence 안내](docs/evidence/README.md)를 따른다.

## 공식 참고

- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html
- https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html
- https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html
- https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-best-practices.html


## 제출 문서

- [아키텍처](docs/architecture.png)
- [문제 해결 기록](docs/troubleshooting.md)
- [정리 체크리스트](docs/cleanup-checklist.md)
- [배포 작업 기록](docs/deployment-log-2026-10-14.md)
- [동료 평가용 증빙 안내](docs/evidence/README.md)
- [빌드 로그](docs/evidence/build.log)
