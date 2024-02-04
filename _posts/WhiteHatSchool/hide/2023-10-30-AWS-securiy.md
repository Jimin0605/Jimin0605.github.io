---
title: "[WHS]AWS 관리 기초"
categories: [WhiteHatSchool, task]
mermaid: true
tag:
- whitehatschool
- 화이트햇스쿨
- preBoB
- AWS
- AWS관리
---

# 개요
이번 AWS 관리기초 과제에서 선택한 가이드항목은 AWS를 내가 처음 사용하기도 하고 AWS를 사용할 때 가장 기본적으로 적용해야하는 것들을 선택했다.
- 관리 콘솔 접근 시 MFA 적용
- 인스턴스 세부 정보에서 “EC2 인스턴스 연결” 기능 비활성화
- Access key 주기적 변경
-IAM 계정 비밀번호 복잡도 및 변경 주기 정책 설정

### 관리 콘솔 접근 시 MFA 적용
관리 콘솔을 접근하기 위해 로그인 시 2차인증인 MFA를 적용해 보안을 강화하는 것이다. 

#### 취약점 확인
- root 및 IAM 계정으로 관리 콘솔 로그인 시 MFA를 적용하지 않은 경우 **취약**
- root 및 IAM 계정으로 관리 콘솔 로그인 시 MFA를 적용한 경우 **취약하지 않음**

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/1.png)

root 및 IAM 계정으로 관리 콘솔 로그인 후 오른쪽 상단의 계정명 클릭 후 보안 자격 증명 클릭

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/2.png)

[멀티 팩터 인증(MFA)] 항목에서 MFA 할당 여부 확인

#### 취약점 조치 방법
![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/3.png)

root 또는 IAM 계정으로 관리 콘솔 로그인 후 오른쪽 상단의 계정명 클릭, **보안 자격 증명** 클릭, **멀티 팩터 인증(MFA)** 항목에서 **MFA 디바이스 할당**클릭


### 인스턴스 세부 정보에서 “EC2 인스턴스 연결” 기능 비활성화
#### 취약점 확인
- *[EC2 인스턴스 연결]* 기능이 활성화되어 있는 경우 **취약**
- *[EC2 인스턴스 연결]* 기능이 비활성화되어 있는 경우 **취약하지 않음**


EC2 서비스에서 인스턴스 -> 인스턴스 세부정보 -> 연결 클릭

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/4.png)

**EC2 인스턴스 연결** 탭의 **연결** 버튼 클릭 시 인스턴스 접속 가능 여부 확인

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/5.png)

#### 취약점 조치 방법
![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/6.png)

설치된 ec2-instance-connect 패키지 삭제

```sh
# 데비안 계열
sudo dpkg -r ec2-instance-connect 

# 레드햇 계열
sudo yum remove ec2-instance-connect
```

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/7.png)

접속 불가 확인


### Access key 주기적 변경
AWS 리소스 핸들링을 위해 IAM 계정에 Access key(Access key ID/Secret access key)를 발급한 경우 주기적으로 변경 필요하다.

#### 취약점 확인
- Access key 변경 주기를 지정하지 않고, 주기적으로 Access key를 변경하지 않은 경우 **취약**
- Access key 변경 주기를 지정하고, 주기적으로 Access key를 변경한 경우 **취약하지 않음**

- 관리 콘솔에서 *[IAM]* 검색 → *[사용자]* 메뉴 클릭
- *[활성 키 수명(Active key age)]*이 내부 정책에서 정의한 Access key 변경 주기를 초과하는지 여부 확인

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/8.png)

#### 취약점 조치 방법
![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/9.png)

관리콘솔에서 Secret Manager검색 후 새 보안 암호 저장 클릭

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/10.png)

Access Key를 저장할 Secret Manager생성(키/값에는 임시 값 입력)

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/11.png)

자동 교체 구성 비활성화

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/12.png)

관리콘솔에서 IAM검색, 정책 메뉴, 정책 생성 클릭

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/13.png)

IAM 서비스에서 역할 메뉴, 역할 만들기 클릭

json을 활용해 json정책 생성
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SecretsManagerReadWritePermission",
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue",
                "secretsmanager:DescribeSecret",
                "secretsmanager:PutSecretValue",
                "secretsmanager:UpdateSecretVersionStage",
                "secretsmanager:ListSecretVersionIds"
            ],
            "Resource": [
                "SECRETS MANAGER의 ARN"
            ]
        },
        {
            "Sid": "KMSWritePermission",
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt",
                "kms:Encrypt"
            ],
            "Resource": [
                "aws/secretsmanager의 ARN"
            ]
        },
        {
            "Sid": "SecretsManagerKMSListingPermission",
            "Effect": "Allow",
            "Action": [
                "kms:ListKeys",
                "secretsmanager:ListSecrets"
            ],
            "Resource": "*"
        }
    ]
}
```

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/14.png)

Json을 활용해 만든 정책과 AWSLambdaBasicExecutionRole 정책 할당

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/15.png)

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/16.png)

관리 콘솔에서 Lambda검색, 함수 생성 클릭

<br>

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/17.png)

함수 생성 설정, Lambda을 활용해 함수 작성, Deploy클릭

```python
import boto3 
import json 
import logging 
import os 
import time 

