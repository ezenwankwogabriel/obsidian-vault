AWS Amplify is a set of tools and services that helps developers build, deploy, and host full-stack web and mobile applications without having to configure every backend component manually.

For AWS SAA-C03, remember: Amplify = a managed development and deployment platform for web and mobile apps.
## Key capabilities

- Hosting: Deploy frontend applications with HTTPS and a content delivery network.
- CI/CD: Connect a Git repository so code changes can trigger builds and deployments.
- Authentication: Integrate user sign-up, sign-in, and identity management, commonly through Amazon Cognito.
- APIs and data: Connect applications to supported API and data services, such as AWS AppSync, API Gateway, and databases.
- Backend development: Configure supported backend resources through Amplify's development tooling.

The exact features available depend on which generation of Amplify tooling and backend configuration you use.

## Amplify vs. other AWS services

| Service                          | Main purpose                                                       | Exam clue                                                  |
| -------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------- |
| AWS Amplify                      | Build, connect, and deploy full-stack web/mobile apps              | Developer-friendly application platform                    |
| AWS Elastic Beanstalk            | Deploy and manage applications on AWS infrastructure               | Platform manages EC2, load balancing, scaling              |
| Amazon S3 static website hosting | Store and serve static website files                               | Simple static hosting                                      |
| Amazon CloudFront                | Cache and deliver content globally                                 | CDN and edge delivery                                      |
| AWS App Runner                   | Run web applications and APIs from source code or container images | Managed web service with minimal infrastructure management |
| AWS Lambda                       | Execute code in response to events or requests                     | Serverless compute                                         |

### Example

Suppose you're building a React application with user accounts and a backend API.

With Amplify, you can connect your Git repository, deploy the frontend, configure authentication, and connect the app to backend services. You don't have to manually build every part of the deployment pipeline.

Amplify does not mean your application has no underlying AWS resources. It simplifies how you develop and manage them.

## SAA-C03 exam traps

1. Frontend hosting and backend compute are different. Amplify hosting does not mean your backend business logic automatically runs inside Amplify; it may use Lambda, AppSync, API Gateway, or other services.
2. Amplify is not a general replacement for EC2. If a question asks for full control over servers and operating systems, EC2 may be more appropriate.
3. Amplify is more than S3 static hosting. It offers an integrated workflow for application hosting and supported backend capabilities.
4. Amplify and CloudFront are not direct equivalents. CloudFront is a content delivery network; Amplify provides a higher-level application development and deployment experience.
