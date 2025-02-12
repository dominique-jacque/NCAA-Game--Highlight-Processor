# NCAA Game Highlight Processor
## Overview
Game Highlight Processor is an automated solution designed to fetch, process, and convert NCAA game highlights. By integrating RapidAPI for highlight retrieval, AWS services for processing and storage, and containerization with Docker, this project streamlines the workflow from data acquisition to media conversion. Additionally, Terraform scripts are used to provision and manage the necessary AWS infrastructure in a scalable, repeatable manner.

## Key Components and Workflow
RapidAPI Integration:
Utilizes RapidAPI to obtain NCAA game highlights. The highlights data is fetched based on specified dates and leagues (with NCAA as the example) and stored as a JSON file in an S3 bucket.

Docker Containerization:
The project is encapsulated within a Docker container, ensuring consistency across various environments (development, staging, production).

AWS MediaConvert Processing:
Once a highlight is fetched, AWS MediaConvert is employed to process the video file—adjusting codec, resolution, bitrate, and audio settings. The processed video is then stored back in an S3 bucket.

Infrastructure as Code:
Terraform scripts automate the creation of AWS resources such as S3 buckets, IAM roles, Elastic Container Registry (ECR), and Elastic Container Services (ECS), ensuring a robust and scalable deployment.

## File Structure Overview
- config.py:

  - Imports required environment variables and assigns them to Python variables.
  - Provides default values for flexible configuration across different environments.

- fetch.py:

  - Determines the date and league (NCAA) to target for highlights.
  - Fetches highlights from RapidAPI and saves the resulting JSON file (e.g., basketball_highlight.json) to an S3 bucket.

- process_one_video.py:

  - Connects to the S3 bucket to retrieve the JSON file.
  - Extracts the first video URL from the JSON data.
  - Downloads the video using the requests library and saves it to a designated folder (videos/) in the S3 bucket.
  - Logs the status of each processing step.
  
- mediaconvert_process.py:

  - Initiates and submits a MediaConvert job to process the downloaded video.
  - Configures video settings (codec, resolution, bitrate) and audio settings. 
  - Stores the processed video back into an S3 bucket.

- run_all.py:

  - Orchestrates the execution of all scripts in sequence, incorporating buffer times to ensure smooth task creation and processing.

- .env File:

  - Contains environment-specific variables that are kept separate from the source code, facilitating secure and flexible configuration.

- Dockerfile:

  - Provides detailed instructions for building the Docker image, ensuring that the application runs consistently across different environments.

- Terraform Scripts:

  - Automate the provisioning of AWS resources (e.g., S3, IAM roles, ECR, ECS), enabling a scalable and repeatable infrastructure setup.
 
## Project Structure:

    src/
    ├── Dockerfile
    ├── config.py
    ├── fetch.py
    ├── mediaconvert_process.py
    ├── process_one_video.py
    ├── requirements.txt
    ├── run_all.py
    ├── .env
    ├── .gitignore
    └── terraform/
      ├── main.tf
      ├── variables.tf
      ├── secrets.tf
      ├── iam.tf
      ├── ecr.tf
      ├── ecs.tf
      ├── s3.tf
      ├── container_definitions.tpl
      └── outputs.tf

## Architectural Diagram

![Screenshot 2025-02-05 204705](https://github.com/user-attachments/assets/50892af9-d626-4d73-82ba-feef8971ec20)


## Prerequisites

Before running the scripts, ensure that you have completed the following:
1. Create a RapidAPI Account:
- Sign up at RapidAPI.com to gain access to highlight images and videos.
- For this project, we use NCAA (USA College Basketball) highlights, which are available for free with the basic plan.
- The Sports Highlights API endpoint will be utilized to fetch the highlights.

2. Confirm Software Installation:
- Docker: Verify that Docker is installed by running 
    docker --version.
- AWS CLI: AWS CloudShell comes pre-installed with the AWS CLI; check it with 
    aws --version.
- Python 3: Ensure Python 3 is available using 
    python3 --version.

3. Retrieve Your AWS Account ID
- Log in to the AWS Management Console.
- Click on your account name in the top right corner to view your AWS Account ID.
- Copy and securely store this ID, as it will be needed for later configuration steps.

4. Obtain AWS Access Keys
- Navigate to the IAM dashboard in the AWS Management Console.
- Under the Users section, select a user and click on the Security Credentials tab.
- Scroll to the Access Key section. Note that while you can view the access key ID, the secret access key cannot be retrieved once created. If you don’t have the secret stored, you will need to generate a new access key.

## Local Setup Instructions
### Step 1: Clone the Repository
    git clone https://github.com/dominique-jacque/Game-Highlight-Processor.git
    cd src

### Step 2: Add Your API Key to AWS Secrets Manager
Replace YOUR_ACTUAL_API_KEY with your real API key and run:

    aws secretsmanager create-secret \
        --name my-api-key \
        --description "API key for accessing the Sports Highlights API" \
        --secret-string '{"api_key":"YOUR_ACTUAL_API_KEY"}' \
        --region us-east-1

### Step 3: Create an IAM Role or User
Open the AWS Management Console and search for IAM.
Navigate to Roles and click Create Role.
For the use case, select S3 and proceed.
Under Add Permissions, search for and attach the following policies:
AmazonS3FullAccess
MediaConvertFullAccess
AmazonEC2ContainerRegistryFullAccess
In the Role Details section, name the role HighlightProcessorRole and create it.
Once the role is created, locate it in the list, click on it, and then go to the Trust relationships tab. Edit the trust policy by replacing it with the following (ensure you replace <your-account-id> and <your-iam-user> with your actual details):

    {
     "Version": "2012-10-17",
      "Statement": [
        {
         "Effect": "Allow",
          "Principal": {
            "Service": [
              "ec2.amazonaws.com",
             "ecs-tasks.amazonaws.com",
              "mediaconvert.amazonaws.com"
           ],
           "AWS": "arn:aws:iam::<your-account-id>:user/<your-iam-user>"
         },
          "Action": "sts:AssumeRole"
       }
     ]
    }

### Step 4: Update the .env File
Open the .env file and set the following variables:

RapidAPI_KEY: Your RapidAPI key (ensure you have subscribed to test the Sports Highlights API).
AWS_ACCESS_KEY_ID: Your AWS access key ID.
AWS_SECRET_ACCESS_KEY: Your AWS secret access key.
S3_BUCKET_NAME: The name of your S3 bucket.
MEDIACONVERT_ENDPOINT: Your MediaConvert endpoint (retrieve it using aws mediaconvert describe-endpoints).
MEDIACONVERT_ROLE_ARN: The ARN for the IAM role (e.g., arn:aws:iam::your_account_id:role/HighlightProcessorRole).

### Step 5: Secure the .env File
Restrict permissions on the .env file to protect your credentials:
    chmod 600 .env

### Step 6: Build and Run the Docker Container Locally
Build the Docker image:
    docker build -t highlight-processor .
Run the Docker container:
    docker run --env-file .env highlight-processor
Running the container will execute fetch.py, process_one_video.py, and mediaconvert_process.py. The following files should be uploaded to your S3 bucket:

Optional: Verify that the raw video is present at s3://[your_bucket]/videos/first_video.mp4.
Optional: Verify that the processed video is available at s3://[your_bucket]/processed_videos/.