logger = logging.getLogger() 
logger.setLevel(logging.INFO) 


def lambda_handler(event, context): 
	arn = event['SecretId'] 
	token = event['ClientRequestToken'] 
	step = event['Step'] 
	# Setup the client 
	secretsmanager_client = boto3.client('secretsmanager') 
	# Make sure the version is staged correctly 
	metadata = secretsmanager_client.describe_secret(SecretId=arn) 
	logging.info(repr(metadata)) 
	versions = metadata['VersionIdsToStages'] 
	if token not in versions: 
		logger.error("Secret version %s has no stage for rotation of secret %s." % (token, arn)) 
		raise ValueError("Secret version %s has no stage for rotation of secret %s." % (token, arn)) 
	if "AWSCURRENT" in versions[token]: 
		logger.info("Secret version %s already set as AWSCURRENT for secret %s." % (token, arn)) 
		return 
	elif "AWSPENDING" not in versions[token]: 
		logger.error("Secret version %s not set as AWSPENDING for rotation of secret %s." % (token, arn)) 
		raise ValueError("Secret version %s not set as AWSPENDING for rotation of secret %s." % (token, arn)) 
	if step == "createSecret": 
		logging.debug("createSecret %s" % arn) 
		logging.info("for IAM user access keys secret creation is handled by IAM ") 
	elif step == "setSecret": 
		logging.debug("setSecret %s" % arn) 
		current_dict = get_secret_dict(secretsmanager_client, arn, "AWSCURRENT", required_fields=['username']) 
		username = current_dict['username'] 
		master_dict = get_secret_dict(secretsmanager_client, current_dict['masterarn'], "AWSCURRENT") 
		master_iam_client = boto3.client('iam', aws_access_key_id=master_dict['accesskey'], aws_secret_access_key=master_dict['secretkey']) 
		# load any pre-existing access keys. sorted by created descending. if the count is 2+ remove the oldest key 
		existing_access_keys = sorted(master_iam_client.list_access_keys(UserName=username)['AccessKeyMetadata'], key=lambda x: x['CreateDate']) 
		if len(existing_access_keys) >= 2: 
			logger.info("at least 2 access keys already exist. deleting the oldest version: %s" % existing_access_keys[0]['AccessKeyId']) 
			master_iam_client.delete_access_key(UserName=username, AccessKeyId=existing_access_keys[0]['AccessKeyId']) 
		# request new access key and gather the response 
		new_access_key = master_iam_client.create_access_key(UserName=username) 
		current_dict['accesskey'] = new_access_key['AccessKey']['AccessKeyId'] 
		current_dict['secretkey'] = new_access_key['AccessKey']['SecretAccessKey'] 
		logging.info('applying new secret value to AWSPENDING') 
		# save the new access key to the pending secret 
		secretsmanager_client.put_secret_value(SecretId=arn, ClientRequestToken=token, SecretString=json.dumps(current_dict), VersionStages=['AWSPENDING']) 
	elif step == "testSecret": 
		logging.debug("testSecret %s" % arn) 
		# load the pending secret for testing 
		pending_dict = get_secret_dict(secretsmanager_client, arn, "AWSPENDING", required_fields=['username'], token = token) 
		# attempt to call an iam service using the credentials 
		test_client = boto3.client('iam', aws_access_key_id=pending_dict['accesskey'], aws_secret_access_key=pending_dict['secretkey']) 
		try: 
			test_client.get_account_authorization_details() 
		except test_client.exceptions.ClientError as e: 
			# the test fails if and only if Authentication fails. Authorization failures are acceptable. 
			if e.response['Error']['Code'] == 'AuthFailure': 
				logging.error("Pending IAM secret %s in rotation %s failed the test to authenticate. exception: %s" % (arn, pending_dict['username'], repr(e))) 
				raise ValueError("Pending IAM secret %s in rotation %s failed the test to authenticate. exception: %s" % (arn, pending_dict['username'], repr(e))) 
	elif step == "finishSecret": 
		logging.debug("finishSecret %s" % arn) 
		# finalize the rotation process by marking the secret version passed in as the AWSCURRENT secret. 
		metadata = secretsmanager_client.describe_secret(SecretId=arn) 
		current_version = None 
		for version in metadata["VersionIdsToStages"]: 
			if "AWSCURRENT" in metadata["VersionIdsToStages"][version]: 
				if version == token: 
					# The correct version is already marked as current, return 
					logger.info("finishSecret: Version %s already marked as AWSCURRENT for %s" % (version, arn)) 
					return 
				current_version = version 
				break 
		# finalize by staging the secret version current 
		secretsmanager_client.update_secret_version_stage(SecretId=arn, VersionStage="AWSCURRENT", MoveToVersionId=token, RemoveFromVersionId=current_version) 
		logger.info("finishSecret: Successfully set AWSCURRENT stage to version %s for secret %s." % (token, arn)) 
	else: 
		raise ValueError("Invalid step parameter") 


