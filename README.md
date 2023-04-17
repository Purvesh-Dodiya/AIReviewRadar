# AI-powered radar that scans and detects issues in code during PR reviews, providing a comprehensive and vigilant review process.

## Overview

AIReviewRadar is a GitHub Action-based tool that uses AI capabilities to assist with Pull Request (PR) reviews. It offers the following features:

1.  Automated code analysis: AIReviewRadar uses Open AI API to analyze code and provides suggestions for code refactoring, helping to improve code quality.
3. Violation notifications: If code violates the set coding standards, AIReviewRadar adds comments to the PR, guiding developers on how to improve their code.
4. Streamlined PR review process: By automating parts of the review process, AIReviewRadar reduces the time and effort required for PR reviews.
5. Objective and consistent review process: AIReviewRadar ensures an objective and consistent review process, reducing subjectivity and inconsistencies.
6. Improved code quality: With AI-powered code analysis and suggestions, AIReviewRadar helps developers improve their code, resulting in better overall code quality.
 

With its AI capabilities and customizable features, AIReviewRadar helps organizations achieve faster PR reviews, maintain coding standards, and improve the quality of their code.

NOTES:

- Your code (files, diff, PR title/description) will be sent to OpenAI's servers
  for processing. Please check with your compliance team before using this on
  your private code repositories.
- OpenAI's API is used instead of ChatGPT session on their portal. OpenAI API
  has a
  [more conservative data usage policy](https://openai.com/policies/api-data-usage-policies)
  compared to their ChatGPT offering.

## Usage

Add the below file to your repository at
`.github/workflows/openai-pr-reviewer.yml`

```yaml
name: AIReviewRadar

permissions:
  contents: read
  pull-requests: write

on:
  pull_request:
  pull_request_review_comment:
    types: [created]

concurrency:
  group:
    ${{ github.repository }}-${{ github.event.number || github.head_ref ||
    github.sha }}-${{ github.workflow }}-${{ github.event_name ==
    'pull_request_review_comment' && 'pr_comment' || 'pr' }}
  cancel-in-progress: ${{ github.event_name != 'pull_request_review_comment' }}

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: purvesh-d/AIReviewRadar@1.0.2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        with:
          debug: false
          review_comment_lgtm: false
```

### Conversation with OpenAI

You can reply to a review comment made by this action and get a response based
on the diff context. Additionally, you can invite the bot to a conversation by
tagging it in the comment (`@openai`).

Examples:

> @openai Can you please review this block of code?

> @openai Please generate a test plan for this file.

Note: A review comment is a comment made on a diff or a file in the pull
request.

#### Environment variables

- `GITHUB_TOKEN`: This should already be available to the GitHub Action
  environment. This is used to add comments to the pull request.
- `OPENAI_API_KEY`: use this to authenticate with OpenAI API. You can get one
  [here](https://platform.openai.com/account/api-keys). Please add this key to
  your GitHub Action secrets.

### Prompts & Configuration

See: [action.yml](./action.yml)

Any suggestions or pull requests for improving the prompts are highly
appreciated.

## FAQs

### Review pull request from forks

GitHub Actions limits the access of secrets from forked repositories. To enable
this feature, you need to use the `pull_request_target` event instead of
`pull_request` in your workflow file. Note that with `pull_request_target`, you
need extra configuration to ensure checking out the right commit:

```yaml
name: AIReviewRadar

permissions:
  contents: read
  pull-requests: write

on:
  pull_request_target:
    types: [opened, synchronize, reopened]
  pull_request_review_comment:
    types: [created]

concurrency:
  group:
    ${{ github.repository }}-${{ github.event.number || github.head_ref ||
    github.sha }}-${{ github.workflow }}-${{ github.event_name ==
    'pull_request_review_comment' && 'pr_comment' || 'pr' }}
  cancel-in-progress: ${{ github.event_name != 'pull_request_review_comment' }}

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: purvesh-d/AIReviewRadar@1.0.2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        with:
          debug: false
          review_comment_lgtm: false
```

### Inspect the messages between OpenAI server

Set `debug: true` in the workflow file to enable debug mode, which will show the
messages.
