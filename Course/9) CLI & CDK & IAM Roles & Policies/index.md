# CLI & CDK & IAM Roles & Policies

## 1) Dry Run (check the permissions):
Sometimes, we’d just like to make sure we have the permissions.
Some AWS CLI commands (such as EC2) can become expensive if they
succeed, say if we wanted to try to create an EC2 Instance.
Some AWS CLI commands (not all) contain a **--dry-run** option to
simulate API calls. 

**Run certain ec2 instance command:**
```cli
aws ec2 run-instances --dry-run - image-id ami-06340c8c12baa6a09 -- instance-type t2.micro
```

## 2) STS Decode Errors:
When you run API calls and they fail, you can get a long error message.
This error message can be decoded using the STS command line: **sts decode-authorization-message**.

**Encoded authorization failure message:**
```text
4Ai9qvVQYCx3kKoQaOtnKpzKCvrDlzzn_8R3vhbWP-TGw5hHcV-nA1oa2OYqRJGLTVXT5ObYDbfI1VxJv6wIxIkcDra0j
mVs3wbRel4fvv9Q6qQp8MepKs7lPvJuRuSyUzL90NpNNwImtrXQgoeipGU_aeZiaBI35CU2rhuykvMtwY9HHOH5tJzCmj
v-wmQmRcuk5opQkOuipSzyoCSNne-0QekiaUe_vBGcfmSEK1lyc9xU9YndY8aXAz5sI5LOKUz9PBoiLs_ph8zKW0Nv1GH
jjsXjoMrHp5uSBoECyWAazFpBsH7TzIVNdnp9NExuq68e5GqpOpjc2__wSrlRntWRkL9RKcAIy_Eg-eoy8emmF9iDRGwe
li0lytGJFnnf_h7RpTI9UtE8nV7FXcdcbXrX6fmgRWW87DyyfelMJuCcG5AWXWuJY37Cco4r90QqB8c9EALL5R4wAdHl
qyPQl3hyVng21GHo6ZETWCNzR28jDlwpnfamvZoUunqdIvbmNWAYQgkFIccfER9snJTCNUF-Srkkc2QhEzBFpNK9HokCz
i06m0u7CxPM4Az-1M1Zu84rV0fXcLawmyDZQzXoRkSVY5fnvd-LPwc2SOKK98hP8bO7M1iIiFIZAGko4clebWclbqHA1H
MPH5p6JxozIPWAaxdACoPy48XAz-rzQAGKq83ASRYtO2BIEUYrbZCZEHpNdn8J1KLtZOK5nS8_xRGS-mqx4XliSv3lXhq
HfABtEIuEd6wWNyRGShKnaY8srUtOX6v2dzbLe5DLrj-tIFsuBdbq1Fmj5P5qGgiw05b7
```

**Decrypt the message from the CLI using the following command:**
```text
aws sts decode-authorization-message --encoded-message 4Ai9qvVQYCx3kKoQaOtnKpzKCvrDlzzn_8R3vhbWP-TGw5hHcV-nA1oa2OYqRJGLTVXT5ObYDbfI1VxJv6wIxIkcDra0j
mVs3wbRel4fvv9Q6qQp8MepKs7lPvJuRuSyUzL90NpNNwImtrXQgoeipGU_aeZiaBI35CU2rhuykvMtwY9HHOH5tJzCmj
v-wmQmRcuk5opQkOuipSzyoCSNne-0QekiaUe_vBGcfmSEK1lyc9xU9YndY8aXAz5sI5LOKUz9PBoiLs_ph8zKW0Nv1GH
jjsXjoMrHp5uSBoECyWAazFpBsH7TzIVNdnp9NExuq68e5GqpOpjc2__wSrlRntWRkL9RKcAIy_Eg-eoy8emmF9iDRGwe
li0lytGJFnnf_h7RpTI9UtE8nV7FXcdcbXrX6fmgRWW87DyyfelMJuCcG5AWXWuJY37Cco4r90QqB8c9EALL5R4wAdHl
qyPQl3hyVng21GHo6ZETWCNzR28jDlwpnfamvZoUunqdIvbmNWAYQgkFIccfER9snJTCNUF-Srkkc2QhEzBFpNK9HokCz
i06m0u7CxPM4Az-1M1Zu84rV0fXcLawmyDZQzXoRkSVY5fnvd-LPwc2SOKK98hP8bO7M1iIiFIZAGko4clebWclbqHA1H
MPH5p6JxozIPWAaxdACoPy48XAz-rzQAGKq83ASRYtO2BIEUYrbZCZEHpNdn8J1KLtZOK5nS8_xRGS-mqx4XliSv3lXhq
HfABtEIuEd6wWNyRGShKnaY8srUtOX6v2dzbLe5DLrj-tIFsuBdbq1Fmj5P5qGgiw05b7
```

**Result:**
```text
{
 "allowed": false,
 "explicitDeny": false,
 "matchedStatements": {
 "items": []
 },
 "failures": {
 "items": []
 },
 "context": {
 "principal": {
 "id": "APOZIAANAVSK6I6FK2RQI:i-66c78ee7",
 "arn": "arn:aws:sts::<aws-account-id>:assumed-role/my-role-ec2/i-123456e7"
 },
 "action": "iam:PassRole",
 "resource": "arn:aws:iam::<aws-account-id>:role/my-role-ec2",
 "conditions": {
 "items": []
 }
 }
}
```

## 3) EC2 Instance Metadata:
It allows AWS EC2 instances to ”learn about themselves” without using an IAM Role for 
that purpose. You can retrieve the IAM Role name from the metadata, but you CANNOT
retrieve the IAM Policy. For it, you can call **http://169.254.169.254/latest/meta-data**
from your ec2 CLI.

```text
curl http://169.254.169.254/latest/meta-data
```

**Result (different metadata that you can get):**
```text
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
events/
hostname
iam/
instance-action
instance-id
instance-life-cycle
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/
```

## 4) MFA with CLI:
To use MFA with the CLI, you must create a temporary session. To do so, you must run
the **STS GetSessionToken API** call.
