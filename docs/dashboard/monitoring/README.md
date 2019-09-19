<!--
title: Serverless Dashboard - Monitoring
menuText: Monitoring
menuOrder: 1
layout: Doc
-->

<!-- DOCS-SITE-LINK:START automatically generated  -->

### [Read this on the main serverless docs site](https://www.serverless.com/framework/docs/dashboard/insights/)

<!-- DOCS-SITE-LINK:END -->

# Monitoring

Serverless Monitoring help you monitor, develop and optimize your serverless application by providing key metrics and alerts.

## Installing

Monitoring is enabled by default when you deploy a Service using the Serverless Framework CLI.

### Configuration

Serverless Framework will automatically enable log collection by adding a CloudWatch Logs Subscription to send logs that match a particular pattern to our infrastructure for processing. This is used for generating metrics and alerts.

When deploying, Serverless Framework will also create an IAM role in your account that allows the Serverless Framework backend to call FilterLogEvents on the CloudWatch Log Groups that are created in the Service being deployed. This is used to display the CloudWatch logs error details views alongside the stack trace.

If you wish to disable log collection, set the following options:

**serverless.yml**

```yaml
custom:
  enterprise:
    collectLambdaLogs: false
```

## Advanced Configuration Options

### Uploading Source Map

The [New Error Alert](#new-error) and the [Error Metrics](#errors) can be used to view the stack trace for the occurence of an error. Tools like Webpack and Typescript generate the packaged code and therefore may obfuscate the stack trace. The Serverless Framework Enterprise Plugin and SDK support sourcemaps to properly generate the stack trace.

To use a sourcemap, ensure that your packaging directory includes the compiled source, original source, and the source maps.

For example, if your directory structure is:

```
$ ls -l dist/* src/*
-rw-r--r--  1 dschep  staff   576B Mar 21 17:21 dist/handler.js
-rw-r--r--  1 dschep  staff   911B Mar 21 17:21 dist/handler.js.map
-rw-r--r--  1 dschep  staff   451B Mar 22 12:13 src/handler.js
```

Then you should have a packaging directory that includes all the files above:

```yaml
package:
  include:
    - src/*.js
    - dist/*.js
    - dist/*.js.map
```

## Capturing non-fatal errors

Your lambda function may throw an exception, but your function handles it in order to respond to the requestor without throwing the error. One very common example is functions tied to HTTP endpoints. Those usually should still return JSON, even if there is an error since the API Gateway integration will fail rather than returning a meaningful error.

For this case, we provide a `captureError` function available on either the `context` or on the module imported from `'./serverless-sdk'`. This will cause the invocation to still display as an error in the serverless dashboard while allowing you to return an error to the user.

Here is an example of how to use it from the `context` object:

```javascript
module.exports.hello = async (event, context) => {
  try {
    // do some real stuff but it throws an error, oh no!
    throw new Error('aa');
  } catch (error) {
    context.captureError(error);
  }
  return {
    statusCode: 500,
    body: JSON.stringify({ name: 'bob' }),
  };
};
```

And to import it instead, import with `const { captureError } = require('./serverless-sdk')` then call `captureError` instead of `context.captureError`.

```javascript
const { captureError } = require('./serverless_sdk');

module.exports.hello = async event => {
  try {
    // do some real stuff but it throws an error, oh no!
    throw new Error('aa');
  } catch (error) {
    captureError(error);
  }
  return {
    statusCode: 500,
    body: JSON.stringify({ name: 'bob' }),
  };
};
```