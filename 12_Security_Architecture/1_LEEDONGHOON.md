# 12. 신분 및 접근 권한 관리

## AWS 자격인증 정보 보호

AWS 리소스에 접근하려는 사람은 자격인증 정보를 이용해 접근 권한을 확인받아야 함

- 리소스에 접근하려는 주체 = `principal`
    - 루트 유저 : AWS 계정에 대한 모든 접근 권한을 가짐
        
        ```json
        "Principal": { "AWS": "arn:aws:iam::123456789012:root" }
        ```
        
        - 새 IAM 유저를 만들고 Administrator Access 권한을 부여해 관리자 계정으로 사용하는 것을 권장
        - MFA (MFA 디바이스 분실 시 루트 계정으로 접근이 안되니 조심)
    - IAM 유저
        
        ```json
        "Principal": { "AWS": "arn:aws:iam::AWS-account-ID:user/user-name" }
        ```
        
        - 콘솔에서 접근하기 위한 패스워드 정책을 강화해 보안 수준을 높일 수 있음
        - access key ID와 secret key가 필요
    - IAM 롤 : **특정 작업을 하기 위해 잠시 빌리는 권한 집합**
        
        ```json
        "Principal": { "AWS": "arn:aws:iam::AWS-account-ID:role/role-name" }
        ```
        
        - **임시 보안 자격 증명** 사용
        - IAM 유저 또는 AWS 리소스는 롤을 지닐 수 있고, 연계된 권한을 상속받음
        - 액세스 키 없이 특정 AWS 리소스에 대한 접근 권한을 부여할 때 편리하게 이용할 수 있음
        - 어떤 리소스에 대한 접근이 필요하면 해당 애플리케이션에 롤을 추가해 접근 권한을 부여할 수 있음
    - AWS 서비스(Service)
        
        ```json
        "Principal": {
            "Service": [
                "ecs.amazonaws.com",
                "elasticloadbalancing.amazonaws.com"
           ]
        }
        ```
        

## 세분화된 권한 부여

정보보안의 근본적 개념 : `최소 권한 부여 원칙` 

- 모든 개체의 접근을 허용하지 않음
- 새로 생성한 IAM 유저 또는 롤은 정책을 통해 명시적으로 권한을 부여받아야 함
- 어드민은 유저에게 하나 이상의 정책을 작성해 principal에 명시하는 방식으로 권한을 부여
    - 정책에는 하나 이상의 permission(권한)이 포함되며,
        - Effect
        - Action
        - Resource
        - Condition
    
    을 가짐
    

## ID 기반 정책

- **정의**: IAM 사용자, 그룹, 역할 같은 `신원(Identity)`에 연결되는 정책
- **부착 대상**: IAM User, IAM Group, IAM Role

### AWS 관리형 정책

: AWS가 만든 정책

![image.png](12%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%87%E1%85%AE%E1%86%AB%20%E1%84%86%E1%85%B5%E1%86%BE%20%E1%84%8C%E1%85%A5%E1%86%B8%E1%84%80%E1%85%B3%E1%86%AB%20%E1%84%80%E1%85%AF%E1%86%AB%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%201ffcb0a994c58046bda8f5c2be27cb45/image.png)

- **장점**
    - 빠르고 간편하게 사용할 수 있음
    - AWS에서 유지보수하므로 항상 최신 권한 구조 반영
- **단점**
    - 수정 불가 (읽기 전용)
    - 너무 포괄적이거나 서비스 단위로 권한 범위가 큼

### 고객 관리형 정책

: 사용자가 직접 정의한 정책

![image.png](12%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%87%E1%85%AE%E1%86%AB%20%E1%84%86%E1%85%B5%E1%86%BE%20%E1%84%8C%E1%85%A5%E1%86%B8%E1%84%80%E1%85%B3%E1%86%AB%20%E1%84%80%E1%85%AF%E1%86%AB%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%201ffcb0a994c58046bda8f5c2be27cb45/image%201.png)

- **장점**
    - 특정 액션, 리소스, 조건 기반 권한 등 **정밀한 제어 가능**
    - 필요시 수정 가능
- **단점**
    - 설계와 유지보수 필요

### 인라인 정책

: 단일 사용자, 그룹, 역할(Role)에 직접 추가하는 방식

- 특정 권한 개체의 일부로만 존재할 수 있음
- 특정 정책을 매우 구체적인 대상에게 확실하게 적용해야 할 때 유용
- 정책이 둘 이상의 엔터티에 적용될 수 있는 경우 관리형 정책을 사용하는 것을 권장

![image.png](12%20%E1%84%89%E1%85%B5%E1%86%AB%E1%84%87%E1%85%AE%E1%86%AB%20%E1%84%86%E1%85%B5%E1%86%BE%20%E1%84%8C%E1%85%A5%E1%86%B8%E1%84%80%E1%85%B3%E1%86%AB%20%E1%84%80%E1%85%AF%E1%86%AB%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%201ffcb0a994c58046bda8f5c2be27cb45/image%202.png)

## **리소스 기반 정책**

