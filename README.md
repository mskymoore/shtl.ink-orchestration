# **shtl.ink orchestration**

This repository tracks the commits of deployed infrastructure, orchestration, and code components of [shtl.ink](https://shtl.ink)

## **Deploy**
**1. Deploy the kubernetes cluster.**
   1. Create the backend bucket in s3 and configure ```versions.tf```, and ```variables.auto.tfvars``` with the appropriate region and bucket name.  Be sure to deny all public access to the bucket.  The bucket policy should look like the following:
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::ACCOUNT_ID:root"
                    },
                    "Action": "s3:ListBucket",
                    "Resource": "arn:aws:s3:::BUCKET_NAME"
                },
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::ACCOUNT_ID:root"
                    },
                    "Action": [
                        "s3:GetObject",
                        "s3:PutObject",
                        "s3:DeleteObject"
                    ],
                    "Resource": "arn:aws:s3:::BUCKET_NAME/terraform.tfstate"
                }
            ]
        }
        ```
   2. Initialize and apply the terraform.
      ```bash
      cd terraform-eks-cluster
      # might have to delete .terraform.lock.hcl
      terraform init
      # while this is running, move on to building the docker images
      terraform apply
      ```
**2. Build the docker images.**  
   1. Create the env.local file for the frontend image
        ```bash
        touch shtl.ink/env.local
        ```
   2. Populate the file, contents should be like the following
        ```bash
        APP_NAME=shtl.ink
        COOKIE_DOMAIN=.shtl.ink
        BASE_URL=https://shtl.ink
        API_BASE_URL=https://api.shtl.ink
        ```
   3. Build and push the images
        ```bash
        # in a new terminal
        cd api.shtl.ink
        docker build -t user/repo:tag .
        docker push user/repo:tag
        cd ../shtl.ink
        docker build -t user/repo:tag .
        docker push user/repo:tag
        ```  
**3. Deploy the kubernetes into the cluster.**  
   1. Update ```shtl.ink-kubernetes/shtl-ink/0500_shtl-ink_deployment.yml``` with the appropriate images and tags that were pushed to dockerhub in the previous step. It's important that the urls configured in this deployment file, match what is configured for the front end image build.
   2. Check that the terraform has finished applying, once it's applied move to the next step.
   3. Follow the steps in ```shtl.ink-kubernetes/Readme.md```

**4. Update any DNS records as necessary.**
