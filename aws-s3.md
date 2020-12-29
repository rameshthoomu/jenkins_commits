# Deploy artifacts to multiple aws accounts using custom pipes

This post helps you to understand how to publish zip artifacts to to multiple aws accounts s3 buckets. In the following steps, I used `kramos/alpine-zip`, `atlassian/pipelines-awscli` and `yaml anchors and alias` to simplify the code. When you are dealing with multiple environments, the code looks redundant and not elegant to manage but the with the help of `yaml anchors` and `alias`, we can reduce a lot. Below I have break down the code into multiple sections and for full view, please checkout this gist file here. https://gist.github.com/rameshthoomu/1a5fc0a37426ec50278d793f81b9e8c1

Below step defination creates a dist directory and zip all the current path files into dist/zipfile.zip file with the help of `kramos/alpine-zip` docker image. I have defined this as `zip-artifact` for future refence.

```
    - step: &zip-artifact
        name: zip artifact
        image:
          name: kramos/alpine-zip 
        script: &zip |-
          mkdir dist
          zip -r dist/zipfile.zip .
```
Below step defination uploads above built artifact to aws s3 account with the help of `atlassian/pipelines-awscli` docker image. Vaules to the variables are provided securily using bibucket repository variables. Also, I have used `aws sts assume-role` to upload artifact to multiple aws accounts. The IAM role which I have created has s3 upload permissions.

```
    - step: &upload-artifact
        name: Upload to aws s3
        image:
          name: atlassian/pipelines-awscli
        script: &upload |-
          ...
          ...
          aws s3 cp dist/zipfile.zip "s3://${ENV}-s3-bucket-name/s3-bucket-${COMMIT_TAG}"
```
let's now create custom pipelines for each environment. `custom` allows user to choose the environment while triggering bitbucket pipelines. Refer the above step defination names using `alias *`.
```
pipelines:
  custom:
    dev:
      - step:
          <<: *zip-artifact
          script:
            - *zip
          artifacts:
          - dist/**
      - step:
          <<: *upload-artifact
          script:
            - *common-vars
            - ENV="dev"
            - AWS_ACCOUNT_ID="xxxxxxxxxxxx"
            - *upload
 ```

the above custom pipeline is for dev environment. If you would like to deploy artifact to different environment's s3 buckets, copy and paste the above dev code and create a new custom pipeline for `tst` and `prod` environments by changing the *ENV* and *AWS_ACCOUNT_ID* variables.
 
See the below gist to checkout the code.

https://gist.github.com/rameshthoomu/1a5fc0a37426ec50278d793f81b9e8c1
