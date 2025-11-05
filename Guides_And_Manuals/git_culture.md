Version Control: 
---------------
A VCS tool, or Version Control System tool, is software used to track and manage changes to files over time, allowing for collaboration and rollback to previous versions. 

Commit Conventions: 
------------------
 [type][scope]: short summary 

| Type       | Meaning                                    |
| ---------- | ------------------------------------------ |
| `feat`     | New feature                                |
| `fix`      | Bug fix                                    |
| `docs`     | Documentation only                         |
| `style`    | Formatting, linting, no logic change       |
| `refactor` | Code restructuring without behavior change |
| `test`     | Adding or updating tests                   |
| `chore`    | Maintenance, CI/CD, build updates          |
| `perf`     | Performance improvements                   |
| `revert`   | Undo a previous commit                     |

docs(readme): add setup instructions

Curiousity Dump:   
---------------
-> My question : 
    why do we say "Add JWT-based user authentication" instead of "Added JWT-based user authentication" ? 
-> Reasoning : 
    When you write a commit message, imagine you’re telling Git what to do, not describing what you did.
    Example:
    You type: git commit -m "Add JWT-based user authentication"
    You’re basically saying: “Hey Git, apply this change — it will add JWT-based authentication.”
    BUT 
    If you wrote: git commit -m "Added JWT-based user authentication"
    It sounds like you’re describing something that already happened in the past — 
    but **Git commits are the record of what happened.**

Refernces: 
---------
https://docs.github.com/en/repositories/creating-and-managing-repositories/best-practices-for-repositories 
https://medium.com/code-factory-berlin/github-repository-structure-best-practices-248e6effc405 
https://dev.to/pwd9000/github-repository-best-practices-23ck 
https://github.com/CodeFactoryBerlin/OpenSourceRepoTemplate?tab=readme-ov-file#compass-roadmap
