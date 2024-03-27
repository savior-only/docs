---
title: pypi 镜像_pypi 下载地址_pypi 安装教程 - 阿里巴巴开源镜像站
url: https://developer.aliyun.com/mirror/pypi?spm=a2c6h.13651102.0.0.3e221b11zVEiHf
clipped_at: 2024-03-28 00:19:21
category: default
tags: 
 - developer.aliyun.com
---


# pypi 镜像_pypi 下载地址_pypi 安装教程 - 阿里巴巴开源镜像站

# PyPI 镜像 动手试一下

免费试用

云服务器 ECS，每月免费额度 280 元 3 个月

推荐场景：

[部署并使用 Docker](https://developer.aliyun.com/adc/scenario/b41bf5d4668e4ec28c28c42ac0cdd886)[快速搭建云上博客](https://developer.aliyun.com/adc/scenario/fdecd528be6145dcbe747f0206e361f3)[搭建微信小程序](https://developer.aliyun.com/adc/scenario/aa0f837b017d44b093cdb6bd456bd0e7)

立即试用

云服务器 ECS，u1 2 核 4GB 1 个月

推荐场景：

[搭建 2048 小游戏](https://developer.aliyun.com/adc/scenario/fc3eb512337041779b1cd4d823ab7b1d?spm=a2c6h.14164896.0.0.4d8a47c56sSFDx&scm=20140722.S_community@@%E5%AE%9E%E9%AA%8C@@fc3eb512337041779._.ID_fc3eb512337041779b1cd4d823ab7b1d-RL_2048%E5%B0%8F%E6%B8%B8%E6%88%8F-LOC_search~UND~community~UND~item-OR_ser-V_3-P0_0)[搭建 turtle 画布](https://developer.aliyun.com/adc/scenario/exp/ebebe473948741008541a1fb79856b72)[搭建 wiki 知识库](https://developer.aliyun.com/adc/scenario/exp/28a41d19c3ce4f01b012d14103784870)

立即试用

## 简介

PyPI(Python Package Index) 是 Python 编程语言的软件存储库。开发者可以通过 PyPI 查找和安装由 Python 社区开发和共享的软件，也可以将自己开发的库上传至 PyPI。

下载地址：[https://mirrors.aliyun.com/pypi/](https://mirrors.aliyun.com/pypi/)

## 公网配置方法

a. 找到下列文件

```plain
~/.pip/pip.conf
```

b. 在上述文件中添加或修改：

```plain
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

## ECS 内网配置方法

a. 找到下列文件

```plain
~/.pip/pip.conf
```

b. 在上述文件中添加或修改：

```plain
[global]
index-url=http://mirrors.cloud.aliyuncs.com/pypi/simple/

[install]
trusted-host=mirrors.cloud.aliyuncs.com
```

## 相关链接

-   官方主页：[https://pypi.org/](https://pypi.org/)

## 软件包预警

### 背景

Python 库的官方仓库 pypi 允许开发者自由上传软件包，这会导致某些攻击者利用这点构造恶意包进行供应链攻击，在用户安装包或者引入包时触发恶意行为。目前国内镜像源与官方镜像源往往并不是完全一致，国内源普遍采用的是增量更新机制，也就是官方删除的恶意包国内不会同步删除。而这一部分恶意包可能会在很长一段时间内对国内用户造成影响。

阿里云安全 - 薪火实验室结合自建检测引擎 - 伏魔引擎长期对供应链恶意样本进行监控和预警，保护阿里 pypi 源安全。

### 删除包列表

以下具有安全威胁的软件包阿里 pypi 源已对应删除，开发者可通过 pip list 命令对于机器上已经安装的 pypi 包进行自查，如发现安装有以下具有安全风险的包，建议尽快删除修复。

| 恶意包 | 备注  |
| --- | --- |
| fake-useragant | 恶意后门 |
| maratlib | 恶意后门 |
| maratlib1 | 恶意后门 |
| matplatlib-plus | 恶意后门 |
| mllearnlib | 恶意后门 |
| mplatlib | 恶意后门 |
| learninglib | 恶意后门 |
| importantpackage | 反弹 shell |
| important-package | 反弹 shell |
| pptest | 反弹 shell |
| ipboards | 反弹 shell |
| owlmoon | 反弹 shell |
| discordsafety | 反弹 shell |
| trrfab | 反弹 shell |
| 10cent10 | 反弹 shell |
| 10cent11 | 反弹 shell |
| yandex-yt | 反弹 shell |
| yiffparty | 反弹 shell |
| giganigga | 反弹 shell |
| trrfab | 反弹 shell |
| rbperf | 恶意后门 |
| osxframeworks | 恶意后门 |
| caldavtester | 恶意后门 |
| kubectl-testing2 | 获取敏感信息 |
| nflx-kragle-scripts | 获取敏感信息 |
| dbattery-python-api | 获取敏感信息 |
| portal-api | 获取敏感信息 |
| csitools | 获取敏感信息 |
| eaa-commons | 获取敏感信息 |
| yow-utils | 获取敏感信息 |
| ipboards | 获取敏感信息 |
| eyesearch | 恶意后门 |
| backdoorxrat | 恶意后门 |
| amazon-freertos-ble-ios-sdk | 获取敏感信息 |
| aws-lambda-dotnet | 获取敏感信息 |
| uber-sdk | 获取敏感信息 |
| aws-xray-sdk-dotnet | 获取敏感信息 |
| python-bigquery-connection | 获取敏感信息 |
| constructs-go | 获取敏感信息 |
| google-auth-library-oauthlib | 获取敏感信息 |
| runtimeconfig | 获取敏感信息 |
| amazon-states-language-service | 获取敏感信息 |
| exeggutor | 获取敏感信息 |
| proto-plus-python | 获取敏感信息 |
| cta | 获取敏感信息 |
| aws-sdk-ruby | 获取敏感信息 |
| python-dataproc | 获取敏感信息 |
| amazon-chime-sdk-js | 获取敏感信息 |
| homebrew-aws | 获取敏感信息 |
| gax-python | 获取敏感信息 |
| google-auth-library-httplib2 | 获取敏感信息 |
| amazon-ecs-init | 获取敏感信息 |
| aws-xray | 获取敏感信息 |
| game-servers | 获取敏感信息 |
| amazon-ec2-utils | 获取敏感信息 |
| aws-encryption-sdk-python | 获取敏感信息 |
| aws-glue-databrew-jupyter-extension | 获取敏感信息 |
| aws-sigv4-auth-cassandra-gocql-driver-plugin | 获取敏感信息 |
| aws-ssm-data-protection-provider-for-aspnet | 获取敏感信息 |
| aws-greengrass-core-sdk-java | 获取敏感信息 |
| aws-lambda-runtime-interface-client | 获取敏感信息 |
| datalabeling | 获取敏感信息 |
| aws-lambda-java-libs | 获取敏感信息 |
| amazon-freertos-ble-ios-sdk | 获取敏感信息 |
| python-storage | 获取敏感信息 |
| aws-appsync-community | 获取敏感信息 |
| amazon-braket-ocean-plugin-python | 获取敏感信息 |
| python-service-directory | 获取敏感信息 |
| aws-greengrass-core-sdk-js | 获取敏感信息 |
| aws-step-functions-data-science-sdk | 获取敏感信息 |
| aws-dax-go | 获取敏感信息 |
| aws-encryption-sdk-c | 获取敏感信息 |
| python-bigquery-storage | 获取敏感信息 |
| python-trace | 获取敏感信息 |
| python-phishingprotection | 获取敏感信息 |
| amazon-neptune-sparql-java-sigv4 | 获取敏感信息 |
| python-policy-troubleshooter | 获取敏感信息 |
| aws-swf-build-tools | 获取敏感信息 |
| azure-api | 获取敏感信息 |
| jumpbox | 获取敏感信息 |
| bluh | 获取敏感信息 |
| aws-lambda-nodejs-runtime-interface-client | 获取敏感信息 |
| yelp-pack | 获取敏感信息 |
| aws-ec2-instance-connect-config | 获取敏感信息 |
| aws-sdk-js | 获取敏感信息 |
| aws-nitro-enclaves-cli | 获取敏感信息 |
| python-network-connectivity | 获取敏感信息 |
| python-bigquery-reservation | 获取敏感信息 |
| aws-iot-device-sdk | 获取敏感信息 |
| yelp-profiling | 获取敏感信息 |
| aws-cdk-rfcs | 获取敏感信息 |
| vscode-test | 获取敏感信息 |
| yelp-batch | 获取敏感信息 |
| aws-app-mesh-controller-for-k8s | 获取敏感信息 |
| cloud-core | 获取敏感信息 |
| python-analytics-data | 获取敏感信息 |
| yelppack | 获取敏感信息 |
| aws-sdk-php-zf2 | 获取敏感信息 |
| aws-xray-sdk-ruby | 获取敏感信息 |
| amazon-ecs-cni-plugins | 获取敏感信息 |
| aws-eb-python-dockerfiles | 获取敏感信息 |
| python-recaptcha-enterprise | 获取敏感信息 |
| replication-delay-client | 获取敏感信息 |
| pythonlibtest | 获取敏感信息 |
| eks-distro-prow-jobs | 获取敏感信息 |
| access-context-manager | 获取敏感信息 |
| policy-troubleshooter | 获取敏感信息 |
| amazon-elastic-inference-tools | 获取敏感信息 |
| trusted-advisor-tools | 获取敏感信息 |
| aws-ofi-nccl | 获取敏感信息 |
| aws-sdk-java-archetype | 获取敏感信息 |
| google-cloudevents | 获取敏感信息 |
| jsii-runtime-go | 获取敏感信息 |
| aws-sessionstore-dynamodb-ruby | 获取敏感信息 |
| yelp-ips | 获取敏感信息 |
| python-aiplatform | 获取敏感信息 |
| nflx-cloudsol-python-libs | 获取敏感信息 |
| amazon-ecs-agent | 获取敏感信息 |
| aws-php-sns-message-validator | 获取敏感信息 |
| python-runtimeconfig | 获取敏感信息 |
| aws-dotnet-extensions-configuration | 获取敏感信息 |
| yourapplication | 获取敏感信息 |
| aws-kinesisanalytics-runtime | 获取敏感信息 |
| python-access-approval | 获取敏感信息 |
| amazon-neptune-csv-to-rdf-converter | 获取敏感信息 |
| python-cloudbuild | 获取敏感信息 |
| aws-sdk-unity-net | 获取敏感信息 |
| aws-step-functions-data-science-sdk-python | 获取敏感信息 |
| amazon-ecs-shim-loggers-for-containerd | 获取敏感信息 |
| aws-lambda-ruby-runtime-interface-client | 获取敏感信息 |
| python-video-transcoder | 获取敏感信息 |
| amazon-ec2-hibinit-agent | 获取敏感信息 |
| aws-iot-device-sdk-js-v2 | 获取敏感信息 |
| aws-sigv4-auth-cassandra-python-driver-plugin | 获取敏感信息 |
| aws-toolkit-jetbrains | 获取敏感信息 |
| aws-aspnet-cognito-identity-provider | 获取敏感信息 |
| base64io-python | 获取敏感信息 |
| containers-roadmap | 获取敏感信息 |
| amazon-cognito-sync-manager-net | 获取敏感信息 |
| aws-sdk-java | 获取敏感信息 |
| python-documentai | 获取敏感信息 |
| hashing-lib | 获取敏感信息 |
| yelp-lib | 获取敏感信息 |
| airbnb-sdk | 获取敏感信息 |
| aws-neuron-driver | 获取敏感信息 |
| python-datacatalog | 获取敏感信息 |
| aws-sdk-php-v3-bridge | 获取敏感信息 |
| aws-xray-sdk-python | 获取敏感信息 |
| aws-tools-for-powershell | 获取敏感信息 |
| sagemaker-sdk | 获取敏感信息 |
| aws-nitro-enclaves-samples | 获取敏感信息 |
| python-webrisk | 获取敏感信息 |
| aws-sdk-ruby-release-tools | 获取敏感信息 |
| yelp-servlib | 获取敏感信息 |
| paypal-python-sdk | 获取敏感信息 |
| aws-iot-device-sdk-v2 | 获取敏感信息 |
| aws-cdk | 获取敏感信息 |
| videointelligence | 获取敏感信息 |
| aws-sdk-php | 获取敏感信息 |
| python-api-common-protos | 获取敏感信息 |
| iot-atlas | 获取敏感信息 |
| json2jsii | 获取敏感信息 |
| yelpcorp | 获取敏感信息 |
| python-language | 获取敏感信息 |
| billingbudgets | 获取敏感信息 |
| ebay-sdk | 获取敏感信息 |
| aws-lambda-python-runtime-interface-client | 获取敏感信息 |
| aws-lambda-runtime-interface-emulator | 获取敏感信息 |
| bigquery-sqlalchemy | 获取敏感信息 |
| aws-toolkit-eclipse | 获取敏感信息 |
| aws-sdk-rails | 获取敏感信息 |
| grpc-sdk | 获取敏感信息 |
| paypalsdk | 获取敏感信息 |
| eks-distro | 获取敏感信息 |
| amazon-ec2-metadata-mock | 获取敏感信息 |
| python-pubsublite | 获取敏感信息 |
| aws-sdk-unity-net | 获取敏感信息 |
| python-cloudbuild | 获取敏感信息 |
| containeranalysis | 获取敏感信息 |
| aws-xray-daemon | 获取敏感信息 |
| bigquery-storage | 获取敏感信息 |
| org-policy | 获取敏感信息 |
| aws-sdk-cpp | 获取敏感信息 |
| jsii-runtime-go | 获取敏感信息 |
| google-cloudevents | 获取敏感信息 |
| aws-xray-sdk-dotnet | 获取敏感信息 |
| aws-lambda-nodejs-runtime-interface-client | 获取敏感信息 |
| aws-ec2-instance-connect-config | 获取敏感信息 |
| base64io-python | 获取敏感信息 |
| aws-toolkit-jetbrains | 获取敏感信息 |
| aws-sigv4-auth-cassandra-python-driver-plugin | 获取敏感信息 |
| python-notebooks | 获取敏感信息 |
| python-channel | 获取敏感信息 |
| aws-sigv4-auth-cassandra-nodejs-driver-plugin | 获取敏感信息 |
| aws-iot-device-sdk-python-v2 | 获取敏感信息 |
| analytics-admin | 获取敏感信息 |
| python-logging | 获取敏感信息 |
| recommendations-ai | 获取敏感信息 |
| python-speech | 获取敏感信息 |
| websecurityscanner | 获取敏感信息 |
| aws-ops-wheel | 获取敏感信息 |
| access-approval | 获取敏感信息 |
| amazon-kinesis-streams-for-fluent-bit | 获取敏感信息 |
| aws-sdk-net-extensions-cognito | 获取敏感信息 |
| aws-cdi-sdk | 获取敏感信息 |
| aws-ofi-nccl | 获取敏感信息 |
| aws-nitro-enclaves-cli | 获取敏感信息 |
| aws-iot-device-sdk | 获取敏感信息 |
| amazon-ssm-agent | 获取敏感信息 |
| python-tasks | 获取敏感信息 |
| codelyzer | 获取敏感信息 |
| amazon-eks-diag | 获取敏感信息 |
| amazon-braket-pennylane-plugin-python | 获取敏感信息 |
| aws-iot-device | 获取敏感信息 |
| aws-iot-device-sdk-embedded-c | 获取敏感信息 |
| ec2-macos-system-monitor | 获取敏感信息 |
| python-compute | 获取敏感信息 |
| awsui-documentation | 获取敏感信息 |
| aws-dynamodb-encryption-java | 获取敏感信息 |
| eks-distro | 获取敏感信息 |
| aws-k8s-tester | 获取敏感信息 |
| amazon-ec2-metadata-mock | 获取敏感信息 |
| aws-toolkit-eclipse | 获取敏感信息 |
| bigquery-sqlalchemy | 获取敏感信息 |
| aws-sdk-rails | 获取敏感信息 |
| aws-lambda-runtime-interface-emulator | 获取敏感信息 |
| billingbudgets | 获取敏感信息 |
| aws-lambda-python-runtime-interface-client | 获取敏感信息 |
| json2jsii | 获取敏感信息 |
| iot-atlas | 获取敏感信息 |
| python-language | 获取敏感信息 |
| game-servers | 获取敏感信息 |
| amazon-ec2-utils | 获取敏感信息 |
| amazon-ecs-init | 获取敏感信息 |
| google-auth-library-httplib2 | 获取敏感信息 |
| aws-greengrass-core-sdk-java | 获取敏感信息 |
| aws-sigv4-auth-cassandra-gocql-driver-plugin | 获取敏感信息 |
| aws-glue-databrew-jupyter-extension | 获取敏感信息 |
| homebrew-aws | 获取敏感信息 |
| gax-python | 获取敏感信息 |
| python-dataproc | 获取敏感信息 |
| aws-xray-sdk-java | 获取敏感信息 |
| amazon-chime-sdk-js | 获取敏感信息 |
| amazon-states-language-service | 获取敏感信息 |
| aws-refcocog-adv | 获取敏感信息 |
| python-analytics-admin | 获取敏感信息 |
| aws-northstar | 获取敏感信息 |
| binary-authorization | 获取敏感信息 |
| aws-secretsmanager-caching-java | 获取敏感信息 |
| aws-cdk-rfcs | 获取敏感信息 |
| python-bigquery-reservation | 获取敏感信息 |
| aws-xray-sdk-ruby | 获取敏感信息 |
| aws-sdk-java-archetype | 获取敏感信息 |
| trusted-advisor-tools | 获取敏感信息 |
| amazon-elastic-inference-tools | 获取敏感信息 |
| aws-sdk-js | 获取敏感信息 |
| aws-toolkit-visual-studio | 获取敏感信息 |
| ec2-macos-init | 获取敏感信息 |
| amazon-sagemaker-examples | 获取敏感信息 |
| python-texttospeech | 获取敏感信息 |
| jobs-for-aws-iot-embedded-sdk | 获取敏感信息 |
| python-gke-hub | 获取敏感信息 |
| elastic-beanstalk-roadmap | 获取敏感信息 |
| amazon-sagemaker-operator-for-k8s | 获取敏感信息 |
| python-container | 获取敏感信息 |
| aws-proton-public-roadmap | 获取敏感信息 |
| python-videointelligence | 获取敏感信息 |
| ec2-hibernate-linux-agent | 获取敏感信息 |
| gax | 获取敏感信息 |
| aws-eks-best-practices | 获取敏感信息 |
| aws-iot-device-sdk-java | 获取敏感信息 |
| ec2-image-builder-roadmap | 获取敏感信息 |
| aws-lambda-go | 获取敏感信息 |
| aws-auto-scaling-custom-resource | 获取敏感信息 |
| network-connectivity | 获取敏感信息 |
| aws-swf-flow-library | 获取敏感信息 |
| python-media-translation | 获取敏感信息 |
| data-qna | 获取敏感信息 |
| aws-sdk-ruby-record | 获取敏感信息 |
| python-irm | 获取敏感信息 |
| python-workflows | 获取敏感信息 |
| aws-xray-java-agent | 获取敏感信息 |
| aws-for-fluent-bit | 获取敏感信息 |
| aws-iot-device-sdk-arduino-yun | 获取敏感信息 |
| google-cloud-python | 获取敏感信息 |
| aws-health-tools | 获取敏感信息 |
| aws-graviton-getting-started | 获取敏感信息 |
| assured-workloads | 获取敏感信息 |
| aws-app-mesh-controller-for-k8s | 获取敏感信息 |
| python-api-common-protos | 获取敏感信息 |
| amazon-ec2-hibinit-agent | 获取敏感信息 |
| aws-sdk-php | 获取敏感信息 |
| videointelligence | 获取敏感信息 |
| python-webrisk | 获取敏感信息 |
| aws-sdk-ruby-release-tools | 获取敏感信息 |
| aws-greengrass-core-sdk | 获取敏感信息 |
| sagemaker-sdk | 获取敏感信息 |
| aws-xray-sdk-python | 获取敏感信息 |
| python-datacatalog | 获取敏感信息 |
| aws-nitro-enclaves-samples | 获取敏感信息 |
| aws-tools-for-powershell | 获取敏感信息 |
| aws-sdk-js-v3 | 获取敏感信息 |
| python-error-reporting | 获取敏感信息 |
| python-firestore | 获取敏感信息 |
| python-binary-authorization | 获取敏感信息 |
| google-auth-library-python-httplib2 | 获取敏感信息 |
| aws-secretsmanager-caching-net | 获取敏感信息 |
| python-dns | 获取敏感信息 |
| aws-extensions-for-dotnet-cli | 获取敏感信息 |
| texttospeech | 获取敏感信息 |
| aws-emr-containers-best-practices | 获取敏感信息 |
| python-area120-tables | 获取敏感信息 |
| python-websecurityscanner | 获取敏感信息 |
| aws-parallelcluster-cookbook | 获取敏感信息 |
| python-audit-log | 获取敏感信息 |
| aws-sdk-php-laravel | 获取敏感信息 |
| aws-step-functions-data-science | 获取敏感信息 |
| aws-kinesisanalytics-flink-connectors | 获取敏感信息 |
| aws-toolkit-vscode | 获取敏感信息 |
| python-bigquery-datatransfer | 获取敏感信息 |
| aws-sdk-js-crypto-helpers | 获取敏感信息 |
| aws-xray-sdk-go | 获取敏感信息 |
| python-api-core | 获取敏感信息 |
| documentai | 获取敏感信息 |
| aws-dotnet-deploy | 获取敏感信息 |
| python-security-private-ca | 获取敏感信息 |
| eks-charts | 获取敏感信息 |
| python-crc32c | 获取敏感信息 |
| service-management | 获取敏感信息 |
| aws-greengrass-core-sdk-python | 获取敏感信息 |
| amazon-neptune-gremlin-java-sigv4 | 获取敏感信息 |
| monitoring-dashboards | 获取敏感信息 |
| device-shadow-for-aws-iot-embedded-sdk | 获取敏感信息 |
| aws-sigv4-auth-cassandra-driver-plugin | 获取敏感信息 |
| aws-iot-device-sdk-java-v2 | 获取敏感信息 |
| amazon-chime-sdk-ios | 获取敏感信息 |
| python-functions | 获取敏感信息 |
| phishingprotection | 获取敏感信息 |
| aiplatform | 获取敏感信息 |
| python-bigquery | 获取敏感信息 |
| deep-learning-containers | 获取敏感信息 |
| aws-dynamodb-encryption-python | 获取敏感信息 |
| crypto-tools | 获取敏感信息 |
| python-os-config | 获取敏感信息 |
| retail | 获取敏感信息 |
| python-dialogflow | 获取敏感信息 |
| irm | 获取敏感信息 |
| device-defender-for-aws-iot-embedded-sdk | 获取敏感信息 |
| python-spanner | 获取敏感信息 |
| aws-sdk-php-symfony | 获取敏感信息 |
| ec2-hibernate-windows-agent | 获取敏感信息 |
| amazon-eks-pod-identity-webhook | 获取敏感信息 |
| amazon-codeguru-profiler-python-agent | 获取敏感信息 |
| python-artifact-registry | 获取敏感信息 |
| aws-lambda-base-images | 获取敏感信息 |
| eks-distro-build-tooling | 获取敏感信息 |
| amazon-keyspaces-cql-to-cfn-converter | 获取敏感信息 |
| python-talent | 获取敏感信息 |
| dcv-color-primitives | 获取敏感信息 |
| python-spanner-django | 获取敏感信息 |
| ec2-spot-instances-integrations-roadmap | 获取敏感信息 |
| bigquery-datatransfer | 获取敏感信息 |
| amazon-freertos-ble-ios-sdk | 获取敏感信息 |
| python-aiplatform | 获取敏感信息 |
| aws-lambda-java-libs | 获取敏感信息 |
| datalabeling | 获取敏感信息 |
| aws-ssm-data-protection-provider-for-aspnet | 获取敏感信息 |
| aws-lambda-runtime-interface-client | 获取敏感信息 |
| aws-sdk-mobile-analytics-js | 获取敏感信息 |
| aws-encryption-sdk-python | 获取敏感信息 |
| aws-xray | 获取敏感信息 |
| aws-xray-sdk-node | 获取敏感信息 |
| dialogflow-cx | 获取敏感信息 |
| pubsublite | 获取敏感信息 |
| google-auth-library-python | 获取敏感信息 |
| aws-iot-device-sdk-js | 获取敏感信息 |
| python-recommendations-ai | 获取敏感信息 |
| google-resumable-media-python | 获取敏感信息 |
| aws-logging-dotnet | 获取敏感信息 |
| aws-sdk-php-silex | 获取敏感信息 |
| constructs-go | 获取敏感信息 |
| aws-sdk-ruby | 获取敏感信息 |
| aws-encryption-sdk-c | 获取敏感信息 |
| python-trace | 获取敏感信息 |
| aws-dax-go | 获取敏感信息 |
| python-bigquery-storage | 获取敏感信息 |
| aws-step-functions-data-science-sdk | 获取敏感信息 |
| aws-greengrass-core-sdk-js | 获取敏感信息 |
| google-auth-library-oauthlib | 获取敏感信息 |
| aws-sessionstore-dynamodb-ruby | 获取敏感信息 |
| aws-greengrass-core-sdk-c | 获取敏感信息 |
| policy-troubleshooter | 获取敏感信息 |
| http-desync-guardian | 获取敏感信息 |
| access-context-manager | 获取敏感信息 |
| eks-distro-prow-jobs | 获取敏感信息 |
| aws-codebuild-docker-images | 获取敏感信息 |
| python-oslogin | 获取敏感信息 |
| python-recaptcha-enterprise | 获取敏感信息 |
| aws-toolkit-azure-devops | 获取敏感信息 |
| python-network-connectivity | 获取敏感信息 |
| aws-eb-python-dockerfiles | 获取敏感信息 |
| aws-nitro-enclaves-sdk-bootstrap | 获取敏感信息 |
| amazon-braket-schemas-python | 获取敏感信息 |
| amazon-sagemaker-clarify | 获取敏感信息 |
| aws-nitro-enclaves-sdk-c | 获取敏感信息 |
| python-bigquery-sqlalchemy | 获取敏感信息 |
| amazon-ec2-net-utils | 获取敏感信息 |
| amazon-ec2-instance-selector | 获取敏感信息 |
| aws-sdk-java-v2 | 获取敏感信息 |
| python-secret-manager | 获取敏感信息 |
| aws-iot-device-sdk-js-v2 | 获取敏感信息 |
| python-video-transcoder | 获取敏感信息 |
| aws-lambda-ruby-runtime-interface-client | 获取敏感信息 |
| amazon-ecs-shim-loggers-for-containerd | 获取敏感信息 |
| python-access-approval | 获取敏感信息 |
| aws-step-functions-data-science-sdk-python | 获取敏感信息 |
| amazon-neptune-csv-to-rdf-converter | 获取敏感信息 |
| aws-dotnet-extensions-configuration | 获取敏感信息 |
| aws-kinesisanalytics-runtime | 获取敏感信息 |
| python-service-directory | 获取敏感信息 |
| python-phishingprotection | 获取敏感信息 |
| aws-swf-build-tools | 获取敏感信息 |
| aws-nitro-enclaves-acm | 获取敏感信息 |
| amazon-braket-ocean-plugin-python | 获取敏感信息 |
| python-storage | 获取敏感信息 |
| aws-ec2-instance-connect-cli | 获取敏感信息 |
| aws-iot-device-sdk-v2 | 获取敏感信息 |
| aws-sdk-go-v2 | 获取敏感信息 |
| aws-iot-device-sdk-cpp | 获取敏感信息 |
| amazon-chime-sdk-component-library-react | 获取敏感信息 |
| amazon-kinesis-firehose-for-fluent-bit | 获取敏感信息 |
| amazon-chime-sdk-android | 获取敏感信息 |
| aws-toolkit-common | 获取敏感信息 |
| gke-hub | 获取敏感信息 |
| aws-sdk-net | 获取敏感信息 |
| aws-sam-build-images | 获取敏感信息 |
| python-bigtable | 获取敏感信息 |
| recaptcha-enterprise | 获取敏感信息 |
| aws-eb-glassfish-dockerfiles | 获取敏感信息 |
| cloud-core | 获取敏感信息 |
| python-monitoring-dashboards | 获取敏感信息 |
| amazon-ssm-document-language-service | 获取敏感信息 |
| aws-dynamodb-encryption | 获取敏感信息 |
| copilot-cli | 获取敏感信息 |
| aws-elastic-beanstalk-cli | 获取敏感信息 |
| efs-utils | 获取敏感信息 |
| python-resource-manager | 获取敏感信息 |
| amazon-cloudwatch-logs-for-fluent-bit | 获取敏感信息 |
| runtimeconfig | 获取敏感信息 |
| homebrew-tap | 获取敏感信息 |
| python-bigquery-connection | 获取敏感信息 |
| proto-plus-python | 获取敏感信息 |
| cta | 获取敏感信息 |
| aws-neuron-runtime-proto | 获取敏感信息 |
| amazon-cloudwatch-agent | 获取敏感信息 |
| aws-app-mesh-roadmap | 获取敏感信息 |
| aws-secretsmanager-caching-python | 获取敏感信息 |
| amazon-redshift-driver | 获取敏感信息 |
| python-billingbudgets | 获取敏感信息 |
| python-api-gateway | 获取敏感信息 |
| python-game-servers | 获取敏感信息 |
| aws-elastic-beanstalk-cli-setup | 获取敏感信息 |
| amazon-braket-examples | 获取敏感信息 |
| aws-neuron-tensorflow | 获取敏感信息 |
| python-service-management | 获取敏感信息 |
| connect-rtc-js | 获取敏感信息 |
| amazon-redshift-jdbc-driver | 获取敏感信息 |
| python-datastore | 获取敏感信息 |
| amazon-cognito-dotnet | 获取敏感信息 |
| amazon-freertos | 获取敏感信息 |
| aws-sdk-php-v3-bridge | 获取敏感信息 |
| aws-neuron-driver | 获取敏感信息 |
| python-documentai | 获取敏感信息 |
| amazon-cognito-sync-manager-net | 获取敏感信息 |
| aws-sdk-java | 获取敏感信息 |
| amazon-ecs-agent | 获取敏感信息 |
| aws-php-sns-message-validator | 获取敏感信息 |
| containers-roadmap | 获取敏感信息 |
| python-runtimeconfig | 获取敏感信息 |
| aws-aspnet-cognito-identity-provider | 获取敏感信息 |
| aws-dotnet-session-provider | 获取敏感信息 |
| python-containeranalysis | 获取敏感信息 |
| aws-node-termination-handler | 获取敏感信息 |
| aws-iot-device-sdk-python | 获取敏感信息 |
| google-cloud-irm | 获取敏感信息 |
| python-managed-identities | 获取敏感信息 |
| google-auth-library | 获取敏感信息 |
| jsii-release | 获取敏感信息 |
| error-reporting | 获取敏感信息 |
| python-retail | 获取敏感信息 |
| aws-eb-dockerfiles | 获取敏感信息 |
| python-dlp | 获取敏感信息 |
| aws-sdk-js-dist-tools | 获取敏感信息 |
| oslogin | 获取敏感信息 |
| compute | 获取敏感信息 |
| amazon-freertos-ble-android-sdk | 获取敏感信息 |
| bigtable | 获取敏感信息 |
| python-test-utils | 获取敏感信息 |
| aws-codedeploy-agent | 获取敏感信息 |
| aws-sdk-go | 获取敏感信息 |
| amazon-ecs-logs-collector | 获取敏感信息 |
| python-pubsublite | 获取敏感信息 |
| google-cloud-python-happybase | 获取敏感信息 |
| aws-iot-device-sdk-cpp-v2 | 获取敏感信息 |
| amazon-neptune-sigv4-signer | 获取敏感信息 |
| aws-cdk-go | 获取敏感信息 |
| python-cloud-core | 获取敏感信息 |
| aws-nitro-enclaves-nsm-api | 获取敏感信息 |
| python-grafeas | 获取敏感信息 |
| bigquery-reservation | 获取敏感信息 |
| python-org-policy | 获取敏感信息 |
| aws-sam-cli-app-templates | 获取敏感信息 |
| bigquery-connection | 获取敏感信息 |
| python-ndb | 获取敏感信息 |
| amazon-braket-sdk-python | 获取敏感信息 |
| python-datalabeling | 获取敏感信息 |
| amazon-neptune-sparql-java-sigv4 | 获取敏感信息 |
| aws-mobile-analytics-manager-net | 获取敏感信息 |
| python-policy-troubleshooter | 获取敏感信息 |
| aws-lambda-dotnet | 获取敏感信息 |
| aws-appsync-community | 获取敏感信息 |
| area120-tables | 获取敏感信息 |
| python-iot | 获取敏感信息 |
| security-private-ca | 获取敏感信息 |
| python-dataproc-metastore | 获取敏感信息 |
| python-securitycenter | 获取敏感信息 |
| audit-log | 获取敏感信息 |
| google-auth-library-python-oauthlib | 获取敏感信息 |
| amazon-redshift-python-driver | 获取敏感信息 |
| amazon-vpc-cni-k8s | 获取敏感信息 |
| amazon-ecs-cli | 获取敏感信息 |
| python-access-context-manager | 获取敏感信息 |
| python-dialogflow-cx | 获取敏感信息 |
| python-data-qna | 获取敏感信息 |
| amazon-vpc-cni-plugins | 获取敏感信息 |
| dataproc-metastore | 获取敏感信息 |
| aws-xray-dotnet-agent | 获取敏感信息 |
| amazon-s3-encryption-client-dotnet | 获取敏感信息 |
| python-billing | 获取敏感信息 |
| aws-greengrass-core | 获取敏感信息 |
| analytics-data | 获取敏感信息 |
| gapic-generator-python | 获取敏感信息 |
| aws-deep-learning-containers-utils | 获取敏感信息 |
| aws-iot-device-v2 | 获取敏感信息 |
| aws-encryption-sdk-java | 获取敏感信息 |
| jsii-docgen | 获取敏感信息 |
| python-vision | 获取敏感信息 |
| python-asset | 获取敏感信息 |
| aws-secretsmanager-caching-go | 获取敏感信息 |
| python-recommender | 获取敏感信息 |
| python-automl | 获取敏感信息 |
| managed-identities | 获取敏感信息 |
| aws-js-sns-message-validator | 获取敏感信息 |
| amazon-vpc-resource-controller-k8s | 获取敏感信息 |
| aws-sigv4-auth-cassandra-java-driver-plugin | 获取敏感信息 |
| google-cloudevents-python | 获取敏感信息 |
| aws-sdk-php-zf2 | 获取敏感信息 |
| python-assured-workloads | 获取敏感信息 |
| python-kms | 获取敏感信息 |
| amazon-ecs-cni-plugins | 获取敏感信息 |
| python-monitoring | 获取敏感信息 |
| aws-dotnet-trace-listener | 获取敏感信息 |
| python-analytics-data | 获取敏感信息 |
| amazon-kinesis-video-streams-parser-library | 获取敏感信息 |
| aws-encryption-sdk-javascript | 获取敏感信息 |
| cloudbuild | 获取敏感信息 |
| video-transcoder | 获取敏感信息 |
| aws-cloudtrail-processing-library | 获取敏感信息 |
| webrisk | 获取敏感信息 |
| aws-app-mesh-examples | 获取敏感信息 |
| artifact-registry | 获取敏感信息 |
| media-translation | 获取敏感信息 |
| jsii-srcmak | 获取敏感信息 |
| aws-fpga | 获取敏感信息 |
| aws-neuron-sdk | 获取敏感信息 |
| amazon-codeguru-profiler-agent | 获取敏感信息 |
| api-common-protos | 获取敏感信息 |
| python-memcache | 获取敏感信息 |
| spanner-django | 获取敏感信息 |
| amazon-ecs-cluster-state-service | 获取敏感信息 |
| uberrides | 获取敏感信息 |
| yelp-aws | 获取敏感信息 |
| yelp-crypto | 获取敏感信息 |
| yelp-markupsafe | 获取敏感信息 |
| google-sdk | 获取敏感信息 |
| gglib | 恶意后门 |
| gethttpanand | 获取敏感信息 |
| guzzlehttp | 恶意后门 |
| yelp-xenial | 获取敏感信息 |
| googlesdk | 获取敏感信息 |
| tesla-sdk | 获取敏感信息 |
| yelpaws | 获取敏感信息 |
| groupon | 获取敏感信息 |
| google-cloud-sdk | 获取敏感信息 |
| aws-api | 获取敏感信息 |
| brawlslib | 恶意后门 |
| yelp-dataset | 获取敏感信息 |
| yelp-email | 获取敏感信息 |
| teslaapi | 获取敏感信息 |
| google-grpc | 获取敏感信息 |
| zoom-api | 获取敏感信息 |
| airbnb-api | 获取敏感信息 |
| zoom-sdk | 获取敏感信息 |
| static-completion | 获取敏感信息 |
| youtube-comment-auto-liker | 获取敏感信息 |
| osxframeworks | 恶意后门 |
| rbperf | 恶意后门 |
| caldavtester | 恶意后门 |
| guzzlehttp | 获取敏感信息 |

### 恶意类型解释

> 恶意后门：恶意程序可以在用户不知情的情况下直接安装启动运行，后台开启特殊接口，方便黑客再次进入受害者计算机，通常会添加非法的启动项，服务等用于长期驻留
> 
> 反弹 shell：黑客在获取执行权限后，可以通过在受害者机器上建立 shell 会话连接至黑客计算机，黑客可以通过 shell 会话直接下发命令控制受害者计算机。
> 
> 获取敏感信息：非法窃取受害者计算机相关敏感信息，比如系统、软件账号密码、配置，系统截屏，私密文件内容，钱包信息等，并将这些信息通过各种手段泄露给黑客服务器。
> 
> 远程命令执行：恶意程序启动会获取黑客服务器信息，并根据远程数据，执行可被黑客控制的远程代码，用于控制受害者机器，执行黑客下发的指令等。
> 
> 挖矿木马：木马在用户不知情的情况下，使用计算机算力对各类数字货币进行挖矿获利，占用机器大量 CPU、GPU 资源，严重影响其他应用执行。
> 
> 篡改剪贴板加密货币地址：恶意程序会监视机器剪贴板，一旦发现用户复制加密货币钱包地址，便将剪贴板中地址替换成黑客指定的钱包地址，用户便可能在不知情中向攻击者转账数字货币。

### 参考

-   [https://heimdalsecurity.com/blog/malicious-pypi-packages-used-to-mine-cryptocurrency/](https://heimdalsecurity.com/blog/malicious-pypi-packages-used-to-mine-cryptocurrency/)
    
-   [https://thehackernews.com/2021/11/11-malicious-pypi-python-libraries.html](https://thehackernews.com/2021/11/11-malicious-pypi-python-libraries.html)
    
-   [https://github.com/rsc-dev/pypi\_malware](https://github.com/rsc-dev/pypi_malware)
    
-   [https://www.helpnetsecurity.com/2019/07/18/malicious-python-packages/](https://www.helpnetsecurity.com/2019/07/18/malicious-python-packages/)
    
-   [https://snyk.io/blog/malicious-packages-found-to-be-typo-squatting-in-pypi/](https://snyk.io/blog/malicious-packages-found-to-be-typo-squatting-in-pypi/)
    
-   [https://www.nbu.gov.sk/skcsirt-sa-20170909-pypi/](https://www.nbu.gov.sk/skcsirt-sa-20170909-pypi/)
    
-   [https://www.itsfoss.net/malicious-packages-mitmproxy2-and-mitmproxy-iframe-removed-from-pypi-directory/](https://www.itsfoss.net/malicious-packages-mitmproxy2-and-mitmproxy-iframe-removed-from-pypi-directory/)
    
-   [https://www.freebuf.com/articles/network/284753.html](https://www.freebuf.com/articles/network/284753.html)
    
-   [https://security.bytedance.com/techs/wuheng-lab-share-spite-components](https://security.bytedance.com/techs/wuheng-lab-share-spite-components)
    
-   [https://bytedance.feishu.cn/sheets/shtcnMIXEYzkTkmruPS9NwMl3ie?sheet=11a166](https://bytedance.feishu.cn/sheets/shtcnMIXEYzkTkmruPS9NwMl3ie?sheet=11a166)
    
-   [https://paper.seebug.org/1562/](https://paper.seebug.org/1562/)
    
-   [https://security.tencent.com](https://security.tencent.com/)
    
-   [https://nakedsecurity.sophos.com/zh/2018/10/30/snakes-in-the-grass-malicious-code-slithers-into-python-pypi-repository/](https://nakedsecurity.sophos.com/zh/2018/10/30/snakes-in-the-grass-malicious-code-slithers-into-python-pypi-repository/)
    
-   [http://www.djbh.net/webdev/web/BuildImproveAction.do?p=getGzdt&id=8a8182566ed3d102016fad4d5a250042](http://www.djbh.net/webdev/web/BuildImproveAction.do?p=getGzdt&id=8a8182566ed3d102016fad4d5a250042)
    
-   [https://security.snyk.io/](https://security.snyk.io/)
    
-   [https://github.com/pypa/pypi-support/](https://github.com/pypa/pypi-support/)