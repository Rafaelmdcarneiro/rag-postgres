# RAG on PostgreSQL
This project creates a web-based chat application with an API backend that can use OpenAI chat models to answer questions about the items in a PostgreSQL database table.

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](placeholder)
[![Open in Dev Containers](https://img.shields.io/static/v1?style=for-the-badge&label=Dev%20Containers&message=Open&color=blue&logo=visualstudiocode)](placeholder)

This project creates a web-based chat application with an API backend that can use OpenAI chat models to answer questions about the items in a PostgreSQL database table. The frontend is built with React and FluentUI, while the backend is written with Python and FastAPI.

This project is designed for deployment to Azure using [the Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/), hosting the app on Azure Container Apps, the database in Azure PostgreSQL Flexible Server, and the models in Azure OpenAI.

* [Features](#features)
* [Opening the project](#opening-the-project)
* [Deployment](#deployment)
* [Local development](#local-development)
* [Costs](#costs)
* [Security guidelines](#security-guidelines)
* [Resources](#resources)

## Features

This project provides the following features:

* Hybrid search on the PostgreSQL database table, using [the pgvector extension](https://github.com/pgvector/pgvector) for the vector search plus [full text search](https://www.postgresql.org/docs/current/textsearch-intro.html), combining the results using RRF (Reciprocal Rank Fusion).
* OpenAI function calling to optionally convert user queries into query filter conditions, such as turning "Climbing gear cheaper than $30?" into "WHERE price < 30".
* Conversion of user queries into vectors using the OpenAI embedding API.

![Screenshot of chat app with question about climbing gear](docs/screenshot_chat.png)

## Opening the project

You have a few options for getting started with this template.
The quickest way to get started is GitHub Codespaces, since it will setup all the tools for you, but you can also [set it up locally](#local-environment).

### GitHub Codespaces

You can run this template virtually by using GitHub Codespaces. The button will open a web-based VS Code instance in your browser:

1. Open the template (this may take several minutes):

    [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](placeholder)

2. Open a terminal window
3. Sign in to your Azure account:

    ```shell
    azd auth login --use-device-code
    ```

4. Provision the resources and deploy the code:

    ```shell
    azd up
    ```

    This project uses gpt-3.5-turbo and text-embedding-ada-002 which may not be available in all Azure regions. Check for [up-to-date region availability](https://learn.microsoft.com/azure/ai-services/openai/concepts/models#standard-deployment-model-availability) and select a region during deployment accordingly.

### VS Code Dev Containers

A related option is VS Code Dev Containers, which will open the project in your local VS Code using the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers):

1. Start Docker Desktop (install it if not already installed)
2. Open the project:

    [![Open in Dev Containers](https://img.shields.io/static/v1?style=for-the-badge&label=Dev%20Containers&message=Open&color=blue&logo=visualstudiocode)](placeholder)

3. In the VS Code window that opens, once the project files show up (this may take several minutes), open a terminal window.

### Local Environment

#### Prerequisites

* [Azure Developer CLI (azd)](https://aka.ms/install-azd)
* [Python 3.10+](https://www.python.org/downloads/)
* [PostgreSQL 14+](https://www.postgresql.org/download/)
* [pgvector](https://github.com/pgvector/pgvector)
* [Docker Desktop](https://www.docker.com/products/docker-desktop/)
* [Git](https://git-scm.com/downloads)

#### Installation

Install required packages:

```shell
pip install -r requirements-dev.txt
```

## Deployment

Once you've opened the project in [Codespaces](#github-codespaces), [Dev Containers](#vs-code-dev-containers), or [locally](#local-environment), you can deploy it to Azure.

1. Sign in to your Azure account:

    ```shell
    azd auth login
    ```

    If you have any issues with that command, you may also want to try `azd auth login --use-device-code`.

2. Create a new azd environment:

    ```shell
    azd env new
    ```

    This will create a folder under `.azure/` in your project to store the configuration for this deployment. You may have multiple azd environments if desired.

3. Provision the resources and deploy the code:

    ```shell
    azd up
    ```

    This project uses gpt-3.5-turbo and text-embedding-ada-002 which may not be available in all Azure regions. Check for [up-to-date region availability](https://learn.microsoft.com/azure/ai-services/openai/concepts/models#standard-deployment-model-availability) and select a region during deployment accordingly.

## Local Development

### Setting up the environment file

Since the local app uses OpenAI models, you should first deploy it for the optimal experience.

1. Copy `.env.sample` into a `.env` file.
2. To use Azure OpenAI, fill in the values of `AZURE_OPENAI_ENDPOINT` and `AZURE_OPENAI_CHAT_DEPLOYMENT` based on the deployed values. You can display the values using this command:

    ```shell
    azd env get-values
    ```

3. To use OpenAI.com OpenAI, fill in the value for `OPENAICOM_KEY`.
4. To use Ollama, update the values for `OLLAMA_ENDPOINT` and `OLLAMA_CHAT_MODEL` to match your local setup and model. Note that you won't be able to use function calling or hybrid search with an Ollama model.

### Running the frontend and backend

1. Run these commands to install the web app as a local package (named `fastapi_app`), set up the local database, and seed it with test data:

    ```bash
    python3 -m pip install -e src
    python ./src/fastapi_app/setup_postgres_database.py
    python ./src/fastapi_app/setup_postgres_seeddata.p
    ```

    If you opened the project in Codespaces or a Dev Container, these commands will already have been run for you.

2. Run the FastAPI backend:

    ```shell
    fastapi dev src/api/main.py
    ```

    Or you can run "Backend" in the VS Code Run & Debug menu.

3. Run the frontend:

    ```bash
    cd src/frontend
    npm run dev
    ```

    Or you can run "Frontend" or "Frontend & Backend" in the VS Code Run & Debug menu.

4. Open the browser at `http://localhost:5173/` and you will see the frontend.

## Costs

Pricing may vary per region and usage. Exact costs cannot be estimated.
You may try the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) for the resources below:

* Azure Container Apps: Pay-as-you-go tier. Costs based on vCPU and memory used. [Pricing](https://azure.microsoft.com/pricing/details/container-apps/)
* Azure OpenAI: Standard tier, GPT and Ada models. Pricing per 1K tokens used, and at least 1K tokens are used per question. [Pricing](https://azure.microsoft.com/pricing/details/cognitive-services/openai-service/)
* Azure Monitor: Pay-as-you-go tier. Costs based on data ingested. [Pricing](https://azure.microsoft.com/pricing/details/monitor/)

## Security Guidelines

This template uses [Managed Identity](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview) for authenticating to the Azure services used (Azure OpenAI, Azure PostgreSQL Flexible Server).

Additionally, we have added a [GitHub Action](https://github.com/microsoft/security-devops-action) that scans the infrastructure-as-code files and generates a report containing any detected issues. To ensure continued best practices in your own repository, we recommend that anyone creating solutions based on our templates ensure that the [Github secret scanning](https://docs.github.com/code-security/secret-scanning/about-secret-scanning) setting is enabled.

## Resources

* [RAG chat with Azure AI Search + Python](https://github.com/Azure-Samples/azure-search-openai-demo/)
* [Develop Python apps that use Azure AI services](https://learn.microsoft.com/azure/developer/python/azure-ai-for-python-developers)
