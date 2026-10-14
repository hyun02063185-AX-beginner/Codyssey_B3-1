# B3-1 배포 작업 기록 — 2026-10-14

## 범위와 기준선

- 대상 계정: Building Things `345471468876`
- 리전: 서울 `ap-northeast-2`
- 배포 소스: `codyssey-mission-tech-glossary`의 `main`
- 배포 기준 커밋: `5ada699c1c462d957295c3e5d1b44567a9cc565b`
- 범위: 기준선에서 정적 사이트를 빌드해 EC2의 Nginx로 제공. 용어사전 기능 저장소는 수정하지 않았다.

## 작업 순서와 결과

1. 기준선에서 `npm run build`를 실행해 `dist`를 생성했다.
2. `b3-1-vpc` (`vpc-0ff58eb92cae855b5`, `10.0.0.0/16`)와 `b3-1-public-subnet` (`subnet-0754b3a7e047d243b`, `10.0.1.0/24`)을 생성했다.
3. `b3-1-igw` (`igw-0a76b6e7ce5f7dbba`)를 VPC에 연결하고, `b3-1-public-rt` (`rtb-0c43593adab690b32`)에 `0.0.0.0/0 → IGW` 경로를 만들었다. Public Subnet은 이 Route Table에 명시적으로 연결했다.
4. `b3-1-web-sg` (`sg-09cbf49fbacde0808`)를 만들었다. inbound는 HTTP TCP 80의 `0.0.0.0/0` 및 생성 당시 SSH TCP 22의 `121.135.181.46/32`, outbound는 all traffic `0.0.0.0/0`이다.
5. 키 페어 이름 `b3-1-ec2-key`를 사용해 `b3-1-web` (`i-0d3d200089acf1e25`)을 생성했다. 개인키 파일·경로·내용은 문서와 Git에 기록하지 않는다.
6. 인스턴스는 Amazon Linux 2023 x86_64, `t3.micro`, private IP `10.0.1.149`, 자동 할당 public IPv4 `13.125.211.95`로 구성했다. IAM instance profile은 없고 IMDSv2는 Required다.
7. 부팅 EBS `vol-0be705be47a8b5285`는 gp3 8 GiB이며 종료 시 삭제가 활성화되어 있다.
8. Nginx를 설치·enable하고 `/var/www/b3-1`에 `dist` 정적 파일을 배치했다.

## 검증 결과

- 내부: `nginx -t` 성공, `curl -I http://localhost`가 HTTP 200을 반환했다.
- 외부: http://13.125.211.95/#/terms 에서 기술용어사전 화면을 확인했다.
- 동료 평가: 자동 할당 Public IPv4를 사용하므로 stop/start 또는 terminate 뒤 IP가 바뀌거나 해제될 수 있다. 평가 전까지 현 인스턴스를 유지한다.

## 리전 및 계정 문제 해결

서울에서 `ec2:CreateVpc`가 SCP explicit deny로 거부되고 콘솔에는 “리전 제한됨”이 표시된 문제를 해결했다. 원인·조치·재발 방지는 [troubleshooting.md](troubleshooting.md)에 기록했다. 초기 관리 계정에서 만든 빈 VPC는 즉시 삭제했으며, 최종 B3-1 리소스는 모두 Building Things 계정에만 남아 있다.

## 보존과 정리

동료 평가 기간에는 서버를 유지하며 즉시 삭제하지 않는다. 평가가 끝나면 [cleanup-checklist.md](cleanup-checklist.md)의 ID 대조 및 정리 순서를 따른다. AWS 결제수단, 주소, 전화번호, MFA 정보, 개인키와 민감한 콘솔 화면은 기록하거나 제출하지 않는다.
