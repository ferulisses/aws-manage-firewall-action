# aws-manage-firewall-action

This script is used to open the AWS Firewall so the server that is buiding the App can access SSH on AWS EC2 servers without leaving the Firewall ports (such as SSH port) open worldwide and without need to allow all Github Deploy Servers IP Address allowed to connect to your servers.

This is done adding the Github Deploy Server IP Address to an AWS Security Group rule attached to the VMs.

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

After creating the Security Group, or if you already have one, get the Rule Id or the Rule Name
- Access EC2 Service
- On the menu on left, in "Network and Security" -> "Security Groups"
- Find and click the Security Group attached to the Instances
- For the rule id copy the field Security Group ID, normaly a string: sg-xxxxxxxxxxxxxx
- For the rule name copy the field Security group name, normaly a string: launch-wizard-xx (or custom name if you defined one in the previous step)

On your Github Repository, in Settings -> Secrets, create:
AWS_ACCESS_KEY_ID with your Key
AWS_SECRET_ACCESS_KEY with your Secret

On your Github Action, use:

```
    steps:
      - id: open-firewall
        uses: ferulisses/aws-manage-firewall-action@v1.3
        with:
          rule-id: 'sg-xxxxxxxx'
          rule-name: 'launch-wizard-xx'
          port: 22
          protocol: tcp
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: 'us-east-1'  # adjust to your region
          
      # add here actions that will access the VM, such as SSH, Rsync, or even Database connections
      
      # the firewall will close automatically at the end
```

## Inputs

| Input | Required | Description |
| --- | --- | --- |
| `aws-access-key-id` | `true` |  |
| `aws-secret-access-key` | `true` |  |
| `aws-region` | `false` | Defaults us-east-1 |
| `rule-id` | `false` | Security Group ID you got from the previuos step |
| `rule-name` | `false` | Security Group Name you got from the previous step |
| `port` | `false` | All the ports are used if you don't define this parameter |
| `protocol` | `false` | Possible values: tcp\|udp\|icmp\|all. Default: all, if you set a specific port you should define a protocol in order to get the expected results |

- One of the parameters rule-id or rule-name must be defined, if you use both the rule-id param has preference over rule-name