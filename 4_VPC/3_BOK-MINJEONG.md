### CIDR
- Classless Inter-Domain Routing (클래스 없는 도메인 간 라우팅)
- IP 주소를 효율적으로 나누고 관리하는 방법
- IP 주소 범위를 나타내는 표기법으로 IP 개수를 정하는 역할을 함   
#### CIDR 블록 표기법
- A.B.C.D/숫자
- '/숫자'는 네트워크 부분의 비트 개수를 의미
- 남은 비트는 호스트(사용 가능한 IP) 개수를 결정    
![alt text](images/CIDR.png)
⚠️ AWS에서는 예약된 5개의 IP 주소가 있기 때문에 **실제 사용 가능한 IP는 "총 IP 개수 - 5"**   
> CIDR 블록을 **"서브넷 마스크"**로 생각하면 쉬움
> CIDR의 "/숫자" 부분은 네트워크와 호스트를 구분하는 비트 개수를 의미
> > e.g. /24는 앞 24비트가 네트워크 주소, 나머지 8비트가 호스트 주소
> > 즉, 남은 8비트로 2^8 = 256개의 IP를 만들 수 잇음
> > > **Network**: IP 주소를 묶는 그룹 (ex. 10.0.0.0/24 -> 10.0.0까지 같은 네트워크), 같은 그룹으로 묶이는 IP 대역   
> > > **Host**: 각 네트워크 안에서 개별적으로 사용되는 장치(서버, PC, 라우터)의 IP    
> 
> 💡 네트워크 비트가 크면 호스트 개수가 줄어들고, 호스트 비트가 크면 하나의 네트워크에서 사용할 수 있는 IP 개수가 많아짐   
> 💡 네트워크가 크면 더 많은 작은 서브넷을 만들 수 있고, 호스트가 크면 한 네트워크 안에서 더 많은 기기를 사용할 수 있음   
<br>

## VPC (Virtual Private Cloud)
- 모든 새로운 AWS 계정에는 default VPC가 있음
- 새로운 EC2 인스턴스는 서브넷을 지정하지 않으면 기본 VPC에 실행됨
- 기본 VPC는 처음부터 인터넷에 연결되어 있어서 인스턴스가 인터넷에 엑세스하고 내부의 EC2 인스턴스는 공용 IPv4 주소를 얻음 -> EC2 인스턴스를 생성하자마자 연결할 수 있는 이유   
<br>

- 단일 AWS 리전에 여러 VPC를 둘 수 있음
- VPC마다 할당된 CIDR는 최대 5개
  - Min.size is /28 (16 IP addresses)
  - Max.size is /16 (65536 IP addresses)
- VPC가 사설 리소스이기 때문에 사설 IPv4만 허용됨 
  - 10.0.0.0 - 10.255.255.255 (10.0.0.0/8)
  - 172.16.0.0 - 172.31.255.255 (172.16.0.0/12)
  - 192.168.0.0 - 192.168.255.255 (192.168.0.0/16)    
⚠️ Your VPC CIDR should NOT overlap with your other networks (e.g. corporate)   

### VPC - Subnet (IPv4)
- VPC 내부에 있는 IPv4 주소의 부분 범위 
- AWS가 각 서브넷마다 IP 주소 5개를 예약 (first 4 & last 1)
- 5 IP 주소는 사용 X, EC2 인스턴스에 할당 X
- e.g. CIDR block 10.0.0.0/24
  - 10.0.0.0 - Network Address (서브넷을 식별)
  - 10.0.0.1 - reserved by AWS for the VPC router (서브넷의 기본 게이트웨이)
  - 10.0.0.2 - reserved by AWS for mapping to Amazon-provided DNS
  - 10.0.0.3 - reserved by AWS for future use
  - 10.0.0.255 - Network Broadcast Address 
    - (AWS는 VPC에서 브로드캐스트를 지원하지 않기에 예약은 되지만 사용하는 건 안됨)   
