author: Sarah Sdao
id: build-and-deploy-lm-dashboard-with-posit-team-and-cortex
summary: Build and deploy an interactive LLM-powered dashboard using the Posit Team Native App and Snowflake Cortex
categories: getting-started,data-science-&-ml,partner-integrations
environments: web
status: Published
feedback link: https://github.com/Snowflake-Labs/sfguides/issues
tags: Getting Started, Data Science, Posit, Python, LLM, AI
language: en

# Build and Deploy an Interactive LLM-Powered Dashboard with the Posit Team Native App and Snowflake Cortex AI

## Overview

In this guide, we'll use the Posit Team Native App to build an interactive dashboard that lets users explore Home Mortgage Disclosure Act (HMDA) data using natural language queries powered by Snowflake Cortex AI. We'll use Positron Assistant and Databot to do some quick, yet powerful exploratory data analysis and develop a Shiny application using the `querychat` and `chatlas` Python packages. Along the way, we'll deploy two data analysis products (a Quarto report and the interactive dashboard) to Posit Connect with one-click publishing for easy sharing across our organization.

By the end of this guide, we'll have a fully functional dashboard where users can ask questions like "What are the most common loan types?" or "How do loan approval rates vary by state?" and get instant visualizations and insights.

### What You Will Learn

- How to securely connect to your Snowflake databases from Posit Workbench and the Positron Pro IDE
- How to leverage Cortex AI using Databot and Positron Assistant to build a Quarto report and Shiny application
- How to create an LLM-powered chat interface for data exploration
- How to deploy the report and dashboard to Posit Connect with one-click publishing

### What You Will Build

- A Snowflake warehouse to query publicly available HMDA mortgage data
- An interactive Shiny dashboard with natural language query capabilities built using Cortex AI
- A published application accessible to your team on Posit Connect

### Prerequisites

