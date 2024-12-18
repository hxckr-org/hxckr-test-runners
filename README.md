The Test Runner Service is a service designed to execute tests for various programming languages in isolated environments. It integrates with RabbitMQ for message queuing, uses Docker for consistent and isolated execution environments, and supports multiple programming languages.

## Table of Contents

1. [Features](#features)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Usage](#usage)
6. [Architecture](#architecture)
7. [Adding Support for New Languages](#adding-support-for-new-languages)
8. [Troubleshooting](#troubleshooting)
9. [Contributing](#contributing)
10. [License](#license)
11. [Sample Test Report](#sample-test-report)

## Features

- Support for multiple programming languages (currently TypeScript and Rust, with easy extensibility)
- RabbitMQ integration for distributed test execution
- Isolated test environments using Docker
- Git repository cloning and management
- Detailed, structured logging of the test process
- Modular and extensible architecture
- Kubernetes-friendly setup (preparation for K8s deployment)
- Pre-built base Docker images for supported languages

## Prerequisites

- Node.js (v20 or later)
- npm or yarn
- Docker
- RabbitMQ server
- Git

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/your-org/test-runner-service.git
   cd test-runner-service
   ```

2. Install dependencies:
   ```
   yarn
   ```

3. Set up environment variables (see [Configuration](#configuration) section)

4. Build the project:
   ```
   yarn build
   ```

5. Build base Docker images:
   ```
   docker build -t ts-base -f baseImages/typescript/Dockerfile .
   docker build -t rs-base -f baseImages/rust/Dockerfile .
   ```

## Configuration

Create a `.env` file in the root directory with the following variables:

```
PORT=3001
RABBITMQ_HOST=
RABBITMQ_PORT=
RABBITMQ_USERNAME=
RABBITMQ_PASSWORD=
TEST_REPO_URL=https://github.com/your-org/test-repo.git
TEST_REPO_BRANCH=main
TEST_REPO_PATH=tests
```

Adjust these values according to your environment.

## Directory Structure

Tests should be organized in the test repository as follows (first test will usually run on stage 2):
```
tests/
  challenge-id-1/
    typescript/
      stage1.test.ts
      stage2.test.ts (if any)
    rust/
      stage1.test.rs
  challenge-id-2/
    ...
```

## Usage

1. Start the service:
   ```
   yarn dev
   ```

2. The service will start listening for test run requests on the RabbitMQ queue defined in `config/rabbitmq.ts`.

3. To submit a test run request, publish a message to the `test_runner_queue` with the following format:
   ```json
   {
     "event_type": "push",
     "repoUrl": "https://github.com/user/repo.git",
     "branch": "main",
     "commitSha": "abcdef123456"
   }
   ```

4. The service will clone the repository, build a Docker image using the pre-built base image, execute the tests/program in an isolated container, and publish the results to the `test_results_queue`.

5. Test results will include detailed logs of the entire process.

## Sample Test Report

Here's an example of a test report generated by the Test Runner Service:

```json
result: {
  "commitSha": "abcdef123456",
  "event_type": "test",
  "success": true | false,
  "output": ""
  }
```

## Architecture

The Test Runner Service uses a modular architecture:

- RabbitMQ for message queuing
- Docker for isolated test environments
- Pre-built base Docker images for each supported language

## Adding Support for New Languages

To add support for a new programming language:

1. Add a new configuration in `config/config.ts` under `LANGUAGE_CONFIGS`.
2. Create a new Dockerfile in `baseImages/` for the new language.
3. Build the new base Docker image.
4. Update the `detectLanguage` function in `config/config.ts` to detect the new language.
5. If necessary, add language-specific logic in `utils/runner.ts`.

## Troubleshooting

- Check Docker is running and the service has permissions to use it.
- Verify RabbitMQ connection if the service isn't receiving messages.
- Ensure the necessary base Docker images are built and available.
- Check the logs for detailed information about any failures in the test process.

## Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.
