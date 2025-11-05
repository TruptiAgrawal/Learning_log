commit conventions : 
------------------
<type>(<scope>): <short summary>

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

feat(auth): add JWT-based login
fix(api): handle null user response
docs(readme): add setup instructions
style(ui): fix button alignment
refactor(core): simplify error handling
test(user): add unit tests for signup flow
chore(deps): update axios to v1.3.2
perf(cache): optimize image loading

My question : why do we say "Add JWT-based user authentication" instead of "Added JWT-based user authentication" ? 
Reasoning : 
When you write a commit message, imagine you’re telling Git what to do, not describing what you did.

Example
You type: git commit -m "Add JWT-based user authentication"
You’re basically saying: “Hey Git, apply this change — it will add JWT-based authentication.”

If you wrote: git commit -m "Added JWT-based user authentication"
It sounds like you’re describing something that already happened in the past — 
but **Git commits are the record of what happened.**
