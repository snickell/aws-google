# Aws::Google [![Build Status](https://travis-ci.com/code-dot-org/aws-google.svg?branch=master)](https://travis-ci.com/code-dot-org/aws-google)

Use Google OAuth as an AWS Credential Provider.

## Installation

Add this line to your application's `Gemfile`:

```ruby
gem 'aws-google'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install aws-google

## Usage

- Visit the [Google API Console](https://console.developers.google.com/) to create/obtain OAuth 2.0 Client ID credentials (client ID and client secret) for an application in your Google account.
- Create an AWS IAM Role with the desired IAM policies attached, and a ['trust policy'](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#term_trust-policy) ([`AssumeRolePolicyDocument`](https://docs.aws.amazon.com/IAM/latest/APIReference/API_CreateRole.html)) allowing the [`sts:AssumeRoleWithWebIdentity`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html) action with [Web Identity Federation condition keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_iam-condition-keys.html#condition-keys-wif) authorizing
your Google Client ID (`accounts.google.com:aud`) and a specific set of Google Account IDs (`accounts.google.com:sub`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "accounts.google.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "accounts.google.com:aud": "123456789012-abcdefghijklmnopqrstuvwzyz0123456.apps.googleusercontent.com",
          "accounts.google.com:sub": [
            "000000000000000000000",
            "111111111111111111111"
          ]
        }
      }
    }
  ]
}
```

- In your Ruby code, construct an `Aws::Google` object by passing in the AWS role, client id and client secret:
```ruby
require 'aws/google'

aws_role = 'arn:aws:iam::[AccountID]:role/[Role]'
client_id = '123456789012-abcdefghijklmnopqrstuvwzyz0123456.apps.googleusercontent.com'
client_secret = '01234567890abcdefghijklmn'

role_credentials = Aws::Google.new(
  role_arn: aws_role,
  google_client_id: client_id,
  google_client_secret: client_secret
)

puts Aws::STS::Client.new(credentials: role_credentials).get_caller_identity
```

- Or, set `Aws::Google.config` hash to add Google auth to the default credential provider chain:

```ruby
Aws::Google.config = {
  role_arn: aws_role,
  google_client_id: client_id,
  google_client_secret: client_secret,
}

puts Aws::STS::Client.new.get_caller_identity
```

- Or, set `credential_process` in your AWS config profile ([`~/.aws/config`](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-configure-files-where)) to `aws-google` to [Source Credentials with an External Process](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sourcing-external.html) without any change to your application code:

```
[profile my_google]
credential_process = aws-google --profile my_google
aws_role = arn:aws:iam::[AccountID]:role/[Role]
google_client_id = 123456789012-abcdefghijklmnopqrstuvwzyz0123456.apps.googleusercontent.com
google_client_secret = 01234567890abcdefghijklmn

```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake test` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/code-dot-org/aws-google.

## License

The gem is available as open source under the terms of the [Apache 2.0 License](http://opensource.org/licenses/apache-2.0).
