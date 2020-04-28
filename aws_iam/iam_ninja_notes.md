# AWS IAM

tl;dr Use the Visual Editor :smile:

## What are IAM policies?

* provide authorization to AWS services and resources
* JSON-formatted
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

* **Principal**:
  - User or resource you want to deny or allow access
  - Indicated by an Amazon Resource Name (ARN)
  - With IAM policies, the principal element is implicit
  ```
  // Specific accounts
  "Principal":{"AWS":"arm:aws:iam::123123:root"}
  "Principal":"AWS":"arm:aws:iam::123123:user/username"
  "Principal":{"AWS":12367987}
  ```
* **Action**: Type of access that is allowed or denied access
  - [List of Actions](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html)
  - Statements must include either `Action` or `NotAction` (usually the first)
  ```
  "Action":"ec2:StartInstances"
  "Action":"iam:ChangePassword"
  ```
* **Resource**: The resources the action will be applied to
  - Statements must include either Resource or NotResource
  ```
  "Resource":"arn:aws:s3:::my_bucket/*"

  // All except this one
  "NotResource":"arn:aws:s3:::my_special_bucket/*"
  ```
* **Condition**: Determines when the access defined is valid
  - Reading conditions:
    - vertical: AND
    - horizontal: OR
  - All conditions must be met for the statement to evaluate to `true`

## Policy Enforcement and Evaluation

```
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
```

=> What happens between the request and the defined policies is just matching.

## Policy types

* *AWS Organizations*: Groups your (multiple) accounts
  - **Service Control Policies (SCP)**: controls which services and actions
    an account can use. Default is a `*` policy for every organization, relying on IAM for access control.
  *Why use it?* Guardrails on the account to disable access to services
  ---
* *AWS Identity and Access Management (IAM)*: actually able to define access to resources,
    services, etc.
  - **Inline policies**: Attached to the user or role, not moveable
  - **Managed policies**: Agnostic from user or role, can be attached to multiple entities
  - **Permission Policies and Permission Boundaries**: access can be limited with a certain
    maximum threshold amount of permissions
  *Why use it?* Set granular permissions based on functions that users or apps need to perform
  ---
* *AWS Security Token Service (AWS STS)*: used when assuming IAM roles
  - **Scoped-down policies**: can be used to limit permissions of a role by session
  *Why use it?* Reduce general shared permissions further
  ---
* *Specific AWS services*: policies only specific to certain services
  - **Resource-based policies**: control access from a specific resource like S3
  *Why use it?* Cross-account access and to control access from the resource

Bonus:
* *VPC Endpoints*: controls access to the service with a VPC endpoint
  - **Endpoint policies**: [same as above]

=> All use the same DSL/policy language!

## How policies work together

### Within an account

```
SPC && (IAM policies || Resource-based policies)
            *if*
            Permissions boundary
            &&
            Permissions Policy (Managed || Inline)
            &&
            Scope-down policy
```

### Across accounts

```
SPC && (IAM policies && Resource-based policies)
            *if*
            Permissions boundary
            &&
            Permissions Policy (Managed || Inline)
            &&
            Scope-down policy
```

## Pro Tips:

* Default pessimistic, default deny
* Use SCPs for things you absolutely don't want anyone to do.
* Boundaries help to not feel like having to watch everyone
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
* Use tags! (Like on anything in AWS)
  - RequestTag: require specific tag value during create actions
  - ResourceTag: control access to resources based on a tag that's on a resource
  ```
  {
    "Effect": "Allow",
    "Action": "ec2:CreateTags",
    "Resource": "*",
    "Condition": {
      // Only tag ec2 resources with your project tag
      "StringEquals": {
        "ec2:ResourceTag/project": ["${aws:PrincipalTag/project}"]
      },
      // Only tag with either of these keys
      "ForAllValues:StringEquals": {
        "aws:TagKeys": ["project", "name"]
      },
      // Specify your project tag for <project>
      "StringEqualsIfExists": {
        "aws:RequestTag/project": ["${aws:PrincipalTag/project}"]
      }
    }
  }
  ```
  ^ You're only able to tag resources tagged with the specified project, but only with the same tag!
    No tag voodoo anymore!

  ```
  {
    "Effect": "Allow",
    "Action": ["ec2:RunInstances"],
    "Resource": ["arn:aws:ec2:*:*:instance/*"],
    "Condition": {
      // Only tag with either of these keys
      "ForAllValues:StringEquals": {
        "aws:TagKeys": ["project", "name"]
      },
      // Requires <project> to have one of these values
      // and requires instances to be t2.micro
      "StringEquals": {
        "aws:RequestTag/project": ["blackjack", "poker"],
        "ec2:InstanceType": "t2.micro"
      }
    }
  }
  ```
  ^ You're only able to tag when launching t2.micro instances, and only with the specified project tags.

* You can tag the IAM users and roles as well
* Any condition key can also be used as a variable as a condition value for string operators
  - `["${aws:PrincipalTag/project}"]`
