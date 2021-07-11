<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/elb/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# ELB CloudFormation Blueprint

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiOE5YTktibDVYcG5KZ0JZeG1ONnhsVk5WaUlmWHVZTldRSi85NG5PRFZOK1doNmRtODhEQjUwdFp1SURIZEI5OHlMT1A5RERiOXFRYTF2d1dBdm0xS2k4PSIsIml2UGFyYW1ldGVyU3BlYyI6IkdhQlNFN0hhd0VqUXpDVnoiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

This blueprint provisions an ELB Load Balancer. Both Application or Network Load Balancers are supported.

* Several [AWS::ElasticLoadBalancingV2::LoadBalancer](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-loadbalancer.html) properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Additionally, properties that require further customization are configurable with [Variables](https://lono.cloud/docs/configs/shared-variables/).  The blueprint is extremely flexible and configurable for your needs.
* By default, an Application ELB is created.
* You can assign existing Security Groups to the ELB or have the blueprint create a managed security group for you.
* A Listener and Target Group is also created and automatically setup.
* Can create an optional managed Route53 record that points to the ELB Endpoint.

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/elb values
3. Deploy blueprint

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "elb", git: "git@github.com:boltopspro/elb.git"
```

## Configure

Use the [lono seed](https://lono.cloud/reference/lono-seed/) command to generate a starter config params files.

    LONO_ENV=development lono seed elb
    LONO_ENV=production  lono seed elb

The files in `config/elb` folder will look something like this:

    configs/elb/
    ├── params
    │   ├── development.txt
    │   └── production.txt
    └── variables
        ├── development.rb
        └── production.rb

Configure the `configs/elb/params` and `configs/elb/variables` files.  The parameters required: `Subnets` and `VpcId`. Example:

configs/elb/params/development.txt:

    Subnets=subnet-111,subnet-222
    VpcId=vpc-111

Note `Subnets` is required when using an Application ELB and you're not using `@subnet_mappings` for precreated static EIPs with a network ELB.

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy.

    LONO_ENV=development lono cfn deploy elb --sure --no-wait
    LONO_ENV=production  lono cfn deploy elb --sure --no-wait

It takes about 5m to deploy the ELB. Times may vary.

If you are using One AWS Account, use these commands instead: [One Account](docs/one-account.md).

## Configure: More Details

### Security Groups

To assign existing security groups to the RDS database use `SecurityGroups`. Example:

    SecurityGroups=sg-111,sg-222

If not set, then the blueprint will create and a managed Security Group and assign it to the ELB.

### Managed Security Group Rules

If you wish to add whitelist rules to the managed security group created by the blueprint, use `@security_group_ingress`. Example:

configs/elb/variables/development.rb:

```ruby
@security_group_ingress = [{
  CidrIp: "0.0.0.0/0", # String
  FromPort: 80, # Integer
  IpProtocol: "tcp", # String
  ToPort: 80, # Integer
},{
  CidrIp: "0.0.0.0/0", # String
  FromPort: 443, # Integer
  IpProtocol: "tcp", # String
  ToPort: 443, # Integer
}]
```

More info: [AWS::EC2::SecurityGroup Ingress](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group-rule-1.html)

### Load Balancer Attributes

You can edit the Load Balancer Attributes with the `@load_balancer_attributes` variable. Example:

configs/elb/variables/development.rb

```ruby
@load_balancer_attributes = [{
  Key: "idle_timeout.timeout_seconds",
  Value: 30
}]
```

More docs: [Filter View:
AWS::ElasticLoadBalancingV2::LoadBalancer LoadBalancerAttribute](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-elasticloadbalancingv2-loadbalancer-loadbalancerattributes.html)

### Route53 DNS Pretty Host Name

It is recommended to create a Route53 pretty endpoint for the ELB Endpoint.  Example:

    DnsName=my-elb.example.com.
    HostedZoneName=example.com.

### HTTPS SSL Termination

To configure the ELB to use HTTPS, configure the `CertificateArn`, `Protocol`, and `Port` parameters. Example:

    CertificateArn=arn:aws:acm:us-west-2:112233445566:certificate/f29db923-7bba-4a9f-9b58-06ba1EXAMPLE # example.com
    Port=443
    Protocol=HTTPS

### Registering Targets

To register targets to the Target Group with code, use the `@targets` variable:

configs/elb/variables/development.rb

```ruby
@targets = [{
  Id: "i-065d6916cc44454d3",
  Port: "8888",
}]
```

The default is `TargetType=instance`. If you registering IP and Port combinations, use `TargetType=ip`.  If you are registering Lambda functions, use `TargetType=lambda`.

### Network Load Balancer

If you wish to create a Network Load Balancer instead of an Application Load Balancer, use `Type=network`.  The Listener `Protocol` and `TargetGroupProtocol` must also be set to [valid supported values](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-targetgroup.html#cfn-elasticloadbalancingv2-targetgroup-protocol) for Network Load Balancers. Example:

    Type=network
    Protocol=TCP # Examples: TCP_UDP, UDP, TCP, TLS
    TargetGroupProtocol=TCP # Examples: TCP_UDP, UDP, TCP, TLS

### Network Load Balancer with Existing Static IPs

If you would like to use static IP addresses from pre-created [Elastic IP Addresses](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html), EIPs, then you can use the `@subnet_mappings` variables. Example:

configs/elb/variables/development.rb

```ruby
@subnet_mappings = [{
  AllocationId: "eipalloc-111",
  SubnetId: "subnet-111"
},{
  AllocationId: "eipalloc-222",
  SubnetId: "subnet-222"
}]
```

Note, you also **must not** use the `Subnets` parameter when using setting the `@subnet_mappings` variable.  We're only allowed set the `Subnets` or `SubnetMappings` property.

### Advanced Properties Customizations

* Several [AWS::ElasticLoadBalancingV2::LoadBalancer](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-loadbalancer.html) properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/).
* More complex properties, can be configurable with
[Variables](https://lono.cloud/docs/configs/shared-variables/).  For example, the `@subnet_mappings` variable allows configure the LoadBalancer SubnetMappings.  Example:

```ruby
@subnet_mappings = [{
  AllocationId: "eipalloc-111",
  SubnetId: "subnet-111"
},{
  AllocationId: "eipalloc-222",
  SubnetId: "subnet-222"
}]
```

Refer to [helpers/variables_helper.rb](app/helpers/variables_helper.rb) for the full list of variables.
