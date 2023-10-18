## The Cloud Resume Challenge

### Introduction
As an engineer passionate about tackling technical challenges, I was on a lookout for a project to showcase my DevOps skills. I stumbled upon the [Cloud Resume Challenge](https://cloudresumechallenge.dev/docs/the-challenge/aws/). This challenge appeared to be the perfect addition to my portfolio, as it demonstrated practical knowledge in several areas:

- Proficiency with a leading cloud provider (AWS).
- Setting up a CI/CD pipeline.
- Managing DNS and TLS Certificates.
- Implementing Infrastructure as Code (IaC).
- Demonstrating coding skills.
- Mastery of source control.
- Achieving certification as an AWS Cloud Practitioner.

While the challenge author discourages sharing the specific solutions online, they encourage participants to share their approach in blog posts. In this blog post, I will walk you through my approach in tackling the Cloud Resume Challenge, step by step.

### HTML and CSS: Building My Resume from Scratch
When I initially considered the challenge, I contemplated using pre-designed templates for the website. However, I couldn't find one that suited my needs, so I decided to create the entire site from scratch by following [this tutorial](https://w3collective.com/html-resume-bootstrap/). Fortunately, [Bootstrap](https://getbootstrap.com/) made this process much more straightforward.


### DNS and TLS X.509 Certificate: Securing My Domain
I acquired a domain from a third-party registrar and integrated it with AWS Route53 for further DNS management. I also leveraged AWS Certificate Manager, which provides free TLS certificates and seamless integration with other AWS services. The certificates are configured for automatic renewal through DNS validation.

Steps:

- Purchased a domain from a third-party registrar.
- Created a public hosted zone for the domain in AWS Route53.
- Updated domain name servers with NS records provided by Route53.
- Imported any existing domain records to Route53 via a DNS zone file.
- Provisioned a certificate in AWS Certificate Manager. Note: ensured that it is in the **us-east-1** region,  as Cloudfront (used later) only works with certificates from that region.
- Set up DNS certificate validation and added CNAME records in Route53.


### Static Website Infrastructure: S3 and CloudFront
For hosting the static website, I opted for the standard AWS combination of Amazon S3 and Amazon CloudFront. Amazon S3 provides a low-cost, durable, and highly scalable storage solution, while CloudFront serves as a content delivery network that improves performance by caching data at Edge Locations globally.

Steps:

- Created an S3 bucket.
- Uploaded the static content (HTML and CSS) to the S3 bucket.
- Created a CloudFront distribution with the S3 bucket as the origin.
- Configured an alternate domain name in CloudFront to correspond to my domain.
- Added an ALIAS record in Route53 that pointed to the CloudFront distribution. Note: there is no additional charge for queries to ALIAS records.


### CI/CD Frontend: Automating Deployment
With my code hosted on GitHub, I used GitHub Actions to set up my CI/CD pipeline. To ensure secure deployment without the use of IAM credentials, I opted for using IAM roles. It is a best practice approach that aligns with security guidelines.

Steps:

- Configured an IAM role to enable GitHub Actions to interact with AWS services. Detailed instructions can be found [here](https://www.automat-it.com/post/using-github-actions-with-aws-iam-roles).
- Created a GitHub Actions workflow file that triggered on every push to the main branch.
- This workflow synchronized new static files with the S3 bucket and invalidated the CloudFront distribution cache.

### Database: Storing Visitor Counts with DynamoDB
To store the visitor count for my site, I needed a place to keep this data. Given that I didn't require complex SQL data, I opted for Amazon DynamoDB, a NoSQL key-value database. DynamoDB is a fully managed, serverless, and scalable solution, making it a cost-effective choice for my requirements.

Steps:

- Created a DynamoDB table (e.g., cloud-resume-stats) with a string partition/primary key (e.g., **stat** for various potential future statistics; for now, only counter is used). I used on-demand capacity.
- Initialized the table with an item representing the visitor count, e.g., **stat: visitor-count, quantity: 0**.

### Lambda: Handling Visitor Counter Logic
Lambda is a serverless solution that is used to increment our visitor counter. While there is a [debate](https://theburningmonk.com/2018/01/aws-lambda-should-you-have-few-monolithic-functions-or-many-single-purposed-functions/) between using a monolithic Lambda function or multiple single-purpose functions, I chose the monolithic approach for simplicity in this context.

Steps:

- Installed the [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) package to interact with DynamoDB from my Python code.
- Implemented Python code to increment the visitor count in DynamoDB and return the last known counter value as a result. I used DynamoDB's increment (INC) key to avoid issues with multiple simultaneous requests.
- Tested the Lambda function using the [moto](https://pypi.org/project/moto/) package to mock the DynamoDB table.
- Created the Lambda function and deployed the code to it.
- Set up a Lambda execution role with permissions to read and update the DynamoDB table, and also permissions to create log streams, log groups, and log events.

### API Gateway: Providing an Interface
I needed an API to allow my frontend application to interact with the Lambda function. I chose to implement Amazon API Gateway V2 (HTTP) for its cost-effectiveness and lower latency.

Steps:

- Created an Amazon API Gateway V2.
- Set up a Lambda proxy integration for POST API calls.

### JavaScript and CORS: Enabling Frontend/Backend Interaction
To call the backend API from the frontend site, I used JavaScript. I encountered a Cross-Origin-Resource-Sharing (CORS) error, but it was easily resolved by configuring CORS on the API Gateway. This allowed me to manage CORS effectively without additional complexity.

Steps:

- Configured CORS settings on the API Gateway.
- Added JavaScript calls to the API Gateway when the page loads.


### CI/CD Backend: Automating the Backend Pipeline
For the backend CI/CD pipeline, I again relied on GitHub Actions. The CI steps involved installing dependencies, ensuring code formatting with Black, running lint checks with pylint, and performing unit tests.
The CD steps, triggered on every push to the main branch, involved configuring an IAM role to connect GitHub Actions to AWS, zipping the Lambda function, and deploying the Lambda function using aws lambda update-function-code.

### Terraform: Infrastructure as Code (IaC)
Manual resource configuration can be error-prone and tedious, so I utilized Terraform to automate the setup of all the resources mentioned in the previous steps. This allowed me to manage the infrastructure efficiently.

I followed the guidelines provided [here](https://cloudresumechallenge.dev/docs/extensions/terraform-getting-started/) to implement Terraform for this configuration.

### CI/CD Terraform: Automating Infrastructure Changes
To streamline Terraform management and ensure a consistent environment, I explored [Terraform Cloud](https://developer.hashicorp.com/terraform/tutorials/cloud-get-started). This provided a more organized and controlled way to handle Terraform changes.

Steps to connect Terraform Cloud, AWS, and GitHub Actions:

- Created a Terraform Cloud account.
- Generated a Terraform Cloud API token and stored it in the GitHub Actions repository secrets.
- Set up a Terraform Cloud workspace to represent the project.
- Generated an IAM user with permissions to make the necessary changes to AWS resources.
- Stored the IAM user credentials in Terraform Cloud as variables, marking them as sensitive (write-only).

For CI, on every push of Terraform code to GitHub:

- Ran tflint to check for issues.
- Checked the formatting of Terraform code.
- Validated configuration files.
- For every pull request, ran terraform plan and outputted the plan to GitHub as a comment.

For CD, on every push to the main branch:

- Triggered a Terraform Cloud run using the Terraform CLI, automatically applying changes.

### Certification: Achieving AWS Cloud Practitioner
Completing the Cloud Resume Challenge not only allowed me to enhance my DevOps skills but also required obtaining the AWS Cloud Practitioner certification.
While this certification was relatively straightforward for me due to prior AWS experience, it motivated me to pursue more challenging certifications such as the AWS Certified Solutions Architect Associate and Professional in the future.

### Conclusion
The result of my efforts can be viewed [here](https://ademirovic.com). Participating in the Cloud Resume Challenge was a rewarding experience that encouraged me to dig deeper into services I had previously superficially understood. I also had the chance to explore new technologies, such as Terraform Cloud, and had fun in the process.

In conclusion, the Cloud Resume Challenge is a fantastic initiative for DevOps enthusiasts to showcase their skills, expand their knowledge, and add valuable certifications to their portfolios.
