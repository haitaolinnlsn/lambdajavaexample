# The CloudFormation template
CFORM = AWSIoT2Lambda2DynamoDB_CF_demo.yml
# AWS Region
REGION = $(shell aws configure get region)

# Mnemonic is for cross company support
MNEMONIC = nlsn
MNEMONIC_UPPER = $(strip $(shell echo $(MNEMONIC) | awk '{print toupper($0)}'))

# The Account, i.e sandbox | nonprod | prod
ACCOUNT = sandbox
ACCOUNT_LOWER = $(shell echo $(account) | tr  '[A-Z]' '[a-z]')

# The Environment, i.e dev|qa|uat|prod
ENV = dev
ENV_LOWER = $(shell echo $(env) | tr  '[A-Z]' '[a-z]')

PARAM_FILE=$(MNEMONIC)-$(ACCOUNT)-$(ENV)-$(REGION).json

# Resource Name
# CloudFormation S3 Buckets
RESOURCE_NAME=$(shell cat $(PARAM_FILE) | jq -r '.[] \
	| select(.ParameterKey=="ServiceName") | .ParameterValue')
RESOURCE_NAME_LOWER = $(shell echo $(RESOURCE_NAME) | tr  '[A-Z]' '[a-z]')
# CloudFormation S3 Buckets
RESOURCE_BUCKET=$(shell cat $(PARAM_FILE) | jq -r '.[] \
	| select(.ParameterKey=="ResourceBucketName") | .ParameterValue')
TEMPLATE_BUCKET=$(shell cat $(PARAM_FILE) | jq -r '.[] \
	| select(.ParameterKey=="TemplateBucketName") | .ParameterValue')

# CloudFormation Stack Name
STACK_NAME=$(RESOURCE_NAME)

# lists all available targets
list:
	@echo "List of available targets:"
	@sh -c "$(MAKE) -p no_targets__ | awk -F':' '/^[a-zA-Z0-9][^\$$#\/\\t=]*:([^=]|$$)/ {split(\$$1,A,/ /);for(i in A)print A[i]}' | grep -v '__\$$' | grep -v 'make\[1\]' | grep -v 'Makefile' | sort"
# required for list
no_targets__:

estimate-cost:
	@aws cloudformation estimate-template-cost \
		--template-body=file://$(CFORM) \
		--parameters=file://$(PARAM_FILE) \
		--output text --query 'Url'

package:

upload: upload-package upload-template

upload-package:

upload-template:

create: package upload
	@aws cloudformation create-stack --stack-name=$(STACK_NAME) \
		--template-body=file://$(CFORM) --capabilities=CAPABILITY_IAM \
		--parameters=file://$(PARAM_FILE) \

update:
	@aws cloudformation update-stack --stack-name=$(STACK_NAME) \
		--template-body=file://$(CFORM) --capabilities=CAPABILITY_IAM \
		--parameters=file://$(PARAM_FILE) \

validate:
	@aws cloudformation validate-template --template-body=file://$(CFORM)

delete: delete-buckets # remove after dev
	@aws cloudformation delete-stack --stack-name=$(STACK_NAME) \
	&& aws cloudformation wait stack-delete-complete --stack-name=$(STACK_NAME) \

delete-force: delete-buckets delete
	@aws cloudformation delete-stack --stack-name=$(STACK_NAME)

delete-buckets:
	@aws s3 rb --force s3://$(LOG_BUCKET) 2>/dev/null || true

describe:
	@aws cloudformation describe-stacks --stack-name=$(STACK_NAME)

events:
	@aws cloudformation describe-stack-events --stack-name=$(STACK_NAME)

resources:
	@aws cloudformation describe-stack-resources --stack-name=$(STACK_NAME)

clean:
	@rm -f *.zip
