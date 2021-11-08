## Overview

Terraform and pipelines facilitating infrastructure-as-code deployment and environmental/blast radius isolation, currently deploying into a single account. Backend configuration is stored in encrypted, versioned S3 bucket with DynamoDB state locking, leveraging terraform workspaces and the latest version (0.14.9 as of writing).

The terraform environment/main files are stored in this repository. Terraform modules which are called by these main files are stored in the [terraform-importable-modules](https://github.com/forus-coop/terraform-importable-modules_) repo.  


## Directory Structure 
```.
├── README.md                         <---- Documentation                      
└── tf                                              
    └── envs                          <---- Environment folders, corresponds with branch/tf workspace name
        └── prod                      <---- Prod environment configuration, commits made to prod branch  
            ├── main.tf               <---- Main terraform configuration with module instantions
            ├── providers.tf          <---- Provider configuration (terraform regions, roles)  
            └── terraform.tf          <---- Terraform backend configuration
        └── dev                       
            ├── main.tf               
            ├── providers.tf             
            └── terraform.tf
```
  
## Github Actions
Each `.github/workflows/*.yaml` file corresponds with a particular terraform action.

1. plan.yaml - **Runs on every push action on non-master branches:** clones the repo, creates/selects a terraform workspace called `branch name`, performs terraform fmt, init, validation and runs a plan on the `tf/envs/<branch name>`. 
2. pr-comment.yaml - **Runs on PR opened, reopened, synchronize or review_requested:** performs the same actions as `plan.yaml` but attaches plan as PR comment. 
3. apply.yaml - **Runs on merged PR to master branch:** clones the repo, creates/selects a terraform workspace called `branch name`, performs terraform fmt, init, validation, plan, and then applies the plan on the `tf/envs/<originating branch name>`.
4. destroy.yaml - **Runs on dispatch:** - clones the repo, selects the correct workspace and runs a destroy on all infrastructure within the workspace. **USE CAREFULLY! TODO: Add targeted destroy capability**

### Example: adding new infrastructure to an existing environment
In this example we are adding some additional infrastructure to the dev workspace:
1. Checkout the dev branch
2. Modify the `tf/envs/dev/main.tf` file with your new terraform code/module call.
3. Commit the changes to dev branch - terraform plan will run.
4. Validate the plan reflects your desired changes, and make any modifications as necessary.
5. Once satisified with plan output, submit a PR to master branch. 
6. PR will add plan as comment for easier review.
7. Once PR has passed review, it is merged to master, where the plan is applied to the `dev` workspace.  

### Example: adding new infrastructure to a new environment
In this example we are creating a new environment, called `stage`:
1. Create a new `stage` branch from master.
2. Create a new `tf/envs/stage` folder, it is suggested to copy the existing providers.tf and terraform.tf from another folder. The only instance where these files would need to be modified is if deploying to a different backend and/or account (terraform workspace provide us with backend isolation within a single bucket).
3. Create/modify the `tf/envs/stage/main.tf` with your desired infrastructure or module calls.
4. Commit the changes to stage branch - terraform plan will run using a newly created `stage` workspace.
5. Validate the plan reflects your desired changes, and make any modifications as necessary.
6. Once satisified with plan output, submit a PR to master branch. 
7. PR will add plan as comment for easier review.
8. Once PR has passed review, it is merged to master, where the plan is applied to the `stage` workspace. 
