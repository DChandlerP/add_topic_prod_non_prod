import os
import requests
import time
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Constants
GITHUB_TOKEN = os.getenv('GITHUB_TOKEN')  # Ensure this is set in your .env file
ORGANIZATION_NAME = os.getenv('ORGANIZATION_NAME')  # Set your organization name in .env
TOPIC_REQUIRED = ['production', 'non-production']
RATE_LIMIT_THRESHOLD = 10  # Number of requests remaining to trigger a pause
GITHUB_API_URL = "https://api.github.com"

def get_headers():
    """ Return the headers required for GitHub API requests. """
    return {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github.v3+json"
    }

def fetch_repositories():
    """ Fetch all repositories in the given organization. """
    url = f"{GITHUB_API_URL}/orgs/{ORGANIZATION_NAME}/repos"
    response = requests.get(url, headers=get_headers())
    response.raise_for_status()
    return response.json()

def check_topics(repository):
    """ Check if the repository has the required topics. """
    url = f"{GITHUB_API_URL}/repos/{ORGANIZATION_NAME}/{repository['name']}/topics"
    response = requests.get(url, headers=get_headers())
    response.raise_for_status()
    topics = response.json().get('names', [])
    return any(topic in topics for topic in TOPIC_REQUIRED)

def list_repositories_without_required_topics(repositories):
    """ Return a list of repositories missing the required topics. """
    repos_missing_topics = []
    for repo in repositories:
        if not check_topics(repo):
            repos_missing_topics.append(repo)
    return repos_missing_topics

def handle_rate_limiting():
    """ Pause the script if nearing the GitHub rate limit. """
    url = f"{GITHUB_API_URL}/rate_limit"
    response = requests.get(url, headers=get_headers())
    response.raise_for_status()
    rate_limit = response.json()['rate']['remaining']
    if rate_limit < RATE_LIMIT_THRESHOLD:
        reset_time = response.json()['rate']['reset']
        current_time = time.time()
        sleep_duration = reset_time - current_time
        print(f"Rate limit nearing. Sleeping for {int(sleep_duration)} seconds.")
        time.sleep(max(sleep_duration, 0))

def issue_exists(repo, title):
    """ Check if an issue with the given title already exists in the repository. """
    url = f"{GITHUB_API_URL}/repos/{ORGANIZATION_NAME}/{repo['name']}/issues"
    params = {'state': 'all', 'per_page': 100}
    response = requests.get(url, headers=get_headers(), params=params)
    response.raise_for_status()
    issues = response.json()
    return any(issue['title'] == title for issue in issues)

def open_issue_on_repos(repos):
    """ Ask the user to confirm opening an issue for each repository missing topics. """
    for repo in repos:
        issue_title = "Please add a topic"
        if issue_exists(repo, issue_title):
            print(f"Issue with title '{issue_title}' already exists in {repo['full_name']}.")
            continue
        print(f"Would you like to open an issue on {repo['full_name']}? (y/n)")
        if input().strip().lower() == 'y':
            create_topic_issue(repo, issue_title)

def create_topic_issue(repo, issue_title):
    """ Create an issue on the repository to add required topics. """
    url = f"{GITHUB_API_URL}/repos/{ORGANIZATION_NAME}/{repo['name']}/issues"
    issue_body = ("This repository is missing the required GitHub topics 'production' or 'non-production'. "
                  "Please add one of these topics to the repository settings. "
                  "For more information on how to add topics, visit: "
                  "https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/classifying-your-repository-with-topics")
    data = {
        "title": issue_title,
        "body": issue_body
    }
    response = requests.post(url, headers=get_headers(), json=data)
    response.raise_for_status()
    print(f"Issue created on {repo['full_name']}")

def main():
    """ Main function to orchestrate the script execution. """
    handle_rate_limiting()
    repositories = fetch_repositories()
    repos_without_topics = list_repositories_without_required_topics(repositories)
    
    if not repos_without_topics:
        print("All repositories have the required topics.")
    else:
        print(f"Found {len(repos_without_topics)} repositories without the required topics.")
        open_issue_on_repos(repos_without_topics)

if __name__ == "__main__":
    main()