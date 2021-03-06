# Python Serverless 기초

Python과 AWS Lambda 를 사용한 crawler 만들기 입니다.

[변규현님(novemberde) Github](https://github.com/novemberde) 많이 참고하였습니다.

**변규현님께 감사드립니다.**

## Objective

Amazon Web Service 를 활용하여 Serverless architecture로 웹크롤러를 배포합니다.

크롤링된 데이터는 DynamoDB에 저장합니다.

## AWS Resources

AWS에서 사용하는 리소스는 다음과 같습니다.

- Cloud9: 코드 작성, 실행 및 디버깅을 위한 클라우드 기반 IDE.
- Lambda: 서버를 프로비저닝하거나 관리하지 않고도 코드를 실행할 수 있게 해주는 컴퓨팅 서비스. 서버리스 아키텍쳐의 핵심 서비스.
- DynamoDB: 완벽하게 관리되는 NoSQL 데이터베이스 서비스로, 원활한 확장성과 함께 빠르고 예측 가능한 성능을 제공.

## Cloud 9 시작하기

Cloud9 은 하나의 IDE입니다. 그렇지만 이전의 설치형 IDE와는 다릅니다. 설치형 IDE는 로컬 PC에 프로그램을 설치하던가
실행하는 방식이었다면, Cloud9은 브라우저가 실행가능한 모든 OS에서 사용이 가능합니다.

맨 처음 Cloud9은 AWS 내에서가 아닌 별도의 서비스로 제공되었습니다. AWS에 인수된 이후 Cloud9은 AWS의 Managed Service형태로 바뀌었고,
AWS의 서비스와 결합하여 사용이 가능해졌습니다. 코드 편집과 명령줄 지원 등의 평범한 IDE 기능을 지니고 있던 반면에, 현재는 AWS 서비스와
결합되어 직접 Lambda 코드를 배포하던가, 실제로 Cloud9이 실행되고 있는 EC2의 컴퓨팅 성능을 향상시켜서
로컬 PC의 사양에 종속되지 않은 개발을 할 수가 있습니다.

그러면 Cloud9 환경을 시작해봅시다.

[Cloud 9 Console](https://ap-southeast-1.console.aws.amazon.com/cloud9/home?region=ap-southeast-1#)에 접속합니다.

아래와 같은 화면에서 [Create Environment](https://ap-southeast-1.console.aws.amazon.com/cloud9/home/create) 버튼을 누릅니다.

![c9-create](/images/c9-create.png)

Name과 Description을 다음과 같이 입력합니다.

- Name: PythonServerless


![c9-create-name](/images/c9-create-name.png)

Configure Setting은 다음과 같이 합니다.

- Environment Type: EC2
- Instance Type: T2.micro
- Cost Save Setting: After 30 minutes
- Network Settings: Default

![c9-conf](/images/c9-conf.png)

모든 설정을 마쳤다면 Cloud9 Environment를 생성하고 Open IDE를 통해 개발 환경에 접속합니다.

접속하면 다음과 같은 화면을 볼 수 있습니다.

1. 현재 Environment name
2. EC2에서 명령어를 입력할 수 있는 Terminal
3. Lambda Functions
    - Local Functions: 배포되지 않은 편집중인 Functions
    - Remote Functions: 현재 설정해놓은 Region에 배포된 Lambda Functions
4. Preferences

![c9-env](/images/c9-env.png)

현재 ap-southeast-1 region에 Cloud9 Environment를 배포했으므로 Default Region이 ap-southeast-1으로 되어 있습니다.
Preferences(설정 화면)에서 ap-northeast-2(Seoul Region)으로 바꾸어줍니다.

- Preferences > AWS Settings > Region > Asia Pacific(Seoul)

![c9-region](/images/c9-region.png)

## Python 설정

### virtualenv 설정
Cloud9 설정을 마친 다음 python version 관리를 합니다.

Python version 관리는 [virtualenv](https://virtualenv.pypa.io/) 라는 툴을 사용합니다.

Cloud9 에서는 python2.7 을 기본으로 alias 하고 있기 때문에 python3을 사용하기 위해 unalias 합니다.

**또한 AWS amazon linux 에서 PYTHON_INSTALL_LAYOUT 이라는 것을 설정하였는데 이부분을 해제해 줍니다.**


```sh
$ unset PYTHON_INSTALL_LAYOUT
$ unalias python
$ virtualenv -p python3 venv
Running virtualenv with interpreter /usr/bin/python3
Using base prefix '/usr'
New python executable in /home/ec2-user/environment/venv/bin/python3
Also creating executable in /home/ec2-user/environment/venv/bin/python
Installing setuptools, pip, wheel...done.
$ . venv/bin/activate
(venv) $ python
Python 3.6.5 (default, Apr 26 2018, 00:14:31)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-11)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

python version 설정을 완료하였습니다.

### ipython 을 이용한 console 개발 환경 설정
virtualenv 로 개발 환경을 설정한 이후 개발의 편의를 위해 `ipython` 을 설치합니다.

```sh
(venv) $ pip install ipython
Collecting ipython
...
Installing collected packages: ipython
Successfully installed ipython-7.0.1
(venv) $ ipython
Python 3.6.5 (default, Apr 26 2018, 00:14:31)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.0.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]:
```



## [Zappa - Serverless Python Web Services](https://www.zappa.io/)



![Zappa main](/images/zappa-main.png)

Zappa는 AWS Lambda serverless 를 위한 python framework 입니다.

> Zappa makes it super easy to build and deploy server-less, event-driven Python applications (including, but not limited to, WSGI web apps) on AWS Lambda + API Gateway.
> Think of it as "serverless" web hosting for your Python apps.
> That means infinite scaling, zero downtime, zero maintenance - and at a fraction of the cost of your current deployments!

Zappa는 python이 설치되어 있는 환경에서 사용할 수 있습니다.

open source로 기여하고 싶다면 [https://github.com/Miserlou/Zappa](https://github.com/Miserlou/Zappa)에서 issue와 pull request를 등록해주세요.


### Zappa 살펴보기

Zappa를 사용하기 위해서 명령어들을 살펴봅시다.

위에서 설정한 virtualenv 환경에서 작업합니다.

```sh
# Zappa 설치
(venv) $ pip install zappa
Successfully installed PyYAML-3.12 Unidecode-1.0.22 ... zappa-0.46.2

# 명령어들을 확인해봅니다.
(venv) $ zappa --help
usage: zappa [-h] [-v] [--color {auto,never,always}]
             {certify,deploy,init,package,template,invoke,manage,rollback,schedule,status,tail,undeploy,unschedule,update,shell}
             ...

Zappa - Deploy Python applications to AWS Lambda and API Gateway.

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         Print the zappa version
  --color {auto,never,always}

subcommands:
  {certify,deploy,init,package,template,invoke,manage,rollback,schedule,status,tail,undeploy,unschedule,update,shell}
    certify             Create and install SSL certificate
    deploy              Deploy application.
    init                Initialize Zappa app.
    package             Build the application zip package locally.
    template            Create a CloudFormation template for this API Gateway.
    invoke              Invoke remote function.
    manage              Invoke remote Django manage.py commands.
    rollback            Rollback deployed code to a previous version.
    schedule            Schedule functions to occur at regular intervals.
    status              Show deployment status and event schedules.
    tail                Tail deployment logs.
    undeploy            Undeploy application.
    unschedule          Unschedule functions.
    update              Update deployed application.
    shell               A debug shell with a loaded Zappa object.
```

여기서 자주 사용하게 될 명령어는 다음과 같습니다.

- init: 프로젝트 생성시 사용
- deploy: 첫 배포할 때 사용
- update: deploy 이 후 변경시 사용
- package: 배포될 패키지의 구조를 보고싶을 때 사용
- invoke: 특정 handler를 동작시킬 때 사용
- undeploy: 배포된 리소스를 제거할 때 사용

간단하게 zappa init 명령어를 확인해 봅니다. deploy 명령어는 추후에 사용하겠습니다.

```sh
# init 으로 기본 설정
(venv) $ zappa init

███████╗ █████╗ ██████╗ ██████╗  █████╗
╚══███╔╝██╔══██╗██╔══██╗██╔══██╗██╔══██╗
  ███╔╝ ███████║██████╔╝██████╔╝███████║
 ███╔╝  ██╔══██║██╔═══╝ ██╔═══╝ ██╔══██║
███████╗██║  ██║██║     ██║     ██║  ██║
╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝  ╚═╝

Welcome to Zappa!

Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!

Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'): dev

AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
Okay, using profile default!

Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
What do you want to call your bucket? (default 'zappa-b2z0giw4k'): USERNAME-serverless-demo

What's the modular path to your app's function?
This will likely be something like 'your_module.app'.
Where is your app's function?: crawler

You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: n

Okay, here's your zappa_settings.json:

{
    "dev": {
        "app_function": "crawler",
        "aws_region": "ap-northeast-2",
        "profile_name": "default",
        "project_name": "serverless-craw",
        "runtime": "python3.6",
        "s3_bucket": "USERNAME-serverless-demo"
    }
}

Does this look okay? (default 'y') [y/n]: y

Done! Now you can deploy your Zappa application by executing:

        $ zappa deploy dev

After that, you can update your application code with:

        $ zappa update dev

To learn more, check out our project page on GitHub here: https://github.com/Miserlou/Zappa
and stop by our Slack channel here: https://slack.zappa.io

Enjoy!,
 ~ Team Zappa!


zappa_settings.json
```


zappa_settings.json 파일을 확인하면 기본적인 zappa 설정 내용을 확인할 수 있습니다.

자세한 설정 내용은 [zappa github](https://github.com/Miserlou/Zappa) 을 통해 확인하면 알 수 있습니다.


## IAM Policy & Role 설정

Zappa는 deploy 할 때 zappa 가 AWS IAM policy와 role을 자동으로 생성하는데, 오늘 실습하는 Cloud9 환경에서는 `Zappa create role` 권한 제한이 있어, manual 하게 생성합니다.

Local 환경에서 개발하시면 이 부분은 자동 생성되기 때문에 넘어가셔도 좋습니다. (이 때 zappa settings은 [이 파일](https://github.com/seunghokimj/python-serverless-demo/blob/master/zappa_settings.local.json)을 참조합니다.)

```
(venv) $ zappa deploy dev
Calling deploy for stage dev..
Creating serverless-craw-dev-ZappaLambdaExecutionRole IAM Role..
Error: Failed to manage IAM roles!
You may lack the necessary AWS permissions to automatically manage a Zappa execution role.
To fix this, see here: https://github.com/Miserlou/Zappa#using-custom-aws-iam-roles-and-policies
```

위와 같이 권한이 없어 에러가 발생합니다.


### IAM Policy(정책) 설정
[IAM 정책 생성 Console](https://console.aws.amazon.com/iam/home?#/policies$new?step=edit) 에 접속합니다.

![iam-create-policy-1](/images/iam-create-policy-1.png)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:*"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "lambda:InvokeFunction"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:DescribeTable",
                "dynamodb:Query",
                "dynamodb:Scan",
                "dynamodb:GetItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem"
            ],
            "Resource": "arn:aws:dynamodb:*:*:*"
        }
    ]
}


```

JSON 탭을 선택하고 위 json 을 입력 후 review 버튼을 누릅니다.

![iam-create-policy-2](/images/iam-create-policy-2.png)

정책 이름을 python-serverless-crawler-policy 로 입력 후 `create policy` 버튼을 누릅니다.

### IAM Role(역할) 설정
`python-serverless-crawler-policy` 라는 정책을 가지는 역할을 생성합니다.

[IAM 역할 생성 Console](https://console.aws.amazon.com/iam/home?#/roles$new?step=type) 에 접속합니다.

![iam-create-role-1](/images/iam-create-role-1.png)

Lambda를 선택 후, `다음` 버튼을 누릅니다.

![iam-create-role-2](/images/iam-create-role-2.png)

`python-serverless-crawler-policy` 정책을 선택 후 `다음:검토` 버튼을 누릅니다.

![iam-create-role-3](/images/iam-create-role-3.png)

역할 이름을 `PythonServerlessCrawlerRole` 로 입력 후 `역할 만들기` 버튼을 누릅니다.


[IAM 역할 신뢰관계 수정 Console](https://console.aws.amazon.com/iam/home?#/roles/PythonServerlessCrawlerRole?section=trust) 에 접속합니다.

![iam-edit-role-1](/images/iam-edit-role-1.png)

신뢰관계를 편집합니다.

![iam-edit-role-2](/images/iam-edit-role-2.png)

아래 json 을 입력하여 `신뢰 정책 업데이트` 버튼을 누릅니다.
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "apigateway.amazonaws.com",
          "lambda.amazonaws.com",
          "events.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

IAM policy와 role 설정을 완료 하였습니다.


## S3 Bucket 생성하기

S3는 Object Storage로 쉽게 설명하자면 하나의 저장소입니다. 파일들을 업로드 / 다운로드 할 수 있으며 AWS에서 핵심적인 서비스 중 하나입니다.
여러 방면으로 활용할 수 있지만 여기서는 소스코드와 관련 라이브러리의 저장소 역할을 합니다.

S3의 메인으로 가서 버킷 생성하기 버튼을 클릭합니다.

![s3-create-btn.png](/images/s3-create-btn.png)

아래와 같이 입력하고 생성버튼을 클릭합니다.

- 버킷 이름(Bucket name): USERNAME-serverless-demo   // 여기서 USERNAME을 수정합니다. ex) seungho-serverless-demo
- 리전(Region): 아시아 태평양(서울)

![s3-create-btn.png](/images/s3-create-1.png)

## DynamoDB 테이블 생성하기

DynamoDB를 설계할 시 주의해야할 점은 [FAQ](https://aws.amazon.com/ko/dynamodb/faqs/)를 참고하시길 바랍니다.

이제 DynamoDB에 Todo table을 생성할 것입니다. 파티션 키와 정렬 키는 다음과 같이 설정합니다.

- 파티션키(Partition Key): portal
- 정렬키(Sort Key): createdAt

그럼 [DynamoDB Console](https://ap-northeast-2.console.aws.amazon.com/dynamodb/home?region=ap-northeast-2#)로 이동합니다.
테이블 만들기를 클릭하여 아래와 같이 테이블을 생성합니다.

![dynamodb-create](/images/dynamodb-create.png)


## Python 크롤링 시작하기
이제 부터는 cloud9을 사용합니다.

파일 트리는 다음과 같습니다.

```txt
environment
└── serverless-crawler  : Crawler
    ├── crawler.py  : Lambda에서 trigger하기 위한 handler가 포한됨 파일
    ├── lambda_test.py: lambda function test
    ├── zappa_settings.json : Zappa config file
    └── requirements.txt : 개발을 위하 필요한 library 정보
```

먼저 터미널을 열어 serverless-crawler 디렉터리를 생성하고 zappa 초기화를 시켜줍니다.

```sh
(venv) ec2-user:~/environment $ mkdir serverless-crawler && cd serverless-crawler && zappa init

```


### serverless-crawler/requirements.txt
```txt
beautifulsoup4==4.6.3
pynamodb==3.3.1
zappa==0.46.2
```

python은 requirements.txt 에 개발에 필요한 라이브러리를 기술합니다.

사용하는 라이브러리는 다음과 같습니다.

- beautifulsoup4: python web scrape 라이브러리
- pynamodb: DynamoDB를 사용하기 쉽도록 Modeling하는 도구
- zappa: python serverless framework

```sh
(venv) ec2-user:~/environment/serverless-crawler $ pip install -r requirements.txt

```


### serverless-crawler/zappa_settings.json

`zappa_settings.json` 을 아래처럼 변경하여 저장합니다.

```json
{
    "dev": {
        "apigateway_enabled": false,
        //"assume_policy": "serverless-crawler-policy.json",    // cloud9 환경이 아닌 경우 주석을 해제하여 serverless-crawler-policy 사용
        "aws_region": "ap-northeast-2",

        //"events": [
        //    {
        //        "function": "crawler.lambda_handler",
        //        "expression": "rate(10 minutes)"
        //    }
        //],
        "keep_warm": false,

        "lambda_description": "Python Serverless Crawler",
        "lambda_handler": "crawler.lambda_handler",
        "memory_size": 128,
        "manage_roles": false,
        "profile_name": "default",
        "project_name": "python-serverless-crawler",
        "role_name": "PythonServerlessCrawlerRole", // 원래 zappa에 의해 자동 생성 되지만, 수동 추가
        "runtime": "python3.6",
        "s3_bucket": "your-username-serverless-demo" // your-username 부분 변경,
    }
}
```

### serverless-crawler/crawler.py

```python
import datetime
import requests
from bs4 import BeautifulSoup
from pynamodb.models import Model
from pynamodb.attributes import UnicodeAttribute, ListAttribute


class PortalKeyword(Model):
    """
    A DynamoDB Keyword
    """
    class Meta:
        table_name = "PortalKeyword"
        region = 'ap-northeast-2'

    portal = UnicodeAttribute(hash_key=True)
    createdAt = UnicodeAttribute(range_key=True)
    keywords = ListAttribute()


def naver_keywords_crawler():
    created_at = datetime.datetime.utcnow().isoformat()[:19]
    naver_keywords = []

    try:
        naver_resp = requests.get('https://www.naver.com/')
        naver_soup = BeautifulSoup(naver_resp.text, 'html.parser')

        for i, tag in enumerate(naver_soup.find_all('span', {'class':'ah_k'})[:20]):
            rank = i+1
            keyword = tag.get_text()

            naver_keywords.append({'rank': rank, 'keyword': keyword})

        keyword_item = PortalKeyword('naver', created_at)
        keyword_item.keywords = naver_keywords
        keyword_item.save()

    except Exception as e:
        print(e)
        return None

    return naver_keywords


def daum_keywords_crawler():
    created_at = datetime.datetime.utcnow().isoformat()[:19]
    daum_keywords = []

    try:
        # daum 의 실시간 검색어 크롤러를 작성해 보세요.
        return True

    except Exception as e:
        print(e)
        return None

    return daum_keywords


def lambda_handler(event, context):

    naver_result = naver_keywords_crawler()
    daum_result = daum_keywords_crawler()

    # print(naver_result)
    # print(daum_result)

    if naver_result and daum_result:
        return 'success'
    else:
        return 'error'

```


### serverless-crawler/lambda_test.py
```python
from crawler import lambda_handler

lambda_handler(None, None)
```

터미널에서 테스트 코드를 실행시켜 봅니다.

```sh
(venv) ec2-user:~/environment/serverless-crawler $ python lambda_test.py
success
```

[DynamoDB Console](https://ap-northeast-2.console.aws.amazon.com/dynamodb/home?region=ap-northeast-2#tables:selected=PortalKeyword)에 들어가서 성공적으로 항목들이 생성되었는지 확인합니다.


## Cloud9에서 배포하기
### Zappa deploy
```sh
(venv) ec2-user:~/environment/serverless-crawler $ zappa deploy dev
Calling deploy for stage dev..
Downloading and installing dependencies..
 - sqlite==python36: Using precompiled lambda package
Packaging project as zip.
Uploading python-serverless-crawler-dev-1538136485.zip (5.8MiB)..
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 6.13M/6.13M [00:01<00:00, 3.48MB/s]
Deployment complete!
(venv) ec2-user:~/environment/serverless-crawler $
```

정상적으로 deploy 되었으면, [Lambda Console](https://ap-northeast-2.console.aws.amazon.com/lambda/home?region=ap-northeast-2#/functions) 에서 새로 생성된 `python-serverless-crawler-dev` 함수를 확인 할 수 있습니다.

### Zappa invoke
invoke 를 해봅니다.

```sh
(venv) ec2-user:~/environment/serverless-crawler $ zappa invoke dev
Calling invoke for stage dev..
[START] RequestId: XXXXXXX-.... Version: $LATEST
[END] RequestId: XXXXXXX-....
[REPORT] RequestId: XXXXXXX-....
Duration: 2274.70 ms
Billed Duration: 2300 ms
Memory Size: 128 MB
Max Memory Used: 47 MB
```

### Zappa update

주기적으로 크롤링 하도록 함수를 update 를 해봅니다.

`zappa_settings.json` 에서 event 주석 된 부분을 주석 해제합니다.

```sh
(venv) ec2-user:~/environment/serverless-crawler $ zappa update dev
Calling update for stage dev..
Downloading and installing dependencies..
 - sqlite==python36: Using precompiled lambda package
Packaging project as zip.
Uploading python-serverless-crawler-dev-1538137619.zip (5.8MiB)..
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 6.13M/6.13M [00:01<00:00, 3.51MB/s]
Updating Lambda function code..
Updating Lambda function configuration..
Scheduling..
Scheduled python-serverless-crawler-dev-crawler.lambda_handler with expression rate(10 minutes)!
Your updated Zappa deployment is live!
```

성공적으로 update 되었습니다.
지금부터 10분마다 주기적으로 DynamoDB에 검색어 랭킹이 쌓입니다.

## 리소스 삭제하기

서버리스 앱은 내리는 것이 어렵지 않습니다.
간단한 Command 하나면 모든 스택이 내려갑니다.
Cloud9에서 새로운 터미널을 열고 다음과 같이 입력합니다.

```sh
(venv) ec2-user:~/environment/serverless-crawler $ zappa undeploy dev
Calling undeploy for stage dev..
Are you sure you want to undeploy? [y/n] y
Unscheduling..
Unscheduled python-serverless-crawler-dev-crawler.lambda_handler.
Deleting Lambda function..
Done!
```


[DynamoDB Console](https://ap-northeast-2.console.aws.amazon.com/dynamodb/home?region=ap-northeast-2#)로 들어가서 Table을 삭제합니다. 리전은 서울입니다.

[Cloud9 Console](https://ap-southeast-1.console.aws.amazon.com/cloud9/home?region=ap-southeast-1)로 들어가서 IDE를 삭제합니다. 리전은 싱가포르입니다.

[S3 Console](https://s3.console.aws.amazon.com/s3/home?region=ap-northeast-2##)로 들어가서 생성된 버킷을 삭제합니다.

## References

- [https://aws.amazon.com/ko/cloud9/](https://aws.amazon.com/ko/cloud9/)
- [https://www.zappa.io/](https://www.zappa.io/)
- [https://www.crummy.com/software/BeautifulSoup/bs4/doc/](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)
- [https://pynamodb.readthedocs.io](https://pynamodb.readthedocs.io)