> 🧐 Exam Tip, if you need 29 IP addresses for EC2 instances:
> - /27 서브넷은 사용할 수 없음 (/27 IP 주소는 32개인데, 32 - 5(예약) = 27 < 29)
> - 따라서 서브넷 사이즈는 /26이어야 함 (64 IP 주소, 64 - 5 = 59 > 29)   
<br>

## Internet Gateway (IGW)
- VPC와 인터넷을 연결하는 역할을 함. 즉, 인터넷과 VPC 간의 트래픽 흐름을 제어
- **public subnet에 있는 EC2 인스턴스와 인터넷 간의 연결을 제공**  
  *(라우팅 테이블을 통해 설정해주어야 EC2 인스턴스가 인터넷과 연결됨)*
- 인터넷 게이트웨이의 역할
  1. VPC 내에서 public subnet에 있는 EC2 인스턴스가 인터넷에 접근할 수 있도록 해줌
  2. VPC의 public subnet에 있는 EC2 인스턴스가 퍼블릭 IP를 가지면, 인터넷 게이트웨이를 통해 인터넷과 연결됨   
  
- 인터넷 게이트웨이 연결 과정
  1. 인터넷 게이트웨이는 VPC와 연결된다.
  2. public subnet에 있는 EC2 인스턴스는 라우팅 테이블에 인터넷 게이트웨이를 경우하는 경로가 설정되어 있어야 인터넷에 접근할 수 있음. 이를 위해 public subnet의 라우팅 테이블에 인터넷 게이트웨이로 가는 경로를 추가해야 함.   
  *(VPC > Route tables > rtb-0000 > Edit routes: 0.0.0.0/0 igw-0000000)*
  3. EC2 인스턴스가 인터넷과 연결될 수 있도록 public IP를 할당해야 하며, 라우팅 테이블을 통해 인터넷 게이트웨이로 트래픽을 전달할 수 있게 된다. 

- private subnet에 인터넷 연결
  - private subnet은 직접적으로 인터넷과 연결되지 않으며, **NAT 게이트웨이**나 **NAT 인스턴스**를 통해서만 인터넷과 연결된다. 이때, private subnet의 라우팅 테이블은 NAT 게이트웨이로 트래픽을 보내게 설정됨
<br>


## Bastion Hosts
![alt text](images/image-1.png)   
- **Bastion Host를 이용해 private EC2 인스턴스에 SSH로 액세스할 수 있음**
- user는 퍼블릭 인터넷에 있고 private subnet에 위치한 ec2 인스턴스에 접근하는 경우에 Bastion host를 사용할 수 있음   
- Bastion host == **public subnet에 있는** EC2 인스턴스
- 특징
  - Bastion host는 반드시 **public subnet**에 있어야 함
  - BastionHost Security Group이라는 자체 보안 그룹이 있음
  - 퍼블릭 서브넷에 있는 bastion host로 프라이빗 서브넷에 있는 EC2 인스턴스에 액세스 할 수 있음   
- 엑세스 방법   
  1. SSH를 Bastion Host에 연결
  2. 해당 Bastion Host가 다시 SSH를 private subnet의 EC2 인스턴스에 연결   

⚠️ **Bastion host를 위해서는 보안 그룹이 반드시 인터넷 액세스를 허용해야 함** (모든 인터넷을 허용하는 것이 아닌 기업의 퍼블릭 CIDR 액세스만 허용하거나 사용자의 인터넷 액세스만 허용하는 등 제한해야 함)   
-> bastion host의 EC2 보안 그룹을 최대한 제한하여 특정 IP만 액세스가 가능하도록 설정할 수 있음   

⚠️ **private subnet의 EC2 인스턴스 보안 그룹에서는 반드시 SSH 액세스를 허용해야 함**   