- A [Snowflake account](https://signup.snowflake.com/?utm_source=snowflake-devrel&utm_medium=developer-guides&utm_cta=developer-guides) with Cortex AI enabled
- Appropriate access to create warehouses, databases, and schemas in Snowflake. This is typically the `sysadmin` role
- Access to the [Posit Team Native App](https://www.snowflake.com/en/developers/guides/analyze-data-with-python-using-posit-team/#:~:text=Pro%20from%20the-,Posit%20Team%20Native%20App,-.%20An%20administrator%20with). An administrator with the `accountadmin` role can provide these for you
- Familiarity with SQL and Python

## Setup

In this section, we'll create a warehouse in Snowflake and access a publicly available dataset that we'll query through our LLM dashboard.

### Create a Warehouse

For this analysis, we'll use the Home Mortgage Disclosure Act (HMDA) dataset from Snowflake's free public data. This dataset contains mortgage application and origination data collected under the Home Mortgage Disclosure Act, including information about loan types, applicant demographics, property characteristics, and loan outcomes across different geographic areas.

In Snowsight, open a SQL worksheet (**+** > **SQL File**). Then, paste in and run the following code, which creates a warehouse for our analysis. Make sure to change the role to your own role.

```sql
USE ROLE SYSADMIN; -- Replace with your actual Snowflake role (e.g., sysadmin)

CREATE OR REPLACE WAREHOUSE MORTGAGE_DATA_WH
    WAREHOUSE_SIZE = 'xsmall'
    WAREHOUSE_TYPE = 'standard'
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE
    INITIALLY_SUSPENDED = TRUE;
```

This command creates a small warehouse called `MORTGAGE_DATA_WH` that will be used to query the public dataset.

### Access the Public Dataset

The HMDA data is available in Snowflake's free public data collection, which is accessible to all Snowflake accounts without any additional setup. The dataset we'll use is located at:

- **Database:** `SNOWFLAKE_PUBLIC_DATA_FREE`
- **Schema:** `HOME_MORTGAGE_DISCLOSURE_ATTRIBUTES`
- **Table:** `HOME_MORTGAGE_DISCLOSURE`

To verify you have access to this data, run the following query in your SQL worksheet:

```sql
SELECT *
FROM SNOWFLAKE_PUBLIC_DATA_FREE.HOME_MORTGAGE_DISCLOSURE_ATTRIBUTES.HOME_MORTGAGE_DISCLOSURE
LIMIT 10;
```

You should see the first 10 rows of the HMDA dataset, which includes columns about mortgage applications, loan details, applicant information, and property characteristics.

## Launch Posit Workbench from the Posit Team Native App

We can now start exploring the data using Workbench. You can find Workbench within the Posit Team Native App, and use it to connect to your database.

### Step 1: Get the Posit Team Native App from the Snowflake Marketplace

- In Snowsight, click on **Marketplace**. If the Posit Team Native App is not already installed, search for "Posit Team" and then click **Get**.

  ![](assets/snowflake-get-posit-team.png)

- You might be asked to validate your email address.
- Choose a name for the App.

### Step 2: Open the Posit Team Native App from Snowsight

Please note that your administrator must first [install and configure](https://docs.posit.co/partnerships/snowflake/posit-team/) the Posit Team Native App--and Posit Workbench within it--before you can follow the remaining steps.

Once your administrator has installed and configured the Posit Team Native App, in Snowsight, navigate to **Horizon Catalog** > **Catalog** > **Installed Apps** > the Posit Team Native App. If you do not see the Posit Team Native App listed, ask your Snowflake account administrator for access to the app.

After clicking on the app, you will see the Posit Team Native App page.

Click **Launch app**.

![](assets/snowflake-launch-app.png)

### Step 3: Open Posit Workbench from the Posit Team Native App

From the Posit Team Native App, click **Posit Workbench**.

![](assets/snowflake-launch-workbench.png)

You might be prompted to first log in to Snowflake using your regular credentials or authentication method.

## Create a Positron Pro Session

Posit Workbench provides several IDEs, including Positron Pro, VS Code, RStudio Pro, and JupyterLab. For this analysis, we will use Positron,
the next-generation data science IDE built for Python and R. It combines the power of a full-featured IDE with interactive data science tools for Python and R.

### Step 1: New Session

Within Posit Workbench, click **+ New Session** to launch a new session.

![](assets/workbench-start-new-session.png)

### Step 2: Select an IDE

When prompted, select Positron Pro. You can optionally give your session a unique name.

![](assets/workbench-create-new-session.png)

#### Step 3: Log into your Snowflake account

Next, connect to your Snowflake account from within Posit Workbench.
Under **Session Credentials**, click the button with the Snowflake icon to sign in to Snowflake. Follow any sign in prompts.

![](assets/workbench-snowflake-login-success.png)

#### Step 4: Launch Positron Pro

Under **Environment**, enter at least 2.5 GB of RAM in the **Memory (GB)** field.

Then, click **Launch** to launch Positron Pro. If desired, you can check the **Auto-join session** option to automatically open the IDE when the session is ready.

![](assets/positron-launch.png)

You will now be able to work with your Snowflake data in Positron Pro. Since the IDE is provided
by Posit Workbench within the Posit Team Native App, your entire analysis will occur securely within Snowflake.

## Install the Necessary Extensions

### Get the Shiny Extension

The Shiny VS Code extension supports the development of Shiny apps in Positron. The Shiny Extension is included automatically in Positron as a [bootstrapped extension](https://positron.posit.co/extensions.html#bootstrapped-extensions).

We'll want Positron Assistant to be able to create a Shiny app to make and share our LLM dashboard, so first we need to make sure we have it installed and enabled:

1. Open the Positron Extensions view: on the right-hand side of Positron Pro, click the Extensions icon in the activity bar to open the Extensions Marketplace.

2. Search for "Shiny" to find the Shiny extension.

![](assets/positron-shiny-extension.png)

3. Verify that you have the Shiny extension:
  - If it is already installed and enabled, you will see a wheel icon.
  - If it is not already installed, click **Install**.
  - If you cannot install it yourself or you find that the extension is disabled, ask your administrator for access.

For more information, see the [Shiny extension documentation](https://open-vsx.org/extension/posit/shiny).

### Get and Enable the Databot Extension

> **Important:** Databot is currently in research preview and not ready for production use.

[Databot](https://positron.posit.co/databot.html) is an AI assistant designed to dramatically accelerate exploratory data analysis for data scientists fluent in Python or R, allowing them to do in minutes what might usually take hours. We'll use it below to both connect to our Snowflake data and conduct some exploratory data analysis (EDA).

1. Open the Positron Extensions view: on the right-hand side of Positron Pro, click the Extensions icon in the activity bar to open the Extensions Marketplace.

2. Search for "Databot".

3. Click **Install** to add the Databot extension.

4. After installation, you'll need to acknowledge the research preview status:
   - Open Settings (`Cmd/Ctrl+,`).
   - Search for "Databot".
   - In the **Databot: Research Preview Acknowledgement** field, type "Acknowledged".

## Access this Guide's Materials

This guide will walk you through the steps contained in <https://github.com/posit-dev/snowflake-posit-build-deploy-LLM-dashboard>. To follow along, clone the repository by following the steps below.

1. Open your home folder:

   - Press `Ctrl/Cmd+Shift+P` to open the Command Palette.
   - Type "File: Open Folder", and press `enter`.
   - Navigate to your home directory and click **OK**.

2. Clone the [GitHub repo](https://github.com/posit-dev/snowflake-posit-build-deploy-LLM-dashboard/) by running the following command in a terminal:

   ```bash
   git clone https://github.com/posit-dev/snowflake-posit-build-deploy-LLM-dashboard/
   ```

   > If you don't already see a terminal open, open the Command Palette (`Ctrl/Cmd+Shift+P`), then select **Terminal: Create New Terminal** to open one.

   > Note: This guide uses HTTPS for git authentication. Standard git authentication procedures apply.

3. Open the cloned repository folder:

   - Press `Ctrl/Cmd+Shift+P` to open the Command Palette.
   - Select **File: Open Folder**.
   - Navigate to `snowflake-posit-build-deploy-LLM-dashboard` and click **OK**.


## Explore Quarto

Before we dive into our data analysis, let's first discuss Quarto, since we've documented the initial steps to connect to our data in a Quarto (`.qmd`) document, [quarto.qmd](https://github.com/posit-dev/snowflake-posit-build-deploy-LLM-dashboard/blob/main/quarto.qmd), and we'll also create a Quarto document with exploratory data analysis in the [Databot](#explore-the-data-with-databot) section below.

[Quarto](https://quarto.org/)
is an open-source publishing system that makes it easy to create
[data products](https://quarto.org/docs/guide/) such as
[documents](https://quarto.org/docs/output-formats/html-basics.html),
[presentations](https://quarto.org/docs/presentations/),
[dashboards](https://quarto.org/docs/dashboards/),
[websites](https://quarto.org/docs/websites/),
and
[books](https://quarto.org/docs/books/). It is available out-of-the-box with Positron Pro and allows data scientists to interweave all of their code, results, output, and prose text into a single literate programming document. This way everything travels together as a reproducible data product.

A Quarto document can be thought of as a regular markdown document,
but with the ability to run code chunks.

You can run any of the code chunks by clicking the `Run Cell` button above the chunk in Positron Pro.

<!-- TODO - take new screenshot of run chunk
![](assets/quarto-run-chunk.png)
-->

To render and preview the entire document, click the `Preview` button
or run `quarto preview quarto.qmd` from the terminal.

![](assets/quarto-preview.png)

This will run all the code in the document from top to bottom and
generate an HTML file, by default, for you to view and share. This is especially helpful for creating multiple plots and other static content.

Learn more about Quarto here: <https://quarto.org/>,
and the documentation for all the various Quarto outputs here: <https://quarto.org/docs/guide/>.
Quarto works with Python, R, and Javascript Observable code out-of-the box, and is a great tool to communicate your data science analyses.

## Create a Virtual Environment

Before we can run any code, we need to set up our Python virtual environment.

1. Open the Command Palette (`Cmd/Ctrl+Shift+P`), then search for and select **Python: Create Environment**.

2. Choose `Venv` to create a `.venv` virtual environment.

3. Select the Python version you want to use.

4. When prompted to **Select dependencies to install**, choose the `requirements.txt` file from the cloned GitHub repo.

   - This creates the virtual environment with the necessary dependencies. Positron activates the virtual environment automatically.
   - If Positron does not activate the virtual environment automatically, open the terminal and run `source .venv/bin/activate`.

## Connect to Snowflake Data

Now that we have our Positron Pro session started with the necessary extensions and dependencies, we can connect to our data in Snowflake. There are two ways we can do this: automatically using Databot, or manually with some code.

### Use Databot to Connect

Instead of manually writing connection code, you can use Databot's built-in Snowflake skill to guide you through the connection process. This is
especially helpful when you're working in Posit Workbench, as Databot can automatically detect and use managed credentials.

In the Databot panel, simply ask:

```
Help me connect to Snowflake
```

Databot will:

1. Detect if you're in Posit Workbench with integrated authentication.

2. Provide the appropriate connection code for your environment.

3. Guide you through discovering available databases, schemas, and tables.

4. Help you explore Semantic Views if available.

If you're in Posit Workbench, Databot will provide zero-argument connection code that uses managed credentials:

```python
import snowflake.connector

# Uses Workbench managed credentials automatically
conn = snowflake.connector.connect(connection_name='workbench')
```

If you're not in Posit Workbench, Databot will prompt you for connection details and provide the appropriate connection code.

Once connected, you can move on to the next section, which is to [configure the `querychat` and `chatlas` packages to work with your Cortex AI-provided LLM](#configure-chatlas-and-querychat).

### Connect with Code

Open the `quarto.qmd` file from `snowflake-posit-build-deploy-LLM-dashboard`. You can now use the **Run Cell** button to run the Python code directly in the Quarto document.

The code below uses `ibis`, which lets you describe queries in Python. This system:

1. Keeps our data in the database, saving memory in the Python session.
2. Pushes computations to the database, saving compute in the Python session.
3. Evaluates queries lazily, saving compute in the database.

We don't need to manage the process, it happens automatically behind the scenes.

To ensure our connection works across different environments (development, Workbench, and Connect), the code also uses `snowflake.connector` and `posit.connect.external.snowflake` to create a flexible connection function:

- **Local development**: Uses explicit credentials
- **Workbench**: Leverages managed credentials via `connection_name="workbench"`
- **Connect**: Uses viewer credentials via `PositAuthenticator`, ensuring each user connects with their own Snowflake credentials, which respects Snowflake's Role-Based Access Control (RBAC)

```python
# Import necessary libraries
from posit.connect.external.snowflake import PositAuthenticator
import snowflake.connector
import ibis
import os

def get_connection():
    """Create a Snowflake connection appropriate for the current environment."""

    # Running in Posit Workbench
    if os.getenv("SNOWFLAKE_HOME") is not None and os.getenv("RSTUDIO_PRODUCT") != "CONNECT":
        con = snowflake.connector.connect(
            connection_name="workbench",
            warehouse="MORTGAGE_DATA_WH",
            database="SNOWFLAKE_PUBLIC_DATA_FREE",
            schema="HOME_MORTGAGE_DISCLOSURE_ATTRIBUTES"
        )

    # Running in Posit Connect (deployed app)
    elif os.getenv("RSTUDIO_PRODUCT") == "CONNECT":
        from shiny import session

        # Get the viewer's session token
        user_session_token = session.http_conn.headers.get("Posit-Connect-User-Session-Token")

        # Create authenticator with viewer's credentials
        auth = PositAuthenticator(
            local_authenticator="EXTERNALBROWSER",
            user_session_token=user_session_token
        )

        con = snowflake.connector.connect(
            account="YOUR_ACCOUNT",  # Replace with your Snowflake account
            warehouse="MORTGAGE_DATA_WH",
            database="SNOWFLAKE_PUBLIC_DATA_FREE",
            schema="HOME_MORTGAGE_DISCLOSURE_ATTRIBUTES",
            authenticator=auth.authenticator,
            token=auth.token,
        )

    else:
        raise ValueError("No Snowflake credentials found. Ensure you're running in Workbench or Connect.")

    return con

# Create connection
con = get_connection()

# Use Ibis for a Pythonic way to interact with Snowflake data
ibiscon = ibis.snowflake.from_connection(con, create_object_udfs=False)

mortgage_data = ibiscon.table("HOME_MORTGAGE_DISCLOSURE")

print("Successfully established secure connection to Snowflake!")
```

We have now used Workbench and Python to connect to the HMDA mortgage data in Snowflake's public datasets, all securely within Snowflake.

## Configure `chatlas` and `querychat`

When we activated the Python virtual environment before, we installed the packages `chatlas` and `querychat`. However, we still need to configure them to use a Cortex AI-provided LLM and our mortgage data.

### `chatlas`

The `chatlas` package provides a `ChatSnowflake` class that integrates with Snowflake Cortex AI. When using Posit Workbench within the Posit Team Native App, you can use the special `connection_name="workbench"` parameter to leverage Workbench's managed credentials:

```python
from chatlas import ChatSnowflake

# Initialize ChatSnowflake with Workbench managed credentials
chat = ChatSnowflake(
    system_prompt="You are a mortgage lending and housing finance data analysis expert",
    model="claude-haiku-4-5", #Choose from available Cortex AI models
    connection_name="workbench",
)

# Test the connection
response = chat.chat("What patterns do you see in home mortgage lending data?")
print(response)
```

When you run the cell, the response output will appear in the console.

### `querychat`

The `querychat` package creates interactive chat interfaces for data exploration. Building on the Ibis connection we established above, we can configure `querychat` to work with our mortgage data:

```python
from querychat import QueryChat

# Convert the Ibis table to a pandas DataFrame
# This loads the data into memory for fast interactive querying
df = mortgage_data.to_pandas()

# Initialize QueryChat with the mortgage data and Cortex AI
qc = QueryChat(
    data_source=df,
    table_name="HOME_MORTGAGE_DISCLOSURE",
    client="snowflake-cortex/claude-haiku-4-5",  # Use Snowflake Cortex AI
    greeting="""
    # Home Mortgage Disclosure Act (HMDA) Data Explorer

    Ask questions about mortgage lending patterns, loan characteristics, and geographic trends.

    **Example questions:**
    - What are the most common loan types?
    - How do loan approval rates vary by state?
    - Compare loan amounts across different property types
    """,
    data_description="""
    Home Mortgage Disclosure Act (HMDA) dataset containing mortgage application and origination data.
    Includes information about loan types, applicant demographics, property characteristics, loan amounts,
    interest rates, and loan outcomes. Data is tracked across U.S. geographic areas, allowing for
    analysis of lending patterns and trends.
    """,
    tools=("query", "update")  # Enable both SQL queries and dashboard filtering
)

# Create the Shiny app
app = qc.app()
```

**Configuration parameters:**

- `data_source`: The pandas DataFrame containing your data (loaded via Ibis from Snowflake)
- `table_name`: Identifier used in SQL queries (use your actual table name)
- `client`: LLM specification in `"provider/model"` format
  - For Snowflake Cortex AI, use `"snowflake-cortex/model-name"`
- `greeting`: Welcome message displayed to users (supports Markdown)
- `data_description`: Context about the dataset that helps the LLM generate accurate queries
- `tools`: Available capabilities
  - `"query"` - SQL analysis only
  - `"update"` - Dashboard filtering only

## Explore the Data with Databot

Before building our dashboard, let's use Databot to explore the mortgage data. Databot is an experimental AI assistant that dramatically accelerates exploratory data analysis (EDA), enabling you to complete analyses in minutes rather than hours. Unlike general coding assistants, Databot is purpose-built for EDA with rapid iteration of short code snippets that execute quickly.

### Open Databot

1. Open the Command Palette (`Cmd/Ctrl+Shift+P`).

2. Type "Open Databot" and select it.

3. The Databot panel will open, ready to analyze your mortgage data.

### Explore the Mortgage Data

With your connection to the `HOME_MORTGAGE_DISCLOSURE` table established (from the previous section), you can now ask Databot to explore the data. Try these prompts:

**Understand the dataset structure:**
```
Explore the mortgage_data and summarize its structure
```

Databot will generate and execute code to show you the columns, data types, and basic statistics.

**Investigate specific patterns:**
```
Explore the relationship between loan amounts and property types
```

Databot will create visualizations and statistical summaries to help you understand lending patterns.

**Compare trends across geography:**
```
How do loan approval rates vary across different states?
```

Databot will analyze geographic patterns and create appropriate visualizations.

**Identify data quality issues:**
```
Check for missing values and data quality issues in the mortgage data
```

Databot will examine the dataset for completeness and potential problems.

### Review Generated Code

As Databot works, it will show you the code it generates before executing it. Review this code to:

- Verify the analysis approach is appropriate
- Understand the transformations being applied
- Learn techniques you can reuse in your own work
- Catch any potential issues before execution

### Create a Quarto Report from Your Exploration

Once you've explored the data with Databot, you can have Databot create a Quarto report that captures your findings and can be published to Connect.

Ask Databot to create the report:

```
Create a .qmd file summarizing the mortgage data exploration
```

Databot will generate a Quarto document that includes:
- The code from your exploration organized into executable chunks
- Narrative text explaining the analyses and findings
- Visualizations and results from your data exploration

After Databot creates the file:

1. Review the generated `.qmd` file to verify the content and narrative.

2. Render the report to preview it:
   - Click the **Preview** button, or
   - Run `quarto preview <filename>.qmd` in the terminal

3. Publish the report to Posit Connect to share with your team:
   - Open the Command Palette (`Cmd/Ctrl+Shift+P`) and select **Publish to Connect**
   - Select your `.qmd` file
   - Configure publishing options and click **Publish**

Your EDA report will now be accessible to your team on Connect, providing context and insights before they use the interactive dashboard.

### Key Insights to Look For

As you explore the data with Databot, consider these questions:

- **Geographic patterns:** Which states or regions have the highest/lowest loan volumes or approval rates?
- **Loan characteristics:** What are the typical loan amounts, interest rates, and loan types?
- **Property patterns:** How do lending patterns vary by property type (single-family, multifamily, etc.)?
- **Data coverage:** Which fields have the most complete data across geographic areas?

These insights will help you understand the dataset and inform how you build the interactive dashboard in the next section.

## Chat with Positron Assistant using Cortex AI

Since we can use natural language to interact with our data via Positron Assistant, we don't need to write any additional SQL queries or data manipulation code. Positron Assistant will handle that for us based on our natural language prompts.

To start a chat with Positron Assistant, click on the Positron Assistant icon in the toolbar:

![](assets/positron-assistant.png)

You can select your model based on what you have available via Cortex AI. This example will use Claude Haiku 4.5.

### Create a Dashboard using Shiny

Ask Positron Assistant to create a Dashboard using Shiny, `chatlas`, and `querychat`. Enter this prompt:

```
Let's create an interactive LLM-powered dashboard with Shiny, querychat, and chatlas that will allow users to ask natural language questions to analyze the data in the HOME_MORTGAGE_DISCLOSURE table. Help me build an LLM-powered dashboard with Shiny, querychat, and chatlas that will let users ask natural language questions to explore the HMDA mortgage data. I want to be able to ask questions like, "What are the most common loan types?" or "How do loan approval rates vary by state?"
```

<!-- TO-DO: ADD ADDITIONAL CONTENT OR DIRECTIONS WHILE TESTING -->

## Deploy to Posit Connect

Now that your dashboard works locally, let's deploy it to Connect so your team can access it. Deployment is a one-click process. Because Workbench and Connect run within the same Native App, the complex network and authentication challenges are eliminated. Once you click “Deploy” in Positron, Connect handles dependency management and ensures your code runs successfully as a deployed artifact.

### Configure Connect Publishing

In Positron, open the command palette (Cmd/Ctrl+Shift+P) and search for "Publish to Connect". If this is your first time publishing, you'll need to configure your Connect server.

Click the "Add Server" button and enter:
- **Server URL**: Your Posit Connect server address (e.g., `https://connect.your-company.com`)
- **API Key**: Generate an API key from your Connect account settings

<!-- TODO: Take screenshot and save as assets/add_server.png
![](assets/add_server.png)
Screenshot should show: The Add Server dialog in Positron with server URL and API key fields
-->

### Publish Your Dashboard

With your `app.py` file open, click the **Deploy** button in the top left corner of Positron, or use the command palette (Cmd/Ctrl+Shift+P) to run "Publish Content".

Select the files to include:
- [x] app.py
- [x] config.py
- [ ] .env (do not publish credential files)

Choose your publishing options:
- **Title**: HMDA Mortgage Data Explorer
- **Access**: Select who should have access (e.g., "All Users" or specific groups)

Click **Publish** to deploy your dashboard to Connect.

<!-- TODO: Take screenshot and save as assets/publish_dialog.png
![](assets/publish_dialog.png)
Screenshot should show: The publish dialog with file selection and deployment options
-->

### Access Your Published Dashboard

Once publishing completes, Positron will provide a URL to your deployed application. The URL will look like:

```
https://connect.your-company.com/content/12345/
```

Click the link to open your dashboard. You can now share this URL with your team, and they can interact with the HMDA mortgage data using natural language queries.

<!-- TODO: Take screenshot and save as assets/deployed_app.png
![](assets/deployed_app.png)
Screenshot should show: The published dashboard running on Connect with the Connect URL visible
-->

> **Note:** Your dashboard will automatically reconnect to Snowflake using viewer-level authentication. Each user who accesses the dashboard will connect with their own Snowflake credentials, ensuring they only see data they have permission to access.

## Conclusion And Resources

### Overview

In this guide, we built a complete LLM-powered dashboard for exploring HMDA mortgage data. We created a Snowflake warehouse to query public data, developed a Shiny application using Positron Assistant and Databot with the `chatlas` and `querychat` packages, and deployed the dashboard to Posit Connect where your team can access it securely.

The steps we took along the way easily transfer to other datasets and use cases. This pattern of combining Snowflake's data platform and Cortex AI, with Posit's authoring and publishing tools enables you to build and share powerful data applications quickly.

### What You Learned

- How to access and query Snowflake public datasets
- How to use Databot for exploratory data analysis and publishable Quarto reports
- How to build with multiple environments in mind using connection code that works seamlessly in Workbench, Connect, and local development
- How to implement viewer-level authentication to ensure each user connects to Snowflake with their own credentials
- How to create LLM-powered interfaces by configuring `chatlas` and `querychat` to enable natural language data exploration with Snowflake Cortex AI
- How to deploy to Connect to publish both Quarto reports and Shiny applications with one-click deployment from Workbench

### Resources

- **Snowflake Cortex AI Documentation**: [Cortex AI Functions](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions)
- **Snowflake Public Data**: [Using Snowflake Data Marketplace](https://docs.snowflake.com/en/user-guide/data-marketplace)
- **Positron**: [Positron Documentation](https://positron.posit.co/)
- **Databot**: [Databot Documentation](https://positron.posit.co/databot.html)
- **`chatlas` Package**: [Documentation](https://posit-dev.github.io/chatlas/)
- **`querychat` Package**: [Documentation](https://posit-dev.github.io/querychat/py/index.html)
- **Shiny**: [Documentation](https://shiny.posit.co/)
- **Posit Workbench**: [User Guide](https://docs.posit.co/ide/server-pro/user/)
- **Posit Connect**: [User Guide](https://docs.posit.co/connect/user/)
- **Related Guides**: [Analyze Data with Python Using Posit Team](https://quickstarts.snowflake.com/guide/analyze-data-with-python-using-posit-team/)
