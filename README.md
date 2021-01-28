# Nested Changeset Bug

Around the time of re:Invent 2020, AWS introduced a new feature to generate changesets across a group of nested stacks. This was of particular interest to me as at the time I was working at an organisation that has a lot of nested stacks.

However, when I first tried it out, I came across a nasty bug which I duly reported via the organsiation's enterprise support plan. This repo contains the CloudFormation submitted to AWS support that reproduces this bug. The bug was acknowledged and a fix estimate given for March 2021.

If you would like to see for yourself whether or not the bug is fixed, give this a go!

## Steps to Reproduce

1. Package and deploy the stack, setting the DeployRule parameter to 0
1. From the CloudFormation Console, select Create Changeset against the root stack, using current template
1. Set the DeployRule parameter value to 1
1. Ensure "Change sets for nested stacks" is selected
1. Create the changeset

Now examine the changesets created across the nested stacks. You will observe the following

1. The root stack shows that SecurityGroupStack will be changed
1. Click through to the SecurityGroupStack changeset
1. Observe that the security group will be REPLACED (should not be)
1. Check the JSON changes. Observe that the change reason is parameter VpcId
1. Now go to the VPC nested stack and note that changeset is failed due to "The submitted information didn't contain changes."

If we now go ahead and execute this changeset, then the correct update happens. The new rule is added and the
security group is NOT replaced.

Imagine my surprise when I tried this on a stack with a dozen nested stacks, each with many resources depending on the VpcId
from a VPC stack that was unchanged. I saw 100+ resources across these stacks that were all claiming that they needed REPLACEMENT
due to a perceived change in VpcId

## Reproduction using PSCloudFormation

If you have my [PSCloudFormation](https://github.com/fireflycons/PSCloudFormation) (v4.1.6.6 or higher), it is super-simple to deploy this stack

```powershell
New-PSCFNStack -StackName nested-changeset-bug -TemplateLocation .\root-stack.yaml -DeployRule 0
New-PSCFNChangeSet -StackName nested-changeset-bug -UsePreviousTemplate -IncludeNestedStacks -ShowInBrowser -DeployRule 1
```
