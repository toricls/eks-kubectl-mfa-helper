# Amazon EKS kubectl MFA helper

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)][license]

[license]: https://github.com/toricls/eks-kubectl-mfa-helper/blob/master/LICENSE
Better friend for your MFA-forced IAM user and kubectl command.

It retrieves and caches your temporary session tokens to pass it to `aws-iam-authenticator` or `aws eks get-token` command.

## Usage

**NOTE: The `mfa-kubectl-helper` script may run only on OSX. Any contribution is appreciated :)**

1. Put the `mfa-kubectl-helper` to your $PATH
2. Let's chmod +x to the `mfa-kubectl-helper`
3. Rewrite your KubeConfig to use `mfa-kubectl-helper` instead of `aws-iam-authenticator` or `aws` commands
4. Make sure you add `AWS_MFA_SERIAL` environment variables. (and `AWS_DEFAULT_REGION` if you configured your aws cli without default region)

The final KubeConfig may something like the followings:

```yaml
# If you're using aws-iam-authenticator
~snip~
users:
- name: arn:aws:eks:us-west-2:${YOUR-ACCOUNT-ID}:cluster/${YOUR-CLUSTER-NAME}
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - token
      - -i
      - ${YOUR-CLUSTER-NAME}
      command: mfa-kubectl-helper
      env:
      - name: AWS_PROFILE
        value: ${YOUR_AWS_PROFILE_NAME}
      - name: AWS_MFA_SERIAL
        value: ${YOUR_AWS_MFA_SERIAL}
      - name: AWS_DEFAULT_REGION
        value: ${YOUR_AWS_DEFAULT_REGION}
~snip~
```

```yaml
# If you're using the EKS GetToken API
~snip~
users:
- name: arn:aws:eks:us-west-2:${YOUR-ACCOUNT-ID}:cluster/${YOUR-CLUSTER-NAME}
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - eks
      - get-token
      - --cluster-name
      - ${YOUR-CLUSTER-NAME}
      command: mfa-kubectl-helper
      env:
      - name: AWS_PROFILE
        value: ${YOUR_AWS_PROFILE_NAME}
      - name: AWS_MFA_SERIAL
        value: ${YOUR_AWS_MFA_SERIAL}
      - name: AWS_DEFAULT_REGION
        value: ${YOUR_AWS_DEFAULT_REGION}
~snip~
```

## FAQ

### I'm getting `Error: unknown command "eks" for "aws-iam-authenticator"`

The `command` parameter in your KubeConfig file (~/.kube/config by default) may `aws-iam-authenticator` but you're trying to use the EKS's GetToken API, so rewrite the command parameter as `aws` instead of `aws-iam-authenticator`.

You need to comment out line.35 and comment in the line.38 in the mfa-kubectl-helper file as well.

### I'm getting `You must specify a region. You can also configure your region by running "aws configure".`

Add something like the following to your KubeConfig file.

```yaml
~snip~
      env:
      ~snip~
      - name: AWS_DEFAULT_REGION
        value: us-west-2
~snip~
```

### I'm getting something like `line 23: AWS_MFA_SERIAL: unbound variable`

Add something like the following to your KubeConfig file.

```yaml
~snip~
      env:
      ~snip~
      - name: AWS_MFA_SERIAL
        value: put your MFA serial here
~snip~
```

## Contribution

1. Fork ([https://github.com/toricls/eks-kubectl-mfa-helper/fork](https://github.com/toricls/eks-kubectl-mfa-helper/fork))
1. Create a feature branch
1. Commit your changes
1. Rebase your local changes against the master branch
1. Create a new Pull Request

## Licence

[MIT](LICENSE)

## Author

[toricls](https://github.com/toricls)