> Bastion host vs NAT Gateway
> 1. Bastion Host를 사용해서 private EC2에 접근
>   - bastion host는 프라이빗 서브넷에 있는 EC2 인스턴스에 SSH나 RDP로 접속할 수 있게 해주는 보안 중개 서버   
>   - 보안을 강화하기 위한 방법으로, 프라이빗 서브넷의 EC2를 직접 노출시키지 않고도 관리할 수 있게 해줌
>
> 2. NAT 게이트웨이를 사용하여 private EC2가 인터넷에 접근
>   - NAT 게이트웨이는 프라이빗 서브넷에 배치된 EC2 인스턴스가 인터넷과 통신할 수 있도록 도와주는 게이트웨이   
>   - 프라이빗 EC2는 아웃바운드 연결만 허용되고, 인터넷으로부터 인바운드 연결은 불가능   
>
> > 💡 Bastion Host는 SSH/RDP를 통해 EC2 관리를 위한 접근을 제공하고, NAT 게이트웨이는 인터넷 연결을 위한 네트워크 경로를 제공   
<br>

## NAT Instance (outdated)
- NAT = Network Address Translation (네트워크 주소 변환)
- Allows EC2 instances in private subnets to connect to the Internet
- 반드시 public subnet에서 실행되어야 함
- Must disable EC2 setting: Source/destination Check
- NAT 인스턴스에는 고정된 Elastic IP가 연결되어 있어야 함   
![alt text](images/image-2.png)   
-> 일부의 IP가 재작성되기 때문에 NAT 인스턴스를 위해 Source/destination Check 설정을 비활성화해야 되는 이유   

#### NAT Instance - Comments
- Pre-configured Amazon Linux AMI is available -> 표준 지원 종료
- 가용성이 좋지 않고 초기화 설정으로 복원할 수 없어서 여러 AZ에 ASG를 생성해야 하고 복원되는 사용자 데이터 스크립트가 필요함 -> 복잡
- 보안 그룹을 관리해야 함   
<br>

## NAT Gateway
- AWS의 관리형 NAT 인스턴스이며 높은 대역폭을 가지고 있음
- 사용량 및 NAT Gateway의 대역폭에 따라 청구됨
- 특정 AZ에서 생성되고 탄력적 IP를 사용
- EC2 인스턴스와 같은 서브넷에서 사용할 수 없어서 다른 서브넷에서 액세스할 때만 NAT Gateway가 도움 됨
- 경로: Private subnet > NAT GW > IGW
- 대역폭으느 초당 5GB, 자동으로 초당 45GB까지 확장할 수 있음
- 보안 그룹을 관리할 필요 X   

### NAT Gateway with High Availability
- NAT Gateway는 단일 AZ에서 복원 가능
- Mush create multiple NAT Gateways in multiple AZs for fault-tolerance
- **고가용성으로 구성하고 싶다면 다중 NAT Gateway를 만들어 전체 AZ에 영향을 주는 재난을 방지할 수 있음**
  ![alt text](images/image-4.png)
- 라우팅 테이블을 통해 AZ를 서로 연결하지 않음 -> 하나에 장애가 생기면 다 멈추니까   
<br>

## VPC Peering
- 여러 VPC를 생성하고 AWS 네트워크를 사용하여 함께 연결하고 싶은 상황, 같은 네트워크에 있는 것처럼 작동하게 만들고 싶을 때 사용
- **Must not have overlapping CIDRs** 
- VPC peering은 두 VPC 간에 발생할 수 있으며 **전이되지 않음**
  - e.g. A-B, B-C가 피어링 되어도 A-C가 통신하도록 하려면 반드시 VPC 피어링 연결을 해야함
- **VPC가 서로 쌍을 이루고 있더라도 서로 다른 VPC의 EC2 인스턴스가 서로 통신할 수 있도록 각 VPC 서브넷의 모든 라우팅 테이블을 업데이트 해야 함**   
- You can create VPC Peering connection between VPCs in **different AWS accounts/regions**
- You can reference a security group in a peered VPC (works cross accounts - same region)