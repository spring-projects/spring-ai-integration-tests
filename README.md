# spring-ai-integration-tests

This repository contains workflows that define the integration test automation for the Spring AI project.

## Adding Secrets

All environment variables are stored in GitHub secrets.

To add or change a secret:

1. Open [spring-projects/spring-ai-integration-tests](https://github.com/spring-projects/spring-ai-integration-tests) in your browser.
2. If you have permissions, you will see "Settings" in the toolbar on that page.
3. Navigate to **Settings** -> **Security** -> **Secrets and variables** -> **Actions**.
4. Click **New repository secret** to add a new environment variable for the integration test pipeline.

## Workflow Schedule

The integration tests run automatically:
- At 4:00 UTC on weekdays (Monday through Friday)
- At 10:00 UTC on weekdays (Monday through Friday)

The workflow can also be triggered manually using the workflow_dispatch event.

## Job Types

### Basic Integration Test Job

Example of a basic integration test job for testing AI models:

```yaml
test-mistral-ai:
  runs-on: ubuntu-latest
  env:
    MISTRAL_AI_API_KEY: ${{ secrets.MISTRAL_AI_API_KEY }}
  steps:
    - name: Check secrets
      id: secret_check
      if: ${{ env.MISTRAL_AI_API_KEY != '' }}
      run: echo "Secrets exist"

    - name: Checkout the action
      uses: actions/checkout@v4

    - name: Integration Test
      if: steps.secret_check.conclusion == 'success'
      uses: ./.github/actions/do-integration-test
      with:
        model-name: mistral-ai
```

The standard steps:
- Define required environment variables using GitHub secrets
- Check if required secrets exist and have values
- Checkout the repository
- Run the integration test only if all secrets are available

The `do-integration-test` action handles:
- Checking environment variables
- Checking out the Spring AI source code
- Setting up JDK 17 with Maven caching
- Installing the Spring AI code without running tests
- Executing the integration tests with retry capability for failing tests

The only customization needed is the model name. The action assumes `models/spring-ai-` prefix, so for the above example with model-name `mistral-ai`, `models/spring-ai-mistral-ai` will be integration tested.

### Multi-Module Test Job

For testing multiple modules in a single job:

```yaml
test-vectorstores:
  runs-on: ubuntu-latest
  env:
    DOCKER_QUIET: 1              # Suppresses Docker CLI progress output
    TESTCONTAINERS_QUIET: true   # Additional quieting for testcontainers
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    MISTRAL_AI_API_KEY: ${{ secrets.MISTRAL_AI_API_KEY }}
    OLLAMA_TESTS_ENABLED: true
  steps:
    - name: Check secrets
      id: secret_check
      if: ${{ env.OPENAI_API_KEY != '' && env.MISTRAL_AI_API_KEY != '' }}
      run: echo "Secrets exist"

    - uses: actions/checkout@v4

    - name: Configure Testcontainers
      run: |
        mkdir -p $HOME
        echo "testcontainers.reuse.enable = true" >> $HOME/.testcontainers.properties

    - name: Integration Test
      uses: ./.github/actions/do-multi-module-test
      with:
        modules: vector-stores/spring-ai-module1,vector-stores/spring-ai-module2
```

The `do-multi-module-test` action is similar to `do-integration-test` but allows testing multiple modules in a single job.

### Autoconfiguration Test

The autoconfiguration test job runs for Spring Boot autoconfiguration testing. Environment variables in the `env` section control which tests are enabled:

```yaml
test-autoconfigure:
  env:
    AZURE_OPENAI_API_KEY: ${{ secrets.AZURE_OPENAI_API_KEY }}
    AZURE_OPENAI_ENDPOINT: ${{ secrets.AZURE_OPENAI_ENDPOINT }}
    # Additional keys can be uncommented when available
  runs-on: ubuntu-latest
```

## Required Environment Variables

The following environment variables are needed for various tests:

### Vector Store Tests
```
OPENAI_API_KEY
MISTRAL_AI_API_KEY
OLLAMA_TESTS_ENABLED
```

### AI Model Tests
```
AZURE_OPENAI_API_KEY
AZURE_OPENAI_ENDPOINT
AZURE_OPENAI_TRANSCRIPTION_API_KEY
AZURE_OPENAI_TRANSCRIPTION_ENDPOINT
AZURE_OPENAI_IMAGE_API_KEY
AZURE_OPENAI_IMAGE_ENDPOINT
MISTRAL_AI_API_KEY
```

## Docker and TestContainers Configuration

For jobs using Docker or TestContainers, the following environment variables are used to reduce log verbosity:
```yaml
env:
  DOCKER_QUIET: 1              # Suppresses Docker CLI progress output
  TESTCONTAINERS_QUIET: true   # Additional quieting for testcontainers
```

TestContainers reuse is enabled with:
```bash
echo "testcontainers.reuse.enable = true" >> $HOME/.testcontainers.properties
```

## Additional Tests

The workflow also includes tests for:
- Spring AI Integration Tests
- Docker Compose Support
- TestContainers Support
- Spring Cloud Bindings

Each of these tests uses the multi-module test action with appropriate module paths.