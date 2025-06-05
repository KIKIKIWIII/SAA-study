클라우드 환경에서 발생하는 다양한 이벤트를 추적 및 기록, 리소스 침탈 시도나 잠재적 위협을 경고하는 다양한 탐지 제어 기법이 있음

## CloudTrail

각종 활동 로그를 기록

아래와 같이 로그 기록의 범위와 어떤 로그를 기록할지 등을 결정해야 함.
- 관리 이벤트, 데이터 이벤트
- read-only, write-only
- 모든 리소스, 특정 리소스
- 모든 리전, 특정 리전
- 글로벌 서비스 기록 여부

활용
- 로그가 저장된 S3 버킷 정책 정의
- SSE-KMS 암호화 등 보안 수준 높이기
- 로그 파일 무결성
- Simple Notification Service 활용

## CloudWatch Logs

다수의 소스로부터 유입되는 로그를 수집해 저장 및 검색이 용이하게 함.

아래와 같은 다양한 AWS 서비스에서 CloudWatch에 로그가 유입됨.
- CloudTrail Logs
- VPC Flow Logs
	- VPC로 유입/유출되는 트래픽 정보 수집.
	- DHCP 트래픽이나, Amazon DNS 서버로 향하는 트래픽은 포함되지 않음
- RDS Logs
- Route 53 DNS Queries
- Lambda
- etc..

## Athena로 로그 검색하기

S3에 저장된 로그를 Athena는 SQL을 이용하여 로그 데이터를 검색할 수 있게 해줌.
- CloudWatch도 로그 데이터를 검색할 수도 있지만 원하는 방식으로 포맷팅이 불가능 함
- CSV, TSV, JSON, ORC, Parquet 형식을 지원함

## AWS Config를 이용한 리소스 환경설정 감사

AWS 리소스의 환경 설정 상태에 대한 모니터링도 필요

**예시: ec2-volume-inuse-check**
- 사용되지 않는 EBS 볼륨 감지 → 비용 절감
- 볼륨이 EC2에 연결되지 않으면 "비준수"로 표시
    

**주요 기능**
- **Config Rules**: 사전 정의된 규칙(예: `s3-bucket-public-read-prohibited`) 또는 커스텀 규칙으로 리소스 상태 평가
- **Configuration Timeline**: EBS 볼륨 생성/삭제 시점, IAM 정책 변경 기록 등 시간별 변경 이력 시각화
    
## Amazon GuardDuty

VPC flow logs, CloudTrail management event logs, Route 53 DNS query logs 등의 로그를 분석해 악성 IP 주소 및 도메인 네임, 악의적인 행동을 탐색

탐지 타입
- Backdoor
- Behavior
- Cryptocurrency
- Pentest
- etc..
[탐지 타입 공식 문서](https://docs.aws.amazon.com/ko_kr/guardduty/latest/ug/guardduty_finding-format.html#guardduty_threat_purposes)

## Amazon Inspector

보안 취약점과 규정 미준수 사항을 자동으로 탐지·관리하기 위해 필요
- GuardDuty가 악성 활동을 탐지한다면, Amazon Inspector는 인프라 취약점을 탐지함

**룰 패키지**
- 사전 정의된 검사 규칙 모음

**심각성 수준**
- 4가지 단계로 나뉨
- high, medium, low, informational

**예시: CVE-2024-1234 취약성 검사**
- 알려진 소프트웨어 취약점(CVE) 존재 시 "심각" 등급으로 보고
- EC2 인스턴스의 오픈 포트 검사 → 의도치 않은 네트워크 노출 감지

## Amazon Detective

AWS 리소스에서 발생한 보안 이벤트와 이상행동을 그래프 데이터베이스로 시각화·분석하는 서비스

- **핵심 기능**
    
    - VPC Flow Logs, CloudTrail, GuardDuty 등에서 정보를 수집해 관계형 그래프 모델로 저장        
    - 의심스러운 행동, 공격 흔적, 리소스 간 연관성 파악
    - 이벤트 간의 인과관계와 영향 분석 가능
        
- **특징**
    - CloudTrail 로그 단순 검색보다 더 빠르고 직관적으로 인사이트 도출
    - 사용 전 GuardDuty 활성화 필요
        



## Security Hub

- **무엇?**  
    AWS 계정 전체의 보안 상태를 한눈에 볼 수 있는 통합 대시보드
    
- **핵심 기능**
    - Inspector, GuardDuty, Macie 등 다양한 보안 서비스의 탐지 결과를 한 곳에 집계
    - AWS 보안 베스트 프랙티스 및 PCI DSS 등 표준과 비교해 상태 진단
    - 차트, 테이블 등 시각화 대시보드 제공
        
- **특징**
    - 보안 관리자는 한눈에 취약점, 이슈를 파악 가능
    - 여러 채널의 정보를 쉽게 비교·분석
        


## Amazon Fraud Detector

기업의 데이터 유행 및 위험 허용 수준에 맞춘 맞춤형 사기 탐지 서비스
    
- **핵심 기능**
    - 머신러닝 기반, 사용자가 설정한 기준에 따라 정상/사기 거래 범위 조정
    - 사기 행위 확률(0~1000점) 산출, 점수에 따라 자동 차단/검토
        
- **특징**
    - 판매자 사기, 가짜 계정, 온라인 지불 사기 등 다양한 케이스 탐지
    - 신규 고객에 대해 더 엄격한 기준 적용 등 유연한 정책 가능
        
## AWS Audit Manager


클라우드 리소스의 사용량 및 제어 수준을 평가하고, 컴플라이언스 감사 보고서를 자동 생성하는 서비스
    
- **핵심 기능**
    - PCI DSS 등 표준을 기준으로 감사 항목 자동 구성        
    - 리소스 구성, 로그 등 증적 자동 수집
    - 추가적인 비즈니스 요구사항도 쉽게 반영
        
- **특징**
    - 감사 준비, 보고서 생성, 규정 준수 입증까지 자동화
    - 온프레미스/멀티클라우드 환경도 지원
        

