<p><a target="_blank" href="https://eraser-qa.web.app/workspace/COnLWUbSrz9btVaNpBWb" id="edit-in-eraser-github-link"><img alt="Edit in Eraser" src="https://firebasestorage.googleapis.com/v0/b/second-petal-295822.appspot.com/o/images%2Fgithub%2FOpen%20in%20Eraser.svg?alt=media&amp;token=968381c8-a7e7-472a-8ed6-4a6626da5501"></a></p>

- [﻿Getting Started](#getting-started)  
    - [﻿Prerequisites](#prerequisites)  
    - [﻿AWS CLI](#aws-cli)  
    - [﻿Cloudformation](#cloudformation) 
- [﻿Useful commands](#useful-commands)  
- [﻿Architecture](#architecture)  
- [﻿Infrastructure](#infrastructure)  
    - [﻿End-to-end cdk deployment flow](#end-to-end-cdk-deployment-flow)  
    - [﻿Directory Structure](#directory-structure)  
    - [﻿Product Specifications](#product-specifications)  
    - [﻿RAG Implementation](#rag-implementation)  
    - [﻿RLHF: Reinforcement learning with feedback](#rlhf-reinforcement-learning-with-feedback)  
    - [﻿Change Monitoring (codename: HawkEye)](#change-monitoring-codename-hawkeye)  
    - [﻿EMU: Embeddings - Generation & Upsert Microservice](#emu-embeddings---generation--upsert-microservice) 
- [﻿Automated Translation pipeline](#automated-translation-pipeline)  
- [﻿Product Indexation in Vector Space](#product-indexation-in-vector-space)  
    - [﻿Usecases:](#usecases) 
## Getting Started
### Prerequisites
- Fisrt you need to install the cdk cli. You can do this by running `npm install -g aws-cdk`  
- You will also need to install the dependencies for this project. You can do this by running `npm install`  .
- You will require node version 18 or above for this project.
- You will also need to have docker running if you want to build and deploy directly.
if you have these basic requirements you can start configuring the aws cli now.
### AWS CLI
- You will need to have the aws cli installed and configured. You can find instructions on how to do this [﻿here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)  
- You will need to have a profile configured with the name `html-parsing-service`  . You can find instructions on how to do this [﻿here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html) 
First run use `aws configure`  
- `aws configure list`  → List of profiles
- `aws configure list-profiles`  
- to create a new profile `aws configure --profile <name>`  
- you will then be prompted to enter the access key, secret access key, region.
- collect these access credentials from @Awsaf
- to view the profile `aws configure list --profile <name>`  
- in our case this will be `html-parsing-service`  
- to get the caller identity: `aws sts get-caller-identity`  
- `~/.aws`  contains the **credentials** and **config** files which we can edit directly.
### Cloudformation
Now, to make any infrastructure changes, we will use the cdk cli. You can find instructions on how to install it [﻿here](https://docs.aws.amazon.com/cdk/latest/guide/cli.html) you can run `cdk bootstrap` to bootstrap the account. This will create a cloudformation stack in your account which will be used to deploy the cdk app.

Once the bootstrapping is complete, you can run `cdk deploy` to deploy the stack. This will create a cloudformation stack in your account which will contain all the resources required for the app to run. You can find more information about the stack in the `cdk.json` file.

---

You should explore the contents of this project. It demonstrates a CDK app with an instance of a stack (`RAGInferenceStack`)

The `cdk.json` file tells the CDK Toolkit how to execute your app.

## Useful commands
- `npm run build`  compile typescript to js
- `npm run watch`  watch for changes and compile
- `npm run test`  perform the jest unit tests
- `cdk deploy`  deploy this stack to your default AWS account/region
- `cdk diff`  compare deployed stack with current state
- `cdk synth`  emits the synthesized CloudFormation template
## Architecture
---

We have 3 main components for the infrastructure:

- API Gateway
- Lambda
- DynamoDB
The API Gateway is the entry point for the user, it receives the request and forwards it to the Lambda function.
The Lambda function is responsible for the inference and returns the result to the API Gateway. The API Gateway then returns the result to the user.

DynamoDB is used to store the results of the inference, and any other related metadata.

![inference](undefined "inference")



Right now we have two core APIs: (with the flexibility to add more)

1. **Translation API:** returns the translated attributes based on input json
2. **Category Generation API:** Returns best suited category for a given product
## Infrastructure
The infrastructure is built using the AWS CDK. The CDK is a software development framework for defining cloud infrastructure in code and provisioning it through AWS CloudFormation. It offers a high-level object-oriented abstraction to define AWS resources imperatively using the power of modern programming languages.

Using Infrastructure as Code (IaC) allows us to manage our infrastructure in a way that is similar to how we manage our application code. So, we can scale our infrastructure in a way that is repeatable, predictable, and consistent.

Stages of CDK Deployment:

- **Bootstrap:** The first step is to bootstrap the account. This will create a cloudformation stack in your account which will be used to deploy the cdk app.
- **Deploy:** Once the bootstrapping is complete, you can run `cdk deploy`  to deploy the stack. This will create a cloudformation stack in your account which will contain all the resources required for the app to run. You can find more information about the stack in the `cdk.json`  file.
- **Destroy:** To destroy the stack, you can run `cdk destroy`  . This will delete the cloudformation stack from your account.
### End-to-end cdk deployment flow
Based on our IAC definition, a cloudformation stack is created in the account. This stack contains the following resources that are spinup simultaneously:

- Lambda function
- API Gateway
- DynamoDB table
- IAM roles and policies
- Cloudwatch logs
The code for the lambda function itself is also stored in S3, which cloudformation creates and manages for us.
**Diagram describing the end-to-end cdk deployment flow**

![cdk-deployment](/docs/diagrams/cdk-deployment.png "cdk-deployment")

### Directory Structure
- `bin/`  : This directory contains the Infrastructure as Code (IAC) for the CDK (Cloud Development Kit) stack. The CDK stack is used to define your cloud resources in a familiar programming language.
- `src/lambda/`  : This directory contains the code for the AWS Lambda function. AWS Lambda is a service that lets you run your code without provisioning or managing servers.
- `src/utils/`  : This directory contains various utilities used throughout the project. This includes:
    - `openai.ts`  : This is a custom wrapper for the OpenAI API. It likely contains functions to interact with OpenAI's services, such as generating text using GPT-3.
    - `pinecone.ts`  : This file contains code related to Pinecone integration. Pinecone is a vector database service, so this file likely contains functions for interacting with a Pinecone database.
    - `tracing.ts`  : This file contains code related to tracing. Tracing is a way to track the execution of your program and understand its performance and behavior.
### Product Specifications
The main goal of this project is to build a collection of APIs that can help automate operations team sourcing process with the help of GPT.

The first part that we tackle here is the product validation part. Every time a user selects an item for sourcing, they need to go through a manual process of validation to ensure the scraped attributes are accurate. We want to automate this process by using GPT to generate the validation suggestions. We also want to extract attributes like weight, dimensions, etc. and a simplified and better composed product title & description.

**Diagram below shows the End-to-End flow for the Product Specifications Implementation**

![product-specifications](/docs/diagrams/product-specifications.png "product-specifications")

Some new items from a product perspective

- topic changes
- identify key products to select
### RAG Implementation
The Lambda function itself does two main things:

- Uses the API request parameters to perform similarity search
- **Prompt Building:** Uses the results of the similarity search to build a prompt for the GPT-3.5/4 model
- Interact with large language model to generate response
- **Validation:** Validate the response from the language model and return the result to the user
---

**Stages of RAG**

1. **Retrieval:** The first stage of RAG is to retrieve the relevant documents from the database. This is done by using the similarity search API of pinecone. The API takes in a query and returns the top k results from the database. The query is generated by using the product attributes and the user input. The user input is used to generate a prompt for the language model. The prompt is generated by using the top k results from the similarity search API. The prompt is then used to generate the response from the language model.
2. **Category Generation:** The second stage of RAG is to generate the answer from the language model. This is done by using the prompt generated in the first stage.
**System Prompt:**
>   You are a merchandising specialist that organizes product listings for an e-commerce website. Based on the product title provided, please come up with a valid and simplified product title, translated in english. The english translated title should be simple and reflect what the actual product is, and suitable to put as an e-commerce product tile in a website. Now, based on this translated title, choose a category which also follows the google product taxonomy. Use the product attributes provided to estimate the width and dimensions of the product. Finally,Please return a valid Json with the following keys: 'category', 'product_title_translated','weight_per_unit_kg','length_cm','width_cm','height_cm'  

Now, for the to-k results of the retrieval, we loop using the metadata:

**User Prompt**
`Product Title: ${metadata?.scraped_title};Variants: ${metadata?.scraped_product_variants};Attributes: ${metadata?.scraped_attributes}`,

**Assistant**

```js
JSON.stringify({
category: metadata?.gpt_product_category_final,
product_title_translated: metadata?.gpt_title,
}),
```
1. **Taxonomy Validation:**
This is an extra step that we have added to the RAG pipeline. This step is used to validate the category generated by the language model and map them to actual DB recors. This is done by using the google product taxonomy. We use the category generated by the language model and compare it with the google product taxonomy using a similarity search with the taxonomy embeddings index. We then return the top k results from the similarity search API.
**Diagram below shows the End-to-End flow for the RAG Implementation**
![rag-categorization](/docs/diagrams/rag-categorization.png "rag")

### RLHF: Reinforcement learning with feedback
We want to use the feedback from the user to improve the model. This is done by monitoring whether or not the user has accepted the GPT suggestion in S-Cube, or rejected. If the user has accepted the suggestion, we can use this as a positive example to improve the model. If the user has rejected the suggestion, we can use this as a negative example to improve the model.

### Change Monitoring (codename: HawkEye)
Listen to Postgres for changed records, which occur whenever an end-user modifies and saves a record from the table in the frontend UI, and place the changed records on the SQS queue so that the Emu microservice can pick them up, embed, and upsert them to keep the Pinecone index in sync with the changes the user is making.

To handle its data bootstrapping responsibilities, HawkEye uses row-level locking to select distinct "batches" of records so that individual workers don't step on one another's work.

HawkEye workers will continue looping through batches of Postgres records and placing each row on the SQS queue as a JSON message until all records have been processed.

Once all records are processed, HawkEye exits its data bootstrapping phrase and will LISTEN to the Postgres database using triggers set up via `pg_notify.` 

Every time a user modifies a record in the frontend UI - that change will result in a table change notification that HawkEye sees. HawkEye will place the JSON containing the old and new records on the SQS queue, which Emu will pick up for processing downstream.

---

For our phase-1 implementation tho, we are opting for a simpler approach. We will be using a simple SQS queue in a producer-consumer pattern. The producer will be the `category-update` api and the consumer will be the **Emu** microservice. The `category-update` api will place the changed records on the SQS queue and the Emu microservice will pick them up, embed, and upsert them to keep the Pinecone index in sync with the changes the user is making.

![change_monitoring](undefined "change_monitoring")



### EMU: Embeddings - Generation & Upsert Microservice
The Emu microservice is responsible for embedding and upserting the records to the Pinecone index. It is a consumer of the SQS queue. It picks up the messages from the queue and embeds the records using the `pinecone` service. It then upserts the records to the Pinecone index.

**Diagrame below shows the Architecture for EMU**

![emu](/docs/diagrams/emu.png "emu")

## Automated Translation pipeline
The automated translation pipeline is responsible for translating the product attributes from the scraped language to english. It is a consumer of the SQS queue. It picks up the messages from the queue and translates the records using the `openai` service. It then upserts the records to the RDS database.

We also incorporate RAG in this pipeline to generate the product title and category for the translated product attributes with better accuracy

**Diagram below shows the Architecture for Automated Translation pipeline**

![translation](/docs/diagrams/translation.png "translation")

## Product Indexation in Vector Space
---

### Usecases:
- semantic search over product records
- Classification
- usage in RAG for translation and content generation
More details [﻿here](./docs/vectorDB.md) 




<!--- Eraser file: https://eraser-qa.web.app/workspace/COnLWUbSrz9btVaNpBWb --->