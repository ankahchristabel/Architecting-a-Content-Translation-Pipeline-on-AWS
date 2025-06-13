# Architecting-a-Content-Translation-Pipeline-on-AWS
Experience with Continuous Integration and Continuous Delivery (CI/CD)

Building the Web Application
Step 1: Setting Up the Infrastructure
Navigate to the folder in which your files are using your preferred CLI (In my project I used the windows command prompt)
In the folder ,I have the following files:
 index.css
 index.html
 translate.py
  index.css
 index.html
 translate.py                                
  
 lambda.py
buildspec.yml
cicd.tf
dev.tfvars
infrastructure.tf
variables.tf
Step 2: Examining infrastructure.tf
This file creates:
•	Two S3 buckets for English and Spanish content
•	A CloudFront distribution using OAC [Origin Access Control]
•	A Lambda@Edge function for origin requests
•	IAM roles and policies for permissions
Snippet for defining S3 buckets:
resource "aws_s3_bucket" "english-bucket" {
  bucket = var.en_bucket_name
}

resource "aws_s3_bucket" "spanish-bucket" {
  bucket = var.es_bucket_name
}
I defined these variables in dev.tfvars file:
en_bucket_name = "my-english-assets-bucket-project5"
es_bucket_name = "my-spanish-assets-bucket-project5"
Snippet of the CloudFront Configuration:
resource "aws_cloudfront_distribution" "s3_distribution" {
  origin {
    domain_name = aws_s3_bucket.english-bucket.bucket_regional_domain_name
    origin_access_control_id = aws_cloudfront_origin_access_control.oac.id
    origin_id = local.s3_origin_id  }}
Step 3: Building the CI/CD Pipeline
Configuring cicd.tf
This file defines:
	A CodeStar connection to GitHub
	An S3 bucket for storing CI/CD artifacts
	A CodeBuild project using buildspec.yml
	A CodePipeline with Source and Build stages
Snippet of the CodePipeline Configuration:
resource "aws_codepipeline" "app_pipeline" {
  name     = "app-pipeline"
  role_arn = aws_iam_role.pipeline_role.arn
  artifact_store {
    location = aws_s3_bucket.codepipeline-bucket.bucket
    type     = "S3" }
  stage {
  name = "Source"
  action {
  category = "Source"
  owner    = "AWS"
  provider = "CodeStarSourceConnection"
 version  = "1"
 output_artifacts = ["source_output"] } }
 stage {
 name = "Build"
 action {
 category = "Build"
 owner    = "AWS"
      provider = "CodeBuild"
      version  = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["build_output"] }}}
Step 4: Running Terraform Deployment
After defining all infrastructure components, deploy them using Terraform:
1.	Terraform init – This command performs initialization steps to prepare the working directory for use with Terraform.

2.	terraform apply -var-file="dev.tfvars"
A successful execution of this command should output Apply Complete.
Step 5: Granting GitHub Access to CodePipeline
1.	Navigate to the CodePipeline console
2.	Under Settings, select Connections.
3.	Locate app-dev-codestar with Pending status.
4.	Select it and click Update pending connection.
5.	Select Install a new App, log in with your GitHub credentials, and authorize AWS.

Step 6: Uploading Files to GitHub
To deploy the application, push the frontend files to your GitHub repository either through the CLI or upload them manually on the GitHub console.
Using the CLI.
git clone https://github.com/ankahchristabel/Chapter6-Project5
cp app/index.html app/index.css app/translate.py Chapter6-Project5/
cd Chapter6-Project5
git add .
git commit -m "Initial version"
git push origin main

Step 7: Verifying the Deployment
•	Retrieve your CloudFront distribution URL from the AWS console.
•	Open it in a browser to check if the website is displayed correctly.
•	Test automatic language detection by switching your browser language settings.


