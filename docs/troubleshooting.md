# B3-1 Troubleshooting

## A. 서울 리전 VPC 생성이 SCP explicit deny로 거부됨

- 증상: Building Things 계정에서 서울 리전 VPC를 만들 때 `ec2:CreateVpc`가 SCP의 explicit deny로 거부되었다.
- 확인: Root에 연결된 `AdvancedModeRegionRestrictionSecurityControlPolicy`의 `RegionFloor` 허용 리전 목록을 확인했다.
- 원인: 허용 리전 목록에 `ap-northeast-2`가 없었다.
- 해결: 정책의 허용 리전 목록에 서울 `ap-northeast-2`만 추가했다.
- 재검증: Building Things 계정의 서울 리전에서 최종 VPC `b3-1-vpc` (`vpc-0ff58eb92cae855b5`)를 생성했다.
- 재발 방지: 리소스 생성 전 대상 계정뿐 아니라 Root 수준 SCP의 허용 리전과 대상 리전이 일치하는지 먼저 확인한다. 과도한 리전 허용으로 우회하지 않는다.

## B. 서울 콘솔에 “리전 제한됨” 오버레이가 표시됨

- 증상: Building Things 계정에서 서울 콘솔로 직접 이동했을 때 서울 리전이 숨김 처리되어 “리전 제한됨” 오버레이가 보였다.
- 확인: 콘솔의 Visible Regions 설정을 확인했다.
- 원인: 서울 `ap-northeast-2`가 표시되는 리전 목록에서 제외되어 있었다.
- 해결: “표시되는 리전 편집”에서 서울 `ap-northeast-2`를 표시하도록 저장했다.
- 재검증: 서울 콘솔에서 VPC·EC2 리소스를 정상적으로 생성·조회하고 배포를 완료했다.
- 재발 방지: SCP의 리전 허용 여부와 Visible Regions의 콘솔 표시 여부를 별도로 점검한다. 둘 중 하나만 맞아도 작업이 가능하다고 가정하지 않는다.

## C. 관리 계정에 빈 VPC를 잘못 생성함

- 경위: 초기 작업 중 관리 계정에서 빈 VPC를 실수로 생성했다.
- 조치: 즉시 삭제했고, 관리 계정의 리소스 ID는 이 평가 문서에 기록하지 않는다.
- 결과: 최종 B3-1 리소스는 모두 Building Things 계정의 서울 리전에만 생성했다.
- 재발 방지: 생성 전 콘솔의 현재 계정과 리전을 확인하고, Name 태그와 리소스 ID를 배포 기록에 즉시 대조한다.

## 준비 단계 관찰: 빌드의 큰 JS 청크 경고

- 환경: 2026-10-14, 기준선 `5ada699c1c462d957295c3e5d1b44567a9cc565b`, Node v24.19.0 / npm 11.9.0.
- 재현: `npm ci` 후 `npm run build`.
- 관찰: 빌드 종료 코드 0, `dist` 생성. Vite가 일부 청크 500 kB 초과 경고를 출력했다.
- 판단/조치: 배포 실패가 아닌 성능 경고다. 기능 동결에 따라 코드 분할 수정 없이 보존했다.
- 증빙: [build.log](evidence/build.log).

일반 진단 순서: EC2 상태·Public IP → Subnet Route Table/IGW → SG의 목적 포트/출발지 → NACL → nginx 상태/listen → 파일 위치/권한 → 브라우저 정적 자산 요청. SSH 시간 초과와 `Permission denied (publickey)`는 구분한다. 전자는 네트워크 경로·SG, 후자는 사용자 이름·개인키·서버 공개키를 우선 점검하며 포트 22 전체 공개로 해결하지 않는다.
