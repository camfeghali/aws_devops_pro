# Cloudformation

Infrastructure as Code service defined in YAML or JSON, in what are called **templates**.

## Terms

- **Logical resources**: The resource that is defined in the template.
- **Physical resources**: The actual created logical resource that is defined in the template.

## Things to note

### Portability
It is important to make templates **portable** between regions, accounts, and the number of times it can be deployed. To do that we can use the following features and constructs.

#### Template Parameters
- Are passed when a template is created / updated.
- Can be referenced within logical resources.
- Can be configured with:
    - defaults, allowed values, min/max, allowed patterns, no echo (masks them in console output and logs), data type.

#### Pseudo Parameters
- Are injected by AWS, they include:
    - `AWS::Region`, `AWS::StackId`, `AWS::StackName`, `AWS::AccountId`

### Intrinsic Functions
- `Ref` and `Fn::GetAtt` -> Reference logical resources within a template
- `Fn::Join` and `Fn::Split` -> To join and split strings
- `Fn::GetAZs`, and `Fn::Select` -> To get availability zones and select elements from a list
- `If`, `And`, `Equals`, `Not`, `Or` -> For conditionals. Can control config values, or even if resources get created.
- `Fn::Base64` and `Fn::Sub` -> Allows you to pass strings as values and replace values with sub.
- `Fn::Cidr` -> To create a Cidr range in VPCs. Used to split and assign Cidr ranges from a VPC to subnets
- `ImportValue`, `FindInMap`, `Transform`

#### Mappings
One template can contian many mappings. It's a key value store that can be up to two level deep. You can retrieve values using `Fn::FindInMap`

#### Outputs
You can optionally output values from a stack.
Those outputs are accessible from a parent stack, or be exported for cross-stack references.

#### DependsOn
Allows you to block the provisioning of resources until another is created.

### Customizing resource lifecycles `WaitCondition`, `CreationPolicy`, `CFN-Signal`.
These are constructs that allow you to customize Cloudformation's resource provisioning lifecycle. For example, they allow you to mark resources as `CREATE_COMPLETE` after a certain bootstrapping step has taken place

#### `CFN-Signal`
Is a Linux utility available on resources, ex: EC2 that allows you to tell cloudformation that a physical resource has been successfully created.

You can tell cloudformation to 
- Wait for 'X' number of succes signals.
- Wait for 'Y' amount of time for those signals, (12 hours max).

It can also send `failure` signals -> **creation fails**.
If timeout is reached -> **creation fails**.

#### Creation Policy & Wait Condition
They define how a Cloudformation validates resource creation according to signal reception. Mainly EC2 instances and Auto-Scaling groups.

Basically, this is how you define in the template what it means to consider a resource as `CREATION_COMPLETE`.

So these work together with the `cfn-signal` utiliy to achieve a desired behaviour.

Ex:

In Cloudformation
```
  MyEC2Instance:
    Type: AWS::EC2::Instance
    CreationPolicy:
      ResourceSignal:
        Count: 1                # Expect one signal
        Timeout: PT15M          # Wait for up to 15 minutes for the signal
    Properties:
      ... other properties
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          # Install necessary packages
          yum update -y
          yum install -y httpd

          # Start HTTPD service
          systemctl start httpd
          systemctl enable httpd

          # Signal CloudFormation that bootstrapping is complete
          /opt/aws/bin/cfn-signal --success true \
            --stack ${AWS::StackName} \
            --resource MyEC2Instance \
            --region ${AWS::Region}
```

