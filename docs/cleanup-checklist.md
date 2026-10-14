# B3-1 Cleanup checklist

상태: 미실행. 삭제 대상은 이번 실습에서 만든 리소스 ID로만 한정한다. Name 태그만으로 판단하지 않는다.

## 삭제 전
- [ ] 실제 AWS 계정·서울 리전·B3-1 리소스 ID 대조
- [ ] 외부 접속 성공, 내부 curl, SG, 경로, IAM, 문제 해결 증빙 저장
- [ ] 삭제 후 AWS URL 접속이 중단됨을 README에 기록

## 정리 순서
- [ ] B3-1 EC2 **Terminate** 완료 확인 (Stop만으로 끝내지 않음)
- [ ] 연결 EBS가 삭제되었는지 Volumes에서 ID 확인, 남은 실습 볼륨만 삭제
- [ ] EIP 미생성·미할당 확인. 자동 Public IPv4 해제 확인
- [ ] B3-1 Snapshot/AMI 추가 생성 여부 확인. 없으면 없음으로 기록
- [ ] 인스턴스 ENI 해제 확인 후 실습 SG 삭제
- [ ] 실습 Key pair 삭제, 로컬 개인키는 제출물에서 제외하고 실습 종료 정책에 따라 정리
- [ ] 실습 Subnet 삭제 (필요 시 명시 Route Table 연결 해제)
- [ ] 사용자 생성 실습 Route Table 삭제
- [ ] IGW 분리·삭제
- [ ] 실습 VPC 삭제 (기본 리소스는 VPC 삭제에 따라 정리)
- [ ] 별도로 만든 IAM 정책/역할이 있을 때만 참조 해제 후 정리
- [ ] NAT Gateway/ELB/RDS/EIP/Snapshot 미생성 상태 재확인
- [ ] 비용 화면은 반영 지연을 고려해 다음 날 재확인. 예산 알림은 최종 비용 확인까지 유지

보호 대상: 기존 GitHub Pages, 기본 VPC 및 타 실습 리소스, Codyssey Management Account, Codyssey Identity Delegated Admin, AWS 조직 설정.

| 확인 시각(KST) | 계정/리전 | 리소스 ID | 결과/증빙 |
|---|---|---|---|
| 미실행 | 미확인 | 미생성 | 미확인 |
