# CloudFormation Custom Resource

```yaml
...
Resources:
  PWGLambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - lambda.amazonaws.com
          Action:
          - sts:AssumeRole
      Policies:
        -
          PolicyName: allowLambdaLogging
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              -
                Effect: "Allow"
                Action:
                  - "logs:*"
                Resource: "*"

  PWGRandomStringLambdaFunction:
   Type: AWS::Lambda::Function
   Properties:
      Code:
        ZipFile: >
          const response = require("cfn-response");
          const randomString = (length, chars) => {
              var result = '';
              for (var i = length; i > 0; --i) result += chars[Math.floor(Math.random() * chars.length)];
              return result;
          };
          exports.handler = (event, context) =>{
            const str = randomString(event['ResourceProperties']['Length'], '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ');
            const responseData = {RandomString: str};
            response.send(event, context, response.SUCCESS, responseData);
          };
      Handler: index.handler
      Runtime: nodejs8.10
      Role: !GetAtt PWGLambdaExecutionRole.Arn
      MemorySize: 128
      Timeout: 20

  DBPassword:
   Type: AWS::CloudFormation::CustomResource
   Properties:
     Length: 16
     ServiceToken: !GetAtt PWGRandomStringLambdaFunction.Arn
     
Outputs:
  DBPasswd:
    Value: !GetAtt DBPassword.RandomString
    Export:
      Name: !Sub "${AWS::StackName}-DBPasswd"
...
```