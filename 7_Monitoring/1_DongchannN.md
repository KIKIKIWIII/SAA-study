# CloudTrail, CloudWatch
CloudTrail, CloudWatch, AWS Config를 사용하여 할 수 있는 것들
* 성능 모니터링
* 애플리케이션 문제점 감지
* 보안 문제점 감지
* 로그 이벤트 모니터링
* AWS 리소스 인벤토리 관리

## CloudTrail

> **참고**: CloudTrail 콘솔의 **Event history** 페이지는 관리 이벤트만 표시하며 데이터 이벤트, Insights 이벤트, 네트워크 활동 이벤트는 표시하지 않음.  

### 이벤트 유형
[Understanding CloudTrail events - AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-events.html)
1. **Management events**
	* AWS 계정의 리소스에 대한 관리 작업 정보
	* **포함 예시**:
		* 보안 구성 (IAM `AttachRolePolicy`)
		* 디바이스 등록 (EC2 `CreateDefaultVpc`)
		* 데이터 라우팅 규칙 설정 (EC2 `CreateSubnet`)
		* 로깅 설정 (CloudTrail `CreateTrail`)
		* 콘솔 로그인 등 비API 이벤트

2. **Data events**
	* 리소스에서 수행되는 리소스 작업 정보
	* **포함 예시**:
		* S3 객체 수준 API 활동 (`GetObject`, `DeleteObject`, `PutObject`)
		* Lambda 함수 실행 활동 (`Invoke`)
		* CloudTrail Lake 채널의 `PutAuditEvents` 활동
		* SNS `Publish` 및 `PublishBatch` API 작업
		* DynamoDB, AppSync, Bedrock, CloudWatch 등의 데이터 작업

3. **Insights events**
	* 계정의 일반적인 사용 패턴과 크게 다른 비정상적인 활동 캡처
	* **감지 예시**:
		* 갑작스러운 특정 API 호출 증가 (예: S3 `deleteBucket` 호출이 분당 20개에서 100개로 증가)
		* 갑작스러운 API 호출 감소 (예: EC2 `AuthorizeSecurityGroupIngress` 호출이 분당 20개에서 0개로 감소)
		* 비정상적인 오류율 증가 (예: IAM `DeleteInstanceProfile`에서 `AccessDeniedException` 오류 급증)

4. **Network activity events**
	* VPC 엔드포인트를 통해 프라이빗 VPC에서 AWS 서비스로 수행된 API 호출 기록
	* **지원 서비스**:
		* AWS CloudTrail
		* Amazon Comprehend Medical
		* Amazon EC2
		* AWS IoT FleetWise
		* AWS KMS
		* AWS Lambda
		* Amazon S3 (다중 리전 액세스 포인트 제외)
		* AWS Secrets Manager

### 이벤트 기록 방법
[What Is AWS CloudTrail? - AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
1. **Event History**
	* 90일간의 Management events 기록을 제공
	* 조회, 검색, 다운로드 가능하며 변경 불가능한 기록
	* 요금 없음

2. **Trails**
	* 이벤트를 S3 버킷에 저장
	* 지속적인 관리 이벤트 사본 하나를 S3 버킷에 무료로 전달 (S3 스토리지 요금은 별도)
	* 단일 리전 또는 전체 리전의 로그 이벤트 기록 가능
	* 단일 리전에 최대 5개까지 Trail 생성 가능
	* **용도**: 단순 저장에 적합

3. **CloudTrail Lake**
	* 사용자 및 API 활동을 위한 관리형 데이터 레이크
	* 기존 행 기반 JSON 형식 이벤트를 Apache ORC 형식으로 변환
	* **용도**: 심층적인 분석과 쿼리에 적합

### 로그 파일 무결성 검증
[Validating CloudTrail log file integrity - AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html)
CloudTrail이 로그 파일을 전송한 후 해당 파일이 수정, 삭제 또는 변경되지 않았는지 확인하기 위해 Log File Integrity를 활성화하여 사용할 수 있음.

## CloudWatch
[What is Amazon CloudWatch? - Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
AWS 리소스 및 AWS에서 실행되는 애플리케이션을 실시간으로 모니터링 함. CloudWatch 홈페이지에서 자동으로 사용자가 사용하는 AWS 서비스에 관한 지표가 표시 됨.

* **CloudWatch Alarms**: 메트릭스 값에 따라 특정 액션을 취할 수 있음
* **CloudWatch Logs**: 리소스의 로그를 수집, 저장, 시각화하고 검색 기능 제공

### CloudWatch Metrics
[Metrics in Amazon CloudWatch - Amazon CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html)
CloudWatch는 네임스페이스를 기준으로 메트릭스를 관리 함. 네임스페이스는 메트릭스를 담는 컨테이너와 같은 역할을 하며, 비슷한 이름을 혼동하지 않도록 도움.

개별 성능 지표는 다음으로 구분됩니다:
* 네임스페이스
* 네임
* 차원(Optional)

예를 들어, 여러 EC2 인스턴스가 있는 경우:
* 각 인스턴스마다 `AWS/EC2` 네임스페이스에 `CPUUtilization` 지표 생성
* `InstanceId`와 같은 차원 이름으로 다른 요소 구분

#### 모니터링 유형

1. **기본 모니터링(basic monitoring)**
	* AWS 서비스 사용 시 자동 활성화
	* 무료로 제공

2. **상세 모니터링(detailed monitoring)**
	* 일부 서비스에서만 제공
	* 추가 요금 부과

#### 해상도 유형

1. **표준 해상도** (1분 단위)
	* AWS 서비스에서 기본적으로 생성되는 지표

2. **고해상도** (1초 단위)
	* 사용자 지정 지표에서 선택 가능
	* 애플리케이션 활동에 대한 즉각적인 통찰력 제공
	* PutMetricData 호출이 더 자주 발생하여 비용 증가 가능
	* 경보 설정: 10초/30초 단위(고해상도) 또는 60초 단위(일반)
	* 고해상도 경보는 더 높은 요금 부과

### 기한 만료 및 삭제
CloudWatch에서는 메트릭스를 임의로 삭제할 수 없으며, 정해진 기한에 자동으로 만료돼 삭제.

* **고해상도 메트릭스**: 3시간 동안 저장
	* 이후 1분 해상도 데이터 포인트로 편입
	* 원래 고해상도 데이터는 만료와 함께 삭제

* **1분 해상도 데이터**: 15일 후
	* 5개 데이터 포인트가 5분 해상도로 편입
	* 이후 63일간 유지

* **5분 해상도 데이터**: 63일 후
	* 12개 데이터 포인트가 1시간 해상도로 편입
	* 이후 15개월간 유지 후 삭제

### 그래프 표현 및 시각화
CloudWatch는 다양한 시각화 도구를 제공

* **그래프 유형**:
	* 선 그래프
	* 막대 그래프
	* 숫자 위젯
	* 게이지 등

* **CloudWatch Dashboards**:
	* 여러 리전의 메트릭스를 하나의 화면에서 모니터링
	* 커스텀 대시보드 생성 가능
	* 팀원들과 공유 가능
	* 특정 URL을 통해 외부 공개 가능