### Task Queues

#### Overview

Task Queues are one of the most common use cases for AWS SQS in data engineering. They serve as an intermediary layer that holds the tasks that are waiting to be executed by worker nodes. Using SQS as a task queue helps in distributing the work and ensures that your system can scale to meet demand. It also improves fault tolerance as tasks are not lost even if a worker node fails.

#### Why Use Task Queues?

- **Load Balancing**: Distributes tasks evenly among multiple workers.
- **Scalability**: Easily add more workers as demand grows.
- **Fault Tolerance**: Tasks stay in the queue even if workers fail, ensuring they are eventually processed.
- **Decoupling**: Separate the task initiation and task execution logic, making the system more maintainable.

#### How It Works

1. **Task Initiation**: An initiator service pushes tasks into the SQS queue.
2. **Task Execution**: Worker nodes pull from the queue, execute the tasks, and optionally push the results to another queue or data store.
3. **Monitoring and Alerts**: AWS CloudWatch can be used to monitor the queue size, ensuring that it doesn’t get too large, which might indicate a problem with worker nodes.

#### Configuration Steps

1. **Create an SQS Queue**: Configure the queue based on your specific requirements (e.g., standard queue for at-least-once delivery or FIFO queue for strict order).
2. **Set Queue Attributes**: Customize the visibility timeout, message retention, etc.
3. **Worker Configuration**: Set up workers to poll the SQS queue and process messages.

#### Listing SQS Queues

```python
"""
Written on 09/06/2023 by:
James Bower https://twitter.com/jamesbower

Python script that returns a list of all AWS SQS Queues.
"""

#!/bin/python3

import boto3
from botocore.exceptions import ClientError
import json
import logging

# Logging
logger = logging.getLogger()
logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s: %(levelname)s: %(message)s')

import warnings
warnings.filterwarnings("ignore")

# SageMaker Setup
os.environ['AWS_DEFAULT_REGION'] = 'us-east-1'
role = sagemaker.get_execution_role()

boto_session = boto3.session.Session()
region = boto_session.region_name

# Variables
AWS_REGION = 'us-east-1'
sqs_resource = boto3.resource("sqs", region_name=AWS_REGION)

def list_queues():
    """
    Creates an iterable of all Queue resources in the collection.
    """
    try:
        sqs_queues = []
        for queue in sqs_resource.queues.all():
            sqs_queues.append(queue)
    except ClientError:
        logger.exception('Could not list queues.')
        raise
    else:
        return sqs_queues

if __name__ == '__main__':
    lst_of_sqs_queues = list_queues()
    for queue in lst_of_sqs_queues:
        logger.info(f'Queue URL - {queue.url}')

```

#### Sending SQS Message

```python
import boto3

# Initialize SQS client
sqs = boto3.client('sqs')

# URL of your SQS queue
queue_url = 'https://sqs.us-west-2.amazonaws.com/123456789012/MyQueue'

# Sending a new task to the queue
response = sqs.send_message(
    QueueUrl=queue_url,
    MessageBody='New ETL Task',
)

# Output the message ID of the new task
print(f"Message ID: {response['MessageId']}")
```
