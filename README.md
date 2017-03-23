first_timer_scraper
===================

This is inspired by `first-timers-only` issues:
How can we make it possible for new-comers to contribute to a project.

This web service tries to solve this by looking at the data:

- What is it that first-timers contributed to?
- Did they continue to contribute?
- Which issues did they solve?
- Which projects do best at welcoming new contributors?

First Timer Contributions
-------------------------

To find first timer pull-requests, we look at organizations and
their repositories.

- **What are first timer repositories?**  
  - They MUST have at least two people contributing
  - They MUST have pull requests

- **What is a first-timer pull-request?**
  - They MUST have at least one commmit
  - In the repository history:
    - The issue commit MUST be preceded by commits of the author
    - At least one "other author" MUST have contributed before
    - There MUST NOT be a commit of the author before the "other author"
  - They may be on any branch.

- **What is a first-timer issue?**  
  There is no automatic linking between the pull-request and the issue.
  Thus, we must assume that one of the following takes place:
  - There is a public web page linked in the pull-request.  
    This is the case with external issue trackers like with django.
    We may assume certain issue trackers.
  - The issue is referenced as github issue in the pull-request.
  - The issue is referenced by commits by the author preceding the merge commit.

Implementation
--------------

For each organization sumbitted:
1. submit all repositories

For each submitted repository
1. clone
2. check if it can be a first-timer repository
3. extract for all commits
   - author email
   - author name
   - commit hash
   - time
4. Get all pull-requests
   - Get the first commit
   - Get the author, email
   - Get the github user issuing this pull-request
5. find first-timer pull-requests

API
---

`ENDING` is either `.html` or `.json`.

- General
  - `GET /`  
    Show a description of the project.
    - Link to this repository#readme.
    - Link to download the code /source.
    - How to contribute.
    - What this is about.
  - `GET /source`  
    Get the source code as a zip file.
- Github Authentication:
  - `GET /auth`
    Show a form to register a new authentication.
  - `POST /auth`  
    Add `username` and `password` to those usable to scrape github.
    They will be tried and removed if invalid.
- Organization
  - `GET /organizations<ENDING>`  
    List all organizations with links to their statuses.  
    ```
    {
      "loklak": {
        "name": "loklak"
        "repositories" : {
          "loklak_server": {
            "name": "loklak_server",
            "full_name": "loklak/loklak_server"
            "urls": {
              "html": "/repository/loklak/loklak_server.html",
              "json": "/repository/loklak/loklak_server.json",
              "github_html": "https://github.com/loklak/loklak_server",
              "github_api": "https://api.github.com/repos/loklak/loklak_server",
            }
          }
        },
        "urls": {
          "html": "/organization/loklak.html",
          "json": "/organization/loklak.json",
          "github_html": "https://github.com/loklak",
          "github_api": "https://api.github.com/orgs/loklak",
        },
        "last_update": "2011-01-26T19:01:12Z",
        "number_of_first_timers": 1,
        "first_timers": {
          "contributor1": {
            "name": "contributor1",
            "urls": {
              "html": "/user/contributor1.html",
              "json": "/user/contributor1.json",
              "github_html": "https://github.com/contributor1",
              "github_api": "https://api.github.com/users/contributor1",
            },
            "pull_requests_in_organization": {
              "loklak/loklak_server#122": {
                "repository_name": "loklak_server",
                "full_name": "loklak/loklak_server#122",
                "number": 122
                "created_at": "2011-01-26T19:01:12Z",
                "urls": {
                  "github_html": "https://github.com/loklak/loklak_server/pulls/122",
                  "github_api": "https://api.github.com/repos/loklak/loklak_server/pulls/122",
                }
              }
            }
          }
        }
      }
    }
    ```
    Users may be listed among them.
  - `GET /organization/<organization><ENDING>`  
    Get an organization and its status.
    Same as the element of `/organizations.json`
  - `POST /organization<ENDING>`  
    Submit an `organization` for scraping.
    This shows an html page with a link to the status of the organization.
- Repository
  - `GET /repositories<ENDING>`  
    List all repositories with links to their statuses.
    See below for the content of the list.
  - `GET /repository/<organization>/<repository><ENDING>`  
    Get the repository and its status.  
    ```
    {
      "name": "loklak_server",
      "full_name": "loklak/loklak_server"
      "urls": {
        "html": "/repository/loklak/loklak_server.html",
        "json": "/repository/loklak/loklak_server.json",
        "github_html": "https://github.com/loklak/loklak_server",
        "github_api": "https://api.github.com/repos/loklak/loklak_server",
      }, 
      "first_timers" : {
        "contributor1": {
          "pull_request": "loklak/loklak_server#122"
          "name": "contributor1",
          "urls": {
            "html": "/user/contributor1.html",
            "json": "/user/contributor1.json",
            "github_html": "https://github.com/contributor1",
            "github_api": "https://api.github.com/users/contributor1",
          },
        }
      },
      "first_timer_pull_requests": {
        "loklak/loklak_server#122" : {
          "repository_name": "loklak_server",
          "full_name": "loklak/loklak_server#122",
          "number": 122
          "created_at": "2011-01-26T19:01:12Z",
          "urls": {
            "github_html": "https://github.com/loklak/loklak_server/pulls/122",
            "github_api": "https://api.github.com/repos/loklak/loklak_server/pulls/122",
          },
          "author": "contributor1"
        }
      }
    }
    ```
  - `POST /repository`  
    Sumbit a `repository` for scraping
- User
  - `GET /users<ENDING>`  
    All the users and their statuses.
  - `GET /user/<user><ENDING>`  
    Return the status of this github user.
    - what are the first-timer repositories?
  - `POST /user/<user>`  
    Trace back the user's repositories to their origins.
    Submit all found organizations.

Command Line
------------

`python3 -m first_timer_scraper <DATA_FOLDER> <SECRETS_FOLDER>`

- `DATA_FOLDER` is the folder where the data is stored.
- `SECRETS_FOLDER` is the folder where the secrets are stored.

Windows:

`py -3.4 -m first_timer_scaper.app data secret`

Data Model
----------

The data model describes what is saved when scraped.

```
{
  "loklak": {
    "repos": {
      "loklak_server": {
        "first_timer_prs": {
          "112": "contributor1"
        },
        "last_update_requested": "2011-01-26T19:01:12Z"
      }
    },
    "last_update_requested": "2011-01-26T19:01:12Z",
    "first_timer_prs":{}
  },
  "contributor1": {
    "first_timer_prs": {
      "loklak/loklak_server": {
        "created_at" : "2011-01-26T19:01:12Z"
        "number": 112 // lowest number wins
      }
    },
    "last_update_requested": "2011-01-26T19:01:12Z",
    "repos": {}
  }
}
```

Further Reading
---------------

- Github API:
  - Rate Limiting: https://developer.github.com/v3/#rate-limiting
- PyGithub: https://github.com/PyGithub/PyGithub/

