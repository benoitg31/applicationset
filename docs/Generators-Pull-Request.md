# Pull Request Generator

The Pull Request generator uses the API of an SCMaaS provider (eg GitHub/GitLab) to automatically discover open pull requests within an repository. This fits well with the style of building a test environment when you create a pull request.


```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapps
spec:
  generators:
  - pullRequest:
      # See below for provider specific options.
      github:
        # ...
```

## GitHub

Specify the repository from which to fetch the Github Pull requests.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapps
spec:
  generators:
  - pullRequest:
      github:
        # The GitHub organization or user.
        owner: myorg
        # The Github repository
        repo: myrepository
        # For GitHub Enterprise (optional)
        api: https://git.example.com/
        # Reference to a Secret containing an access token. (optional)
        tokenRef:
          secretName: github-token
          key: token
        # Labels is used to filter the PRs that you want to target. (optional)
        labels:
        - preview
  requeueAfterSeconds: 1800
  template:
  # ...
```

* `owner`: Required name of the GitHub organization or user.
* `repo`: Required name of the Github repositry.
* `api`: If using GitHub Enterprise, the URL to access it. (Optional)
* `tokenRef`: A `Secret` name and key containing the GitHub access token to use for requests. If not specified, will make anonymous requests which have a lower rate limit and can only see public repositories. (Optional)
* `labels`: Labels is used to filter the PRs that you want to target. (Optional)

## Template

As with all generators, several keys are available for replacement in the generated application.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapps
spec:
  generators:
  - pullRequest:
    # ...
  template:
    metadata:
      name: 'myapp-{{ branch }}-{{ number }}'
    spec:
      source:
        repoURL: 'https://github.com/myorg/myrepo.git'
        targetRevision: '{{ head_sha }}'
        path: kubernetes/
        helm:
          parameters:
          - name: "image.tag"
            value: "pull-{{ head_sha }}"
      project: default
      destination:
        server: https://kubernetes.default.svc
        namespace: default
```

* `number`: The ID number of the pull request.
* `branch`: The name of the branch of the pull request head.
* `head_sha`: This is the SHA of the head of the pull request.

## Webhook Configuration

When using a Pull Request generator, ApplicationSet polls every requeueAfterSeconds(default: 30 minutes) interval to detect changes. To eliminate this delay from polling, the ApplicationSet webhook server can be configured to receive webhook events. ApplicationSet supports PullRequest webhook notifications from GitHub.

The configuration is almost the same as the one described in the Git Generator, but there is one difference if you want to use the Pull Request Generator as well, so if you want to use it, additionally configure the following settings.

In section 1, add an event so that a webhook request will be sent when a pull request is created, closed, or label changed.
Select `Let me select individual events.` and enable the checkbox for `Pull requests`.

![Add Webhook](./assets/webhook-config-pull-request.png "Add Webhook Pull Request")

The Pull Request Generator will requeue when the next action occurs.

- `opened`
- `closed`
- `reopened`
- `labeled`
- `unlabeled`
- `synchronized`

For more information about each event, please refer to the official documentation at github.com.