def get_secret_dict(secretsmanager_client, arn, stage, required_fields=[], token=None): 
	# Only do VersionId validation against the stage if a token is passed in 
	if token: 
		secret = secretsmanager_client.get_secret_value(SecretId=arn, VersionId=token, VersionStage=stage) 
	else: 
		secret = secretsmanager_client.get_secret_value(SecretId=arn, VersionStage=stage) 
	plaintext = secret['SecretString'] 
	secret_dict = json.loads(plaintext) 
	# Run validations against the secret 
	for field in required_fields: 
		if field not in secret_dict: 
			raise KeyError("%s key is missing from secret JSON" % field) 
	# Parse and return the secret JSON string 
	return secret_dict
```

JSON(2)를 활용해 IAM 계정이 자신의 패스워드와 Access Key를 변경하도록 하는 정책 생성
```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "iam:ListUsers",
                        "iam:GetAccountPasswordPolicy"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "iam:*AccessKey*",
                        "iam:ChangePassword",
                        "iam:GetUser",
                        "iam:*ServiceSpecificCredential*",
                        "iam:*SigningCertificate*"
                    ],
                    "Resource": [
                        "arn:aws:iam::*:user/${aws:username}"
                    ]
                }
            ]
        }
```
        
Access Key를 자동으로 교체할 IAM 계정에 JSON(2)를 활용해 만든 정책 할당

AWSLambda_FullAccess 정책을 할당한 IAM계정의 AWS CLI접속후 명령어 실행
```bash
aws lambda add-permission --function-name [lambda의 ARN] --principal secretsmanager.amazonaws.com --action lambda:InvokeFunction --statement-id SecretsManagerAccess
```


![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/18.png)

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/19.png)

관리콘솔에서 Secret Manager검색, 생성한 보안 암호 클릭, 보안 함호값검색 클릭, 편집 클릭

보안 암호값 편집

- accesskey: Access key를 변경하고자 하는 IAM계정의 Access Key값
- secretkey: Access key를 변경하고자 하는 IAM 계정의 Secret Access Key값
- username:Access key를 변경하고자 하는 IAM계정명
- masterarn: 해당 Secret Manager의 ARN

교체편집 클릭, 교체구성 설정

![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/20.png)

Secret Manager에서 Access Key변경 확인

### IAM 계정 비밀번호 복잡도 및 변경 주기 정책 설정
관리 콘솔에 접근 가능한 IAM 계정의 비밀번호 복잡도 및 변경 주기 정책을 설정하여 brute force 등 계정 탈취 공격에 대한 보호 필요

#### 취약점 확인
- 영문ㆍ숫자ㆍ특수문자 중 2종류를 조합하여 10자리 이상 또는 3종류를 조합하여 8자리 이상의 길이로 비밀번호를 구성하지 않거나, 분기별 1회 이상 비밀번호를 변경하지 않는 경우 **취약**
- 영문ㆍ숫자ㆍ특수문자 중 2종류를 조합하여 10자리 이상 또는 3종류를 조합하여 8자리 이상의 길이로 비밀번호를 구성하고, 분기별 1회 이상 비밀번호를 변경하는 경우 **취약하지 않음**

- IAM서비스 권한을 보유한 IAM계정으로 로그인
- 서비스에서 IAM검색, 계정 설정 메뉴 클릭
- 비밀번호 정책 항목에서 비밀번호 복잡도 적용 여부 확인


![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/21.png)



#### 취약점 조치 방법
![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/22.png)
암호 정책에서 편집클릭


![image](https://Jimin0605.github.io/assets/img/WHS/AWS%ea%b8%b0%eb%b3%b8%ea%b4%80%eb%a6%ac/23.png)
사용자지정 선택후 위처럼 요구사항 선택

## 선택과제
내가 좋아하는 분야는 offensive security로 이에 해당하는 직문인 모의해킹, 레드팀 등의 직무를 갖고싶다. 화이트햇스쿨 이전 국정원에서 주관한 윤리적해커양성이라는 교육을 들은 후 국정원에 대한 목표가생겼다. 해당 목표를 달성하기 위해 2가지의 길을 선택중이다. 하나는 대학졸업후 바로 공채로 들어가기, 다른 회사에서 경력을 채운 뒤 경력직으로 들어가기.

## Reference
[https://rogue-gouda-f87.notion.site/MFA-e9289520fb5640d3aa885d43b353beaa](https://rogue-gouda-f87.notion.site/MFA-e9289520fb5640d3aa885d43b353beaa)
