# aws-lambda

#### 简介
这是一个用来demo如何使用AWS Lambda的项目。主要使用的语言为python。
想要了解什么是AWS Lambda可以点[这里](http://docs.amazonaws.cn/lambda/latest/dg/welcome.html)

#### 前置条件
## Docker
[安装方法](https://docs.docker.com/docker-for-mac/install/)

## aws-sam-cli
```
//假如没有pip3，请先安装python3

pip3 install aws-sam-cli
```

##virtualenv
用来配置一个python的依赖隔离环境
```
pip3 install virtualenv
```

#### 演示步骤

## 初始化项目
```
sam init --runtime python3.6
```

注意，这里使用 `--runtime python3.6` 来指定相应的开发语言，默认时Node.js。其他支持的语言参考。
执行完命令之后，会新建了一个hello_world文件夹，里面有基本的执行文件。

## 配置virtualenv
```
virtualenv hello_world //指定依赖安装位置
source hello_world/bin/activate //激活依赖环境
```

## 准备运行环境
```
//安装requirements里面所述依赖到build目录里面
pip install -r requirements.txt -t hello_world/build/ 

//将所有hello_world里面的python文件复制到build目录里面
cp hello_world/*.py hello_world/build/
```

## 启动api网关
```
sam local start-api
```
注意，启动网关前，需要保证docker已经启动。成功启动之后，应该看到命令后有如下提示，证明网关已经启动。并且监听端口3000。假如有http GET请求访问http://127.0.0.1:3000/hello，将会调用HelloWorldFunction。
```
2018-07-17 10:33:46 Mounting HelloWorldFunction at http://127.0.0.1:3000/hello [GET]
2018-07-17 10:33:46 You can now browse to the above endpoints to invoke your functions. You do not need to restart/reload SAM CLI while working on your functions changes will be reflected instantly/automatically. You only need to restart SAM CLI if you update your AWS SAM template
2018-07-17 10:33:46  * Running on http://127.0.0.1:3000/ (Press CTRL+C to quit)
```

## 访问url
```
curl http://localhost:3000/hello
```
返回结果
```
{"message": "hello world", "location": "61.148.62.130"}% //location的值可能根据你调用的网络环境有不同
```

#### 原理解释
## 文件夹结构
```
.
├── README.md                   <-- README文件，自由编写相关须知事项
├── hello_world                 <-- 所有的代码放置的位置
│   ├── app.py                  <-- 自定义的Lambda函数代码，文件名字不限
│   └── build                   <-- 最后用于执行的代码，包含依赖部分
├── requirements.txt            <-- Python所有依赖的声明，类似Maven的pom
├── template.yaml               <-- SAM的模版，下面详细介绍
└── tests                       <-- 单元测试（待研究）
    └── unit
        ├── __init__.py
        └── test_handler.py
```

## Template 内容
```
Transform: AWS::Serverless-2016-10-31
Description: >
    sam-app

    Sample SAM Template for sam-app

# More info about Globals: https://github.com/awslabs/serverless-application-model/blob/master/docs/globals.rst
Globals:
    Function:
        Timeout: 3


Resources:

    HelloWorldFunction: //函数的名字，这个在sam指令里面可以指定哪个函数被执行
        Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
        Properties:
            CodeUri: hello_world/build/ //代码存放的位置
            Handler: app.lambda_handler //指定python文件里面的那个方法作为执行方法
            Runtime: python3.6 //运行环境
            Environment: # More info about Env Vars: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#environment-object
                Variables:
                    PARAM1: VALUE //这里可以定义一下函数里面需要用到的环境变量
            Events:
                HelloWorld:
                    Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
                    Properties:
                        Path: /hello 
                        Method: get

//此部分以下还没搞清楚用途，似乎不是必须的，待研究
Outputs:

    HelloWorldApi:
      Description: "API Gateway endpoint URL for Prod stage for Hello World function"
      Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"

    HelloWorldFunction:
      Description: "Hello World Lambda Function ARN"
      Value: !GetAtt HelloWorldFunction.Arn

    HelloWorldFunctionIamRole:
      Description: "Implicit IAM Role created for Hello World function"
      Value: !GetAtt HelloWorldFunctionRole.Arn
```