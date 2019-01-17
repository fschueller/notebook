# AWS IAM

## What are IAM policies?

* provide authorization to AWS services and resources
* Two parts:
  - Specification: Definining access policies
  - Enforcement: Evaluation policies
* **Definition** happens on your side: you specify which IAM principals are allowed to
  perform which actions on specific AWS resrouces and under which conditions.
* **Enforcement** happens on AWS/IAM side: the request to AWS gets evaluated and will
  be returned with a Yes/No answer.

## Policy structure
```
{
  "Statement": [
    "Effect": "effect",
    "Principal": "principal",
    "Action": "action",
    "Resource": "arn",
    "Condition": {
      "condition": {
        "key":"value"
      }
    }
  ]
}
```

* **Principal**: User or resource you want to deny or allow access
* **Action**: Type of access that is allowed or denied access
* **Resource**: The resources the action will be applied to
* **Condition**: Determines when the access defined is valid

## Policy Enforcement and Evaluation

1. Decision starts at *Deny*
2. Evaluate all applicable policies
3. Is there an explicit *Deny*? -> Yes -> Final decision = *Deny*
                          |
                          v
                          No
                          |
    v---------------------+
4. Is there an *Allow*? -> Yes -> Final decision = *Allow*
                  |
                  v
                  No
                  |
    v-------------+
5. Final decision = *Deny* (default Deny)

=> What happens between the request and the defined policies is just matching.

## Policy types

* *AWS Organizations*: Groups your accounts
  - **Service Control Policies (SCP)**: guardrails, not an access control in the true sense.
    Default is a `*` policy for every organization, relying on IAM for access control.
* *AWS Identity and Access Management (IAM)*: actually able to define access to resources,
  services, etc.
  - **Permission Policies and Permission Boundaries**: access can be limited with a certain
    maximum threshold amount of permissions
* *AWS Security Token Service (AWS STS)*: used when assuming IAM roles
  - **Scoped-down policies**: can be used to limit permissions of a role by session
* *Specific AWS services*: policies only specific to certain services
  - **Resource-based policies**: control access from a specific resource like S3
* *VPC Endpoints*: controls access to the service with a VPC endpoint
  - **Endpoint policies**: [same as above]

=> All use the same DSL/policy language!

## How policies work together

### Within an account

SPC **&&** (IAM policies **||** Resource-based policies)
            *if*
            Permissions boundary
            &&
            Permissions Policy (Managed || Inline)
            &&
            Scope-down policy

### Across accounts

SPC **&&** (IAM policies **&&** Resource-based policies)
            *if*
            Permissions boundary
            &&
            Permissions Policy (Managed || Inline)
            &&
            Scope-down policy

## Pro Tips:

* Default pessimistic, default deny
* Use SCPs for things you absolutely don't want anyone to do.
*
* To restrict actions to regions, use `RequestedRegion` condition
  ```
  {
    "Effect": "Allow",
    "Action": [
      "secretsmanager:*",
      "s3:PutObject",
      ......
    ],
    "Resource": "*",
    "Condition": {
      "StringEquals": {
        "aws:RequestedRegion": [
          "eu-central-1",
          "eu-west-1"
        ]
      }
    }
  }
  ```
* Naming conventions!
