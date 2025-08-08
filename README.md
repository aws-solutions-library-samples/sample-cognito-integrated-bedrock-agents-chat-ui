# Guidance for Secure Chat UI for Amazon Bedrock Agents on AWS

## Table of Contents
- 🏛️ [Architecture Overview](#architecture-overview)
- 📋 [Solution Overview](#overview) 
- 💰 [Cost](#cost) 
- ✅ [Prerequisites](#prerequisites)
- 🚀 [Deployment Steps](#deployment-steps)
- 🔍 [Deployment Validation](#deployment-validation)
- 📘 [Running the Guidance](#running-the-guidance)
- 🔒 [Security Considerations](#security-considerations)
- 🚀 [Performance Optimization](#performance-optimization)
- ➡️ [Next Steps](#next-steps)
- 🧹 [Cleanup](#cleanup)
- ❓ [FAQ, Known Issues, Additional Considerations, and Limitations](#faq-known-issues-additional-considerations-and-limitations)
- 📝 [Revisions](#revisions)
- ⚠️ [Notices](#notices)
- 👥 [Authors](#authors)

## Architecture Overview

![Architecture diagram showing the flow between user, Amplify hosting, Cognito authentication, and Bedrock Agents](https://github.com/user-attachments/assets/daa34609-19af-49a9-b5d6-0d70c7e534a3)
*Figure 1: Architecture diagram showing the secure authentication flow and connection to Amazon Bedrock Agents*

The architecture implements a secure pattern for browser-based applications to interact with Amazon Bedrock Agents:

1. **Frontend Hosting**: AWS Amplify hosts the React application, providing scalable content delivery
2. **Authentication**: Amazon Cognito manages user authentication and provides temporary AWS credentials
3. **Secure API Access**: Temporary credentials allow the frontend to make authenticated calls to Amazon Bedrock
4. **AI Interaction**: Amazon Bedrock Agents process user queries and return responses directly to the frontend

This architecture eliminates the need for a custom backend while maintaining enterprise-grade security.


## Solution Overview

This guidance demonstrates how to build a secure, browser-based chat application that connects directly to [Amazon Bedrock Agents](https://aws.amazon.com/bedrock/agents/). The solution addresses the need for organizations to quickly deploy AI-powered chat interfaces with enterprise-grade security and authentication.

**What**: A React-based chat UI that securely connects to Amazon Bedrock Agents.

**Who**: Developers and organizations looking to implement secure AI chat interfaces.

**Why**: To provide a production-ready solution for deploying secure AI chat applications with minimal setup.

The solution leverages [AWS Amplify](https://aws.amazon.com/amplify/) for hosting and deployment, while implementing secure access through [Amazon Cognito](https://aws.amazon.com/cognito/)'s User and Identity Pools for temporary credential management and API authentication.


### Demo

https://github.com/user-attachments/assets/83c4f1c9-9495-4d9c-a323-2e8bbf484523

### High-Level Steps:

1. The user navigates to the Secure Chat UI URL, which is hosted on AWS Amplify
2. The page is returned with HTML, CSS, JavaScript. User is now able to input the configuration details for Amazon Cognito and Amazon Bedrock Agents
3. Upon configuration completion, the user is prompted to authenticate using Amazon Cognito with a username and password configured for them in the user pool
4. After successful authentication, Cognito Identity Pool will negotiate temporary credentials from [AWS Security Token Service (STS)](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html)
5. Cognito Identity Pool passes temporary AWS credentials to the frontend
6. Once authenticated, the user now sees the Secure Chat UI chat prompt to interact with the Amazon Bedrock Agent that is configured

## Cost

You are responsible for the cost of the AWS services used while running this Guidance.  

As of May 2025, the cost for running this Guidance with the default settings in the US East (N. Virginia) Region is approximately **$8.65** per month for serving up to 300 daily active users and hourly summarization by 5 developers.

We recommend creating a [Budget](https://console.aws.amazon.com/billing/home#/budgets) through [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) to help manage costs. Prices are subject to change. For full details, refer to the pricing webpage for each AWS service used in this Guidance.

### Sample Cost Table

| AWS Service      | Dimensions                                                                 | Cost (USD)     |
|------------------|-----------------------------------------------------------------------------|----------------|
| Amazon Cognito   | 1,000 active users per month without advanced security feature              | $0.00/month    |
| AWS Amplify      | 5 developers committing code twice a day + 300 daily active users           | $8.00/month    |
| Amazon Bedrock   | 5 developers summarizing 2K to 1K output tokens hourly using Amazon Nova Lite | $0.65/month    |

## Prerequisites

**Development Tools**
- Node.js v18+
- Latest npm version

**AWS Account Requirements**
- Access to the following services:
   - AWS Amplify (for hosting and deployment)
   - Amazon Bedrock Agents (for AI functionality)
   - Amazon Cognito (for authentication)

- [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) permissions to:
   - Configure and manage Bedrock Agents
   - Set up Cognito user/identity pools
   - Deploy Amplify applications

Note: Ensure your AWS account has sufficient permissions before starting the deployment process.

## Deployment Steps

**Objective**: Set up and deploy the Secure Chat UI application with proper configuration.

### Clone repository

1. Clone repository to your local machine:

```bash
git clone https://github.com/aws-samples/sample-cognito-integrated-bedrock-agents-chat-ui.git
```

2. Change directory to the folder:
```bash
cd sample-cognito-integrated-bedrock-agents-chat-ui
```

3. Install dependencies:
```bash
npm install
```

**Success Criteria**: All dependencies are installed without errors.

### Local testing before deploying to AWS Amplify

1. Start the development server:

```bash
npm run dev
```

2. Open your browser and navigate to http://localhost:5173
3. Verify the application loads correctly

**Success Criteria**: The application is running locally and the UI is displayed correctly.

### Manual deployment to AWS Amplify

**Objective**: Deploy the application to AWS Amplify manually.

1. Build the application:

```bash
npm run build
```

2. Package the dist folder contents:

```bash
cd dist
zip -r ../deployment.zip ./*
```

3. Navigate to the **AWS Amplify Console** in your AWS account
4. Click on **Host web app** > **Deploy without Git provider**
5. Upload the deployment.zip file created in step 2
6. Follow the prompts to complete the deployment

**Success Criteria**: The application is successfully deployed and accessible via the Amplify URL.

### Automated deployment to AWS Amplify

**Objective**: Deploy the application to AWS Amplify using GitHub integration.

1. [Fork](https://github.com/aws-samples/sample-cognito-integrated-bedrock-agents-chat-ui/fork) this repository

2. Open your [AWS Management console and go to AWS Amplify](https://console.aws.amazon.com/amplify/apps)

3. Follow the steps in this [documentation](https://docs.aws.amazon.com/amplify/latest/userguide/setting-up-GitHub-access.html#setting-up-github-app) to create an app and connect to the GitHub repository you just forked

4. After connecting to your repository, you will be at the "App Setting" step in the AWS Amplify console. From there click on **Edit YML File** and paste the following:
```YML
version: 1
frontend:
  phases:
    build:
      commands:
        - npm install
        - npm run build
  artifacts:
    baseDirectory: dist
    files:
      - "**/*"
  cache:
    paths:
      - node_modules/**/*
```
5. Click **Next** and then on **Save and Deploy** if everything looks good in the review page

**Success Criteria**: The application is successfully deployed and accessible via the Amplify URL.

## Set up Amazon Bedrock Agent

**Objective**: Configure an Amazon Bedrock Agent to work with the chat UI.

1. Navigate to the **Amazon Bedrock console** in your AWS account
2. [Create](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-create.html) and configure your Bedrock Agent
3. Note down the Agent ID and other relevant configuration [details](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-view.html)
4. To test this solution with a sample agent, you can leverage the Virtual Meteorologist Agent [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template located [here](https://github.com/aws-samples/virtual-meteorologist-using-amazon-bedrock-agents/blob/main/cfn-virtual-meteorologist-using-amazon-bedrock-agents.yaml)

**Success Criteria**: Amazon Bedrock Agent is created and configured with the necessary permissions.

## Deployment Validation

**Objective**: Verify that all components are working correctly together.

1. Navigate to your deployed Amplify application URL
2. Verify that the login page appears and you can authenticate using Cognito credentials
3. After login, confirm that the chat interface loads correctly
4. Enter a test prompt and verify that the Bedrock Agent responds appropriately

**Success Criteria**:
- Amplify app is successfully deployed and accessible
- Authentication with Cognito works correctly
- The chat interface renders properly
- The Bedrock Agent responds to user inputs

If any validation step fails, refer to the Troubleshooting section below.

## Running the Guidance

**Objective**: Use the deployed chat application to interact with your Bedrock Agent.

1. Launch the app (locally or via Amplify)
2. Log in via Cognito
3. Enter a prompt into the chat input
4. Observe the response generated by the Bedrock Agent

### Expected Output

- You will see a conversational response in the chat UI rendered by the React app
- All requests go securely through Amplify using temporary credentials

### Debugging and Logging
- Errors from the backend will be printed out in the chat for your visualization and troubleshooting
- More detailed causes of an error will be also printed in the console log of the browser
- For development and debugging purposes there are specific events that are printed in the console log of the browser

## Security Considerations

This solution implements several security best practices:

1. **Authentication**: Uses Amazon Cognito for secure user authentication
2. **Temporary Credentials**: Leverages AWS Security Token Service to provide short-lived credentials
3. **No Stored Secrets**: No long-term credentials are stored in the frontend
4. **HTTPS**: All communication is encrypted in transit

**Additional Security Recommendations**:
- Enable Multi-Factor Authentication (MFA) in your Cognito User Pool
- Implement the principle of least privilege for IAM roles
- Consider enabling [AWS WAF](https://aws.amazon.com/waf/) for additional protection against common web exploits

## Performance Optimization

To optimize the performance of your deployment:

1. **Response Time**: Configure appropriate timeouts for Bedrock Agent interactions
2. **Caching**: Implement client-side caching for frequently accessed resources
3. **Monitoring**: Set up [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) metrics to track performance and identify bottlenecks

## Next Steps

This implementation leverages AWS Cloudscape Design System components to create a consistent and professional user interface aligned with AWS design patterns. For future enhancements, you can also take advantage of Amplify Gen2's chat capabilities, which would allow you to seamlessly integrate interactive chat features while maintaining the same AWS-native look and feel. The addition of Amplify Gen2 chat components would complement the existing Cloudscape foundation, particularly in areas requiring real-time user interaction and support.

## Cleanup

**Objective**: Remove all resources created by this guidance to avoid ongoing charges.

1. **Delete Amplify app**  
   - From AWS Amplify Console, delete the app

2. **Delete Cognito pools**  
   - Remove both User and Identity Pools

3. **Delete Bedrock Agent**  
   - Navigate to the Amazon Bedrock console and delete the created agent

4. **Optional:** Delete associated IAM roles and policies

**Success Criteria**: All resources are successfully removed and no longer incurring charges.

## FAQ, Known Issues, Additional Considerations, and Limitations

### Known Issues

- Some browsers may block third-party cookies, which may affect login behavior

### Troubleshooting

1. **Authentication Issues**:
   - Verify that your Cognito User Pool is correctly configured
   - Check browser console for CORS-related errors

2. **Agent Not Responding**:
   - Confirm that your Bedrock Agent has the necessary permissions
   - Verify that the Agent ID is correctly entered in the configuration

3. **Deployment Failures**:
   - Check Amplify build logs for any errors
   - Ensure all dependencies are correctly specified in package.json

### Additional Considerations

- Amazon Bedrock requests are charged per token
- Consider implementing rate limiting for production deployments
- For high-traffic applications, evaluate scaling options for Cognito

For issues or feature requests, please use the [GitHub Issues tab](https://github.com/aws-samples/browser-client-bedrock-agents/issues).

## Revisions

- **v1.0.0** – Initial release with Bedrock Agent integration and Amplify deployment support

## Notices

Customers are responsible for making their own independent assessment of the information in this Guidance.  
This Guidance:  
(a) is for informational purposes only,  
(b) represents AWS current product offerings and practices, which are subject to change without notice, and  
(c) does not create any commitments or assurances from AWS and its affiliates, suppliers, or licensors.  

AWS products or services are provided "as is" without warranties, representations, or conditions of any kind, whether express or implied.  
AWS responsibilities and liabilities to its customers are controlled by AWS agreements, and this Guidance is not part of, nor does it modify, any agreement between AWS and its customers.

## Authors
- Sergio Barraza
- Salman Ahmed
- Ravi Kumar
- Ankush Goyal
