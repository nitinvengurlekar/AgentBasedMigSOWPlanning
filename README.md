# AgentBasedMigSOWPlanning - Oracle Migration Agent

A multi-step AI agent built with LangChain and OpenAI Functions to generate an Oracle Cloud database migration guide, convert documents to PDF, and send email summaries.

## Features

- Generate a 3-part migration guide (Planning, Execution, Post‑Migration Validation) based on user inputs.
- Render a Statement of Work (SOW) with effort estimates.
- Convert generated Markdown documents into downloadable PDFs.
- Send high-level email summaries with guide and SOW details.

## Prerequisites

- Python 3.8 or higher
- An OpenAI API key (set as `OPENAI_API_KEY`)

## Installation

```bash
git clone https://github.com/yourusername/oracle-migration-agent.git
cd oracle-migration-agent
pip install -r requirements.txt
