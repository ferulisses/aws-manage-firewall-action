# aws-open-firewall-action

This action will add the Github Deploy Server IP Address to an AWS Security Group rule.

This is used to open the AWS Firewall so the server that is buiding the App can access SSH on AWS EC2 servers without leaving the Firewall ports (such as SSH port) open worldwide and without need to allow all Github Deploy Servers IP Address allowed to connect to our servers.

This action is intended to be used together with aws-close-firewall-action, so you will open the SSH port before access the VMs and close after that.

## requirements:

You need API access to your AWS account

## instructions:

Setup a security group in your AWS account:

(you can skip this step if you already have a security group attached to the VM such as the 'default' group)
- Login to AWS Console
- Access EC2 Service
- On the menu on left, in "Network and Security" -> "Security Groups"
- Click on "Create Security Group"
  - on Name, choose a simple name, such as "github-deploy" (this is our default by the way)
  - leave Description and Rules all empty, create the Security Group
- On the menu on left, go to "Instances" -> "Instances"
- Select the instances that you want to allow access
- Click on Actions menu -> Security -> Change Security Groups
  - search for the security group and Associate it to the selected Instances, click Save

On your Github Repository, in Settings -> Secrets, create:
AWS_ACCESS_KEY_ID with your Key
AWS_SECRET_ACCESS_KEY with your Secret

On your Github Action, use:

```
    steps:
      - id: open-firewall
        uses: ferulisses/aws-manage-firewall-action@v1
        with:
          rule-name: 'github-deploy'  # use the same name that you created on AWS Console, github-deploy is the default so you can ommit it
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: 'us-east-1'  # adjust to your region, us-east-1 is the default so you can ommit
          action: 'open'  # open is the default, so you can ommit it
          
      # add here actions that will access the VM, such as SSH, Rsync, or even Database connections
      
      - id: close-firewall
        uses: ferulisses/aws-manage-firewall-action@v1
        with:
          rule-name: 'github-deploy'  # use the same name that you created on AWS Console
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: 'us-east-1'  # adjust to your region 
          action: 'close'
      
```
