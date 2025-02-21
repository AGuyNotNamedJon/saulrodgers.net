+++
title = "Azure Noob Part 1 - Getting things started"
date = "2025-02-20"
draft = "true"
type = "post"
tags = [ "Azure", "Hugo", "Noob", "Blog" ]
description = "How I am learning the basics of Azure and it's services with-in."
categories = [ "Azure", "Blog" ]
series = [ "Azure Noob" ]
+++
## Getting Started with Building a Hugo Website on Azure

In this guide, I'll walk through the steps to set up your environment for building a Hugo website using Azure. We'll cover installing necessary tools, setting up Azure resources, and creating your first Hugo site.

### Prerequisites

Before we begin, ensure you have the following:
- An Azure account
- Administrative access to your machine

### Step 1: Installing VS Code with Azure Extensions

Visual Studio Code (VS Code) is a powerful, lightweight code editor. To install VS Code:

1. Download and install VS Code from [here](https://code.visualstudio.com/).
2. Open VS Code and go to the Extensions view by clicking the Extensions icon in the Activity Bar on the side of the window.
3. Search for and install the following extensions:
    - Azure Account
    - Azure CLI Tools
    - Azure Functions
    - Azure Resource Manager (ARM) Tools

### Step 2: Installing the Azure CLI

The Azure Command-Line Interface (CLI) is a set of commands used to create and manage Azure resources. To install the Azure CLI:

1. Follow the instructions for your operating system [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
2. Verify the installation by running `az --version` in your terminal.

### Step 3: Installing Azure Functions Core Tools

Azure Functions Core Tools lets you develop and test your functions on your local computer. To install:

1. Follow the instructions for your operating system [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local).
2. Verify the installation by running `func --version` in your terminal.

### Step 4: Creating Management Groups in Azure

Management Groups help you organize your resources in Azure. We'll create two groups: Development and Production.

1. Open the Azure Portal.
2. Navigate to "Management Groups" and click "Add Management Group".
3. Create a group named "Development".
4. Repeat the process to create a group named "Production".

### Step 5: Setting Up Budgets in Azure

Setting budgets helps you manage and control your spending. To set up budgets:

1. In the Azure Portal, navigate to "Cost Management + Billing".
2. Select "Budgets" and click "Add".
3. Create a budget for the "Development" group.
4. Repeat the process to create a budget for the "Production" group.

### Step 6: Installing Hugo

Hugo is a fast and flexible static site generator. To install Hugo:

1. Open your terminal.
2. Run the following command to install Hugo using webget:
    ```sh
    webget https://github.com/gohugoio/hugo/releases/download/v0.88.1/hugo_extended_0.88.1_Windows-64bit.zip
    ```

### Step 7: Creating a New Hugo Site with a Theme

Now, let's create your first Hugo site:

1. Open your terminal and navigate to the directory where you want to create your site.
2. Run the following command to create a new site:
    ```sh
    hugo new site my-hugo-site
    ```
3. Navigate to the new site directory:
    ```sh
    cd my-hugo-site
    ```
4. Add a theme to your site. For example, to add the "Ananke" theme:
    ```sh
    git init
    git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
    echo 'theme = "ananke"' >> config.toml
    ```

### Conclusion

You've now set up your environment for building a Hugo website using Azure. You have installed the necessary tools, created management groups, set budgets, and created your first Hugo site. Happy coding!