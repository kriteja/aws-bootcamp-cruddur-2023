# Week 0 — Billing and Architecture
## Homework Tasks
1. [AWS Command Line Interface (AWS CLI) installation & verification on GitPod and Desktop](#1-aws-cli-installation--verification-on-desktop)
    - [AWS CLI installation on Desktop](#aws-cli-installation-on-desktop)
    - [AWS CLI installation on Gitpod](#aws-cli-installation-on-gitpod)
        - [Create IAM user and Generate AWS Credentials](#create-iam-user-and-generate-aws-credentials)
        - [Set Env Vars](#set-env-vars)
3. [Create a Budget in AWS Account](#2-create-a-budget-in-aws-account)
4. [Recreate Conceptual Architectural Deisgn](#3-recreate-conceptual-architectural-deisgn)
5. [Recreate Logical Architectural Deisgn](#4-recreate-logical-architectural-deisgn)



### 1. AWS CLI Installation & Verification on Desktop 
###### AWS CLI installation on Desktop. 
[AWS Official Documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) for installing AWS CLI on Windows using Command Prompt.
```
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```
- Verify the installation by comand `aws`
- If you receive below error, try relaunching the Command Prompt. 

``` 
'aws' is not recognized as an internal or external command, operable program or batch file.
```

![AWS CLI in Win10](https://user-images.githubusercontent.com/40818088/219865329-ca746071-0c9b-47a8-8d2c-d4fae780e76f.PNG)

###### AWS CLI installation on Gitpod

- Link Github with Gitpod - [Official documentation for linking](https://www.gitpod.io/docs/configure/authentication/github) 
- Since Gitpod terminal runs on `Linux`run the below commands

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

- Once the CLI is installed, verify the ame using `aws` command. 
- Update our `.gitpod.yml` file the below task

```sh
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```
- Verify the installation in terminal using `aws` command 
###### Create IAM user and Generate AWS Credentials
- Go to IAM Users Console
- Create a User Group called `Admin` and apply `AdministratorAccess`
- Create the user and add the user to the group. 
- Click on `Security Credentials` and `Create Access Key`
- Choose AWS CLI Access
- Download the CSV with the credentials

###### Set Env Vars
- Register your AWS credentials using 

```
gp env AWS_ACCESS_KEY_ID=""
gp env AWS_SECRET_ACCESS_KEY=""
gp env AWS_DEFAULT_REGION=us-east-1
``` 
- Verify the same using 
```sh
aws sts get-caller-identity
```
- You should receive the below response

```
{
    "UserId": "AIDAWJ5PFJ73Y32HHCMWQ",
    "Account": "433622175735",
    "Arn": "arn:aws:iam::433622175735:user/Krishna_iam"
}
```

### 2. Create a Budget in AWS Account

- Created a Budget for $1.

![Budget Overview](https://user-images.githubusercontent.com/40818088/219865691-3577831a-bbb7-4fd6-b5f8-f3240cafe683.PNG)

- Budget alerts

![Budget Alerts](https://user-images.githubusercontent.com/40818088/219874656-1f51ea32-4149-417b-bacf-61cc1c6f1652.PNG)

### 3. Recreate Conceptual Architectural Deisgn

- A rough Conceptual Architectural representation of Cruddur App. 
    - Lucid chart link - [Conceptual Architectural Diagram](https://lucid.app/lucidchart/ade1b5c0-f197-4531-af69-595fd108fab4/edit?viewport_loc=-458%2C-456%2C2936%2C1381%2C0_0&invitationId=inv_661ffc19-233b-4456-a625-e1559d3d2140)

![Conceptual Diagram](https://user-images.githubusercontent.com/40818088/219865768-9c0a6ee4-e9cf-496f-88b3-544c72759ba6.PNG)

### 4. Recreate Logical Architectural Deisgn

- A detailed Logical Architectural Deisgn of Cruddur App. 
    - Lucid chart link - [Logical Architectural Diagram](https://lucid.app/lucidchart/ade1b5c0-f197-4531-af69-595fd108fab4/edit?viewport_loc=-1600%2C-621%2C3500%2C1647%2CQqkyQg5f_pXj&invitationId=inv_661ffc19-233b-4456-a625-e1559d3d2140)

![Logical Diagram 1](https://user-images.githubusercontent.com/40818088/219873927-18dbcad0-e74e-4c58-b26a-9f39b303bae5.PNG)




