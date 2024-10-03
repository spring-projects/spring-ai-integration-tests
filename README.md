# spring-ai-integration-tests

This repository contains workflows that define the integration test automation for the Spring AI project.

## Adding secrets
All environment variables are stored in Github secrets.

To add or change a secret:

open https://github.com/spring-projects/spring-ai-integration-tests in your browser.

if you have permissions you will see "Setting" in the toolbar on that page

Setting -> Security :: Secrets and variables -> Actions

Click 'New repository secret' to add another environment variable for the integration test pipeline.

## Basic Integration Test Job

Job name and runs in a ubuntu container:
```yaml
  test-anthropic:
    runs-on: ubuntu-latest
```

Environment variables needed by the Integration tests:
```yaml
    env:
        ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

The standard steps to:
- check for the environment variables in the `env:` section, if any environment variables are missing or do not have a value the job will be failed
- checkout che springAI source
- install java 17
- compile the springAI code without running tests
- execute the integration test are all in a custom action

```yaml
    steps:
      - name: Checkout the action
        uses: actions/checkout@v4

      - name: Integration Test
        uses: ./.github/actions/do-integration-test
        with:
          model-name: anthropic
```

The only customization needed is the model name... custom action assumes `models/spring-ai-` so for the above example the model-name is `anthropic` and `models/spring-ai-anthropic` will be integration tested.
## Autoconfiguration Test
The autoconfiguration test runs for any of the properties in the `env` section
and when they have values the environment variable needs to be uncommented to be
included in the autoconfiguraton test

```yaml
  test-autoconfigure:
    env:
      # TODO: uncomment keys that have values
#      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
#      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AZURE_OPENAI_API_KEY: ${{ secrets.AZURE_OPENAI_API_KEY }}
      AZURE_OPENAI_ENDPOINT: ${{ secrets.AZURE_OPENAI_ENDPOINT }}
#      AZURE_OPENAI_TRANSCRIPTION_DEPLOYMENT_NAME: ${{ secrets.AZURE_OPENAI_TRANSCRIPTION_DEPLOYMENT_NAME }}
#      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
#      HUGGINGFACE_API_KEY: ${{ secrets.HUGGINGFACE_API_KEY }}
#      HUGGINGFACE_CHAT_URL: ${{ secrets.HUGGINGFACE_CHAT_URL }}
#      MISTRAL_AI_API_KEY: ${{ secrets.MISTRAL_AI_API_KEY }}
#      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
#      STABILITYAI_API_KEY: ${{ secrets.STABILITYAI_API_KEY }}
    runs-on: ubuntu-latest
```