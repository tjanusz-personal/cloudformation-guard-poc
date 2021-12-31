# cloudformation-guard-poc

This project contains POC information on cloudformation guard policy as code tool.

AWS cloud formation security tool (https://github.com/aws-cloudformation/cloudformation-guard)

The project contains the following directories:
* **rules** - sample guard rule definitions (basic and advanced)
* **sample_templates** - example application templates (e.g. global id, illustrations, etc.) to use as samples for either tool
* **tests** - sample guard unit tests for various rules defined

## Development environment setup
Refer to the cloudformation-guard github site for how to download and configure the tool for local use.

```bash
$ brew install cloudformation-guard
```

## Running sample Validation commands
To run various checks against templates use the 'validate' command.

```bash
# run all basic checks against all files in the sample_templates directory
cfn-guard validate --data sample_templates --rules rules/basic

# run the global id template against all basic checks
cfn-guard validate --data sample_templates/gid_template.yaml --rules rules/basic

# run the illustrations calc API template against all advanced checks
cfn-guard validate --data sample_templates/illustrations_calc_api_template.yaml --rules rules/advanced

# run the global id template with all 'dev' tests (output to Json format)
cfn-guard validate --data sample_templates/gid_template.yaml --rules rules/basic/all_dev_checks.guard -o json

# run the claims ui template against all dev level checks
cfn-guard validate --data sample_templates/claims-ui-template.yaml --rules rules/basic/all_dev_checks.guard -o json
```

## Running sample unit tests
To run various unit tests for each type use the 'test' command

```bash
# run tests against basic sqs
cfn-guard test -r rules/basic/sqs_queue_security.guard -t tests/basic/sqs_queue_security_tests.yaml

# run tests against advanced api gateway
cfn-guard test -r rules/advanced/sam_api_gateway_deny_global.guard -t tests/advanced/sam_api_gateway_deny_global_tests.yaml

```

## Sample Rules
The POC attempted to code some basic and advanced rules for certain AWS services currently being evaluated or used by teams.

The current AWS services include:
* **sqs**
* **s3**
* **api gateway**

Sample rule ideas were obtained by referring to the [trend micro cloudformity knowledge-base](https://www.cloudconformity.com/knowledge-base/)


## Jenkins Integration
TODO: need to figure this out

## Known limitations/questions
This section lists any relevant findings
* SAM template guards will be slightly different than pure Cloud formation template files (e.g. AWS:Lambda:Function != AWS::Serverless::Function) so need to be super specific on the properties we're guarding for (e.g. Name is OK for both but not others)
* Jekins build integration is at the 'process' level only (e.g. exit code 1 vs. 0)
* Validations require well formed template files (e.g. ansible template .j2 files problematic)
* Validations only work on the exact file so if the template uses cloud formation parameters in some values these are NOT evalulated during the validation process!
```
Parameters:
  CFNEnvPrefix:
    Type: String
    Default: dev

Resources:
    Type: AWS::Lambda::Function
        Properties:
            FunctionName: !Join ["-", ["clarifi-archiving", !Ref CFNEnvPrefix]]  <== not able to put validation here

```
* Ansible based projects would need to design around using this tool to validate errors! For example, Clarifi builds multiple CF files via ansible for each component of architecture during the deployment process this would have to be redesigned to have three steps: generate templates, run validation, deploy templates
* Valdation output is messy/confusing. In order to get something somewhat cohesive I grouped all the rules into a single dev rule file so I could see an easier output. Having them in separate .guard files made it messy.
* Can only run unit tests one at a time. 
* Unit test output is messy and hard to find errors.
* Doesn't support "short-form" of intrinsic functions (e.g. !Join, !Sub) docs state to use Fn::Join or Fn::Sub instead.
* validations don't handle resolving SAM template 'Globals'
```
Globals:
  Function:
    Timeout: 900

Resources:
  FileCheckerFunction:
    Type: AWS::Serverless::Function
    Properties:
     Name: blah
     **** <--  If there was a function check for Timeout this would NOT be found!
```
* Validation errors don't easily indicate which item in CF caused the failure. So having lots of elements where only 1 causes the failure can make debugging the single failure difficult.
* Outputs are cli, json only
* No sonarcube integration
* CLI based integration is (exit 0 on failure)


## TODO
Items requiring followup include:
* Jenkins integration - how to add to jenkins and see if there's a failure?
* Policy repo - Where do we store these policy files for all builds to use?
* Create a lambda that fires on cloudformation stack push (to check and stop deploy or not)