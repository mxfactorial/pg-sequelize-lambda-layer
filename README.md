avoids deploying
```sh
du -sh node_modules
15M	node_modules
```
with [pg](https://www.npmjs.com/package/pg) and [sequelize](https://www.npmjs.com/package/sequelize) as dependencies

### build your own
1. `yarn install`
1. zip layer:
    ```sh
    mkdir nodejs && \
    cp -r node_modules nodejs/node_modules && \
    zip -r layer.zip nodejs && \
    rm -rf nodejs
    ```
1. customize prefixed cli variable assignments, then [publish](https://docs.aws.amazon.com/cli/latest/reference/lambda/publish-layer-version.html):
    ```sh
    APP_NAME=lambda-layer RUNTIME=nodejs REGION=us-east-1; \
    aws lambda publish-layer-version \
          --layer-name=$APP_NAME-$RUNTIME-deps \
          --description="$APP_NAME dependencies" \
          --zip-file=fileb://$(PWD)/layer.zip \
          --region=$REGION \
          --query 'LayerVersionArn' \
          --output=text
    ```
1. assign aws account number, then add [permission](https://docs.aws.amazon.com/cli/latest/reference/lambda/add-layer-version-permission.html):
    ```sh
    APP_NAME=lambda-layer RUNTIME=nodejs REGION=us-east-1 AWS_ACCOUNT_NUMBER=123456789012; \
    aws lambda add-layer-version-permission \
        --layer-name=$APP_NAME-$RUNTIME-deps \
        --statement-id GetLayerPermission \
        --action lambda:GetLayerVersion  \
        --principal $AWS_ACCOUNT_NUMBER \
        --version-number 1 \
        --region=$REGION
    ```

finally, reference `LayerVersionArn` from publish output in cloudformation or console