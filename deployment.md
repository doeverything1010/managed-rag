# 인프라 설치하기

## 모델 사용 권한 설정하기

여기서는 Multi-Region LLM을 사용하기 위하여, 아래 링크에 접속하여, [Edit]를 선택한 후에 모든 모델을 사용할 수 있도록 설정합니다. 특히 Anthropic Claude와 "Titan Embeddings V2 - Text"은 LLM 및 Vector Embedding을 위해서 반드시 사용이 가능하여야 합니다.

- [Model access - Oregon](https://us-west-2.console.aws.amazon.com/bedrock/home?region=us-west-2#/modelaccess)
- [Model access - N.Virginia](https://us-east-1.console.aws.amazon.com/bedrock/home?region=us-east-1#/modelaccess)
- [Model access - Canada](https://ca-central-1.console.aws.amazon.com/bedrock/home?region=ca-central-1#/modelaccess)
- [Model access - London](https://eu-west-2.console.aws.amazon.com/bedrock/home?region=eu-west-2#/modelaccess)
- [Model access - Sao Paulo](https://sa-east-1.console.aws.amazon.com/bedrock/home?region=sa-east-1#/modelaccess)

<!-- 
- [Model access - Sydney](https://ap-southeast-2.console.aws.amazon.com/bedrock/home?region=ap-southeast-2#/modelaccess),
- [Model access - Paris](https://eu-west-3.console.aws.amazon.com/bedrock/home?region=eu-west-3#/modelaccess)
- [Model access - Mumbai](https://ap-south-1.console.aws.amazon.com/bedrock/home?region=ap-south-1#/modelaccess)  -->


![noname](https://github.com/kyopark2014/llm-chatbot-using-claude3/assets/52392004/ca7f361a-1993-498e-93b6-ef19c620cbb1)


## CDK를 이용한 인프라 설치하기


여기서는 [Cloud9](https://aws.amazon.com/ko/cloud9/)에서 [AWS CDK](https://aws.amazon.com/ko/cdk/)를 이용하여 인프라를 설치합니다.

1) [Cloud9 Console](https://us-west-2.console.aws.amazon.com/cloud9control/home?region=us-west-2#/create)에 접속하여 [Create environment]-[Name]에서 “chatbot”으로 이름을 입력하고, EC2 instance는 “m5.large”를 선택합니다. 나머지는 기본값을 유지하고, 하단으로 스크롤하여 [Create]를 선택합니다.

![noname](https://github.com/kyopark2014/chatbot-based-on-Falcon-FM/assets/52392004/7c20d80c-52fc-4d18-b673-bd85e2660850)

2) [Environment](https://us-west-2.console.aws.amazon.com/cloud9control/home?region=us-west-2#/)에서 “chatbot”를 [Open]한 후에 아래와 같이 터미널을 실행합니다.

![noname](https://github.com/kyopark2014/chatbot-based-on-Falcon-FM/assets/52392004/b7d0c3c0-3e94-4126-b28d-d269d2635239)

3) EBS 크기 변경

아래와 같이 스크립트를 다운로드 합니다. 

```text
curl https://raw.githubusercontent.com/kyopark2014/technical-summary/main/resize.sh -o resize.sh
```

이후 아래 명령어로 용량을 80G로 변경합니다.
```text
chmod a+rx resize.sh && ./resize.sh 80
```


4) 소스를 다운로드합니다.

```java
git clone https://github.com/kyopark2014/managed-rag
```

5) cdk 폴더로 이동하여 필요한 라이브러리를 설치합니다.

```java
cd managed-rag/cdk-managed-rag/ && npm install
```

6) CDK 사용을 위해 Bootstraping을 수행합니다.

아래 명령어로 Account ID를 확인합니다.

```java
aws sts get-caller-identity --query Account --output text
```

아래와 같이 bootstrap을 수행합니다. 여기서 "account-id"는 상기 명령어로 확인한 12자리의 Account ID입니다. bootstrap 1회만 수행하면 되므로, 기존에 cdk를 사용하고 있었다면 bootstrap은 건너뛰어도 됩니다.

```java
cdk bootstrap aws://[account-id]/us-west-2
```

7) 인프라를 설치합니다.

```java
cdk deploy --require-approval never --all
```

설치가 완료되면 아래와 같은 Output이 나옵니다. 

![noname](https://github.com/kyopark2014/llm-multimodal-and-rag/assets/52392004/c5da1590-50b8-49bf-a3dc-686dfdc00fc3)


8) API에 대한 Credential을 획득하고 입력합니다.

- 일반 검색을 위하여 [Tavily Search](https://app.tavily.com/sign-in)에 접속하여 가입 후 API Key를 발급합니다. 이것은 tvly-로 시작합니다.

Tavily의 경우 1000건/월을 허용하므로 여러 건의 credential을 사용하면 편리합니다. 따라서, 아래와 같이 array형태로 입력합니다. 

```java
["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","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"]
```
  
- 날씨 검색을 위하여 [openweathermap](https://home.openweathermap.org/api_keys)에 접속하여 API Key를 발급합니다.
- [langsmith.md](./langsmith.md)를 참조하여 [LangSmith](https://www.langchain.com/langsmith)에 가입후 API Key를 발급 받습니다.

[Secret manger](https://us-west-2.console.aws.amazon.com/secretsmanager/listsecrets?region=us-west-2)에 접속하여, [openweathermap-langgraph-agent](https://us-west-2.console.aws.amazon.com/secretsmanager/secret?name=openweathermap-langgraph-agent&region=us-west-2), [tavilyapikey-langgraph-agent](https://us-west-2.console.aws.amazon.com/secretsmanager/secret?name=tavilyapikey-langgraph-agent&region=us-west-2), [langsmithapikey-langgraph-agent](https://us-west-2.console.aws.amazon.com/secretsmanager/secret?name=langsmithapikey-langgraph-agent&region=us-west-2)에 접속하여, [Retrieve secret value]를 선택 후, api key를 입력합니다.

9) HTMl 파일을 S3에 복사합니다.

아래와 같이 Output의 HtmlUpdateCommend을 붙여넣기 합니다. 

![noname](https://github.com/kyopark2014/llm-multimodal-and-rag/assets/52392004/1e273934-07ba-4319-bbdb-82445e424568)

10) Output의 WebUrlformanagedragchatbot 복사하여 브라우저로 접속합니다.