- **정의**: S3 버킷, Lambda, SNS 등 **리소스에 직접 연결되는 정책**
- **부착 대상**: S3, SQS, SNS, Lambda 등

```json
{
    "Version": "2012-10-17",
    "Id": "Policy1746602846804",
    "Statement": [
        {
            "Sid": "Stmt1746602836691",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::ggzz-img/*"
        }
    ]
}
```

---

## 권한의 경계

- 퍼미션 정책 : 특정 권한 개체의 접근을 허용하는 정책
- 다수의 퍼미션 정책을 부여 ⇒ 특정 권한 개체에게 지나치게 많은 권한을 부여되는 것을 막는 장치로서 의미가 있음
    1. **Permissions Boundary 정책**을 먼저 설정
        
        ex. EC2에 대해서만 모든 작업 허용 (ec2:*)
        
    2. 이후 해당 유저에게 다른 퍼미션 정책을 붙여도,
        
        **ex. AdministratorAccess** 전체 권한 부여
        
    3. 실제 유저가 할 수 있는 작업은
        
        **⇒ 두 정책의 교집합**, 즉 **EC2 관련 작업만 허용**
        
- 특정 권한 개체에 대한 최대 권한을 정의한 관리형 정책을 선택하는 방식으로 간단하게 설정 가능

ex. EC2 서비스에 대한 모든 액션을 허용하는 정책을 생성한 뒤 이를 IAM 유저에게 부착해 권한의 경계로 사용할 수 있음

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"ec2:*"
			],
			"Resource":
		}
	]
}
```

<aside>
💡

권한의 경계 적용을 받는 유저는 더 높은 수준의 권한을 부여받더라도 권한의 경계를 벗어난 작업은 할 수 없음

</aside>

## Role (~을 갖는 주체)

: 어느 리소스가 다른 리소스에 접근할 권한을 부여할 때 사용

ex. 인스턴스가 DynamoDB에 접근이 필요

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:CreateTable",
        "dynamodb:PutItem",
        "dynamodb:ListTables",
        "dynamodb:DescribeTable",
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:Query",
        "dynamodb:UpdateItem",
        "dynamodb:UpdateTable"
      ],
      "Resource": "*"
    }
  ]
}
```

## 인스턴스 프로필

: 트러스트 정책을 통해 어떤 AWS 리소스가 해당 롤을 지니고 있는지 설명 가능

<aside>
💡

`트러스트 정책` :  IAM Role을 누가 사용할 수 있는가? 

- 누가 사용할지 모를 수 있기 때문에 “누가 사용할 수 있는지”를 정의해야 함

`퍼미션 정책` : 무엇을 할 수 있는가 (IAM User)

- 본인이 principal이기 때문에 누가 사용할 지를 따로 정의를 할 필요가 없음
</aside>

ex. DynamoDB에 접근해야 하는 애플리케이션은 EC2 인스턴스에서 실행 중이므로 트러스트 정책을 통해 해당 롤에 EC2 인스턴스에 대한 퍼미션이 있다는 사실을 증명하는 트러스트 정책

```json
{
  "Version": "2012-10-17",
  "Statement": [
	  {
	      "Effect": "Allow",
	      "Principal": {
	        "Service": "ec2.amazonaws.com"
	      },
	      "Action": "sts:AssumeRole"
    }
  ]
}
```

- EC2 인스턴스는 직접 IAM Role을 붙일 수 없는데, 그래서 IAM Role을 담을 수 있는 인스턴스 프로필이 필요
- EC2에 붙는 건 이 인스턴스 프로필이고, 내부에 있는 IAM Role이 실질적으로 권한을 작동시킴
- 인스턴스 프로필과 이번 롤이 연결된 것을 확인

```json
{
  "InstanceProfiles": [
    {
      "InstanceProfileId": "AIPA:IGKR6VR52WTHHX0R6",
      "InstanceProfileName": "MyAppRole",
      "Path": "/",
      "Arn": "arn:aws:iam::xxxxxxxxxxxx:instance-profile/MyAppRole",
      "CreateDate": "2018-10-13T03:20:46Z",
      "Roles": [
        {
          "RoleId": "AROAIV2GSBS4E4HAEFTJA",
          "RoleName": "MyAppRole",
          "Path": "/",
          "Arn": "arn:aws:iam::xxxxxxxxxxxx:role/MyAppRole",
          "CreateDate": "2018-10-13T03:20:46Z",
          "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
              {
                "Effect": "Allow",
                "Principal": {
                  "Service": "ec2.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
              }
            ]
          }
        }
      ]
    }
  ]
}
```

- 인스턴스와 인스턴스 프로필이 연결되면 AWS STS(Security Token Service)가 자동으로 **임시 자격 증명**(Access Key, Secret Key, Session Token)을 발급
    - `"Action": "sts:AssumeRole"`
- 이 자격 증명은 EC2 인스턴스 내에서 아래 URL을 통해 확인 가능
    - `http://169.254.169.254/latest/meta-data/iam/security-credentials/{RoleName}`