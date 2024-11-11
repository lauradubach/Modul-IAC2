# Mein vorgehen

Auf de ansible VM folgends installieren:

`sudo apt install awscli`
`sudo apt install ansible`
`ansible-galaxy collection installcommunity.aws`

Nun die AWS Details aus dem LAB angeben:

`aws configure`
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: 
Default output format [None]: 

Nun den key generieren und importieren:
`ssh-keygen -t rsa -b 2048 -f ~/.ssh/aws_ansible_key -N ""`
`aws ec2 import-key-pair --key-name "my-ansible-key" --public-key-material fileb://~/.ssh/aws_ansible_key.pub --region us-east-1`
`aws ec2 describe-key-pairs --region us-east-1`

