<!--
.. title: Where do the logs from fake lambda go?
.. slug: moto-lambda-logs
.. date: 2024-10-30 00:00:00
.. tags: python,testing,devops
.. category: 
.. link: 
.. description: 
.. type: text
-->

I've written before about [moto](https://docs.getmoto.org/en/latest/). It is a library for mocking out AWS services under test, and it is basically magic.

Recently, I was working on a test for some code that invokes an AWS lambda function, and I was using moto to mock out the lambda service. The fact that moto can do this is quite impressive in itself, but during the course of this I found myself needing some visibility onto what was happening in the lambda while moto was running it in the mock lambda environment. This led me to the question "Where do the logs from fake lambda go?"

It turns out the answer to this question is: "Into fake CloudWatch".

On one level this answer makes complete logical sense. Simultaneously I found this a bit mind-blowing 🤯 Moto really is incredibly capable software.

So, armed with this knowledge, how do we use it?

I've pushed a repo with some complete working code demonstrating this to [https://github.com/chris48s/moto-lambda-logs-demo](https://github.com/chris48s/moto-lambda-logs-demo). Here's a cut down version leaving out some imports and helper functions for the sake of brevity.

First lets define a toy lambda function we can test. As well as returning a response, our handler also prints something to stdout.

```py
# handler.py
def lambda_handler(event, context):
    print("log message")

    return {"statusCode": 200, "body": "Hello from Lambda!"}
```

We can use moto to run this handler in a mock lambda environment and test the response like this:

```py
def test_lambda():
    with mock_aws():
        role = _get_mock_role()

        lambda_client = boto3.client(
            "lambda",
            region_name="eu-west-1"
        )

        fn = lambda_client.create_function(
            FunctionName="TestLambdaFunction",
            Runtime="python3.10",
            Role=role["Role"]["Arn"],
            Handler="handler.lambda_handler",
            Code={"ZipFile": _make_lambda_zip()},
        )

        response = lambda_client.invoke(
            FunctionName=fn["FunctionName"],
            Payload=json.dumps({}),
        )

        payload = json.loads(
            response["Payload"].read().decode()
        )
        assert (
            payload == {
                "statusCode": 200,
                "body": "Hello from Lambda!"
            }
        )
        ...
```

Having invoked the lambda, we can then also inspect the CloudWatch logs generated while running that function and make assertions about anything written to the log streams. In this case, I'm asserting the output of our `print()` statement made it into the logs.

```py
def test_lambda():
    with mock_aws():

        ...
        assert (
            payload == {
                "statusCode": 200,
                "body": "Hello from Lambda!"
            }
        )

        logs_client = boto3.client(
            "logs",
            region_name="eu-west-1"
        )
        log_streams = logs_client.describe_log_streams(
            logGroupName=f"/aws/lambda/{fn['FunctionName']}"
        ).get("logStreams")

        log_events = logs_client.get_log_events(
            logGroupName=f"/aws/lambda/{fn['FunctionName']}",
            logStreamName=log_streams[0]["logStreamName"],
        ).get("events")

        assert (
            len([
                e for e in log_events
                if e["message"] == "log message"
            ]) == 1
        )
```

Moto really does provide an exceptionally deep and comprehensive mock AWS environment.
