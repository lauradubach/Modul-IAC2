Zuerst folgendes installieren:
```bash
pip install awscli
pip install boto boto3
pip install ansible-core
```

Dann denn key hintelegen:
`aws configure`

File erstellen inventory.aws_ec2.yml und folgendes im file hinterlegen:
```bash
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
```
!Wichtig es muss eine Instance im AWS vorhanden sein! 

Zum Schluss diesen Befehl zum Ausführen angeben:
`ansible-inventory -i inventory.aws_ec2.yml --list`