# add_topic_prod_non_prod

# GitHub Topics Checker

This script checks if all repositories in a specified GitHub organization have the topics "production" or "non-production". If any repositories are missing these topics, the script will prompt you to open an issue requesting the addition of the required topics.

## Prerequisites

- Python 3.x
- GitHub account with a personal access token
- Installed packages: `PyGithub`, `python-dotenv`, `requests`

## Setup

1. Clone this repository to your local machine.
2. Install the required Python packages:
   ```bash
   pip install PyGithub python-dotenv requests