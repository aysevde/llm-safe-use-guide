# Safe and Effective Use of LLMs for Coding

A practical guide for students on using AI coding tools (ChatGPT, Claude, GitHub Copilot, etc.) safely and effectively.

---

## Table of Contents

- [1. Hiding API Keys](#1-hiding-api-keys)
- [2. Using .gitignore](#2-using-gitignore)
- [3. What You Share with LLMs](#3-what-you-share-with-llms)
- [4. Personal Safety and Privacy](#4-personal-safety-and-privacy)
- [5. Using LLMs Effectively](#5-using-llms-effectively)
- [6. Academic Integrity](#6-academic-integrity)
- [7. Quick Reference Checklist](#7-quick-reference-checklist)

---

## 1. Hiding API Keys

An API key is like a password that lets your code access a service (OpenAI, Google Maps, Spotify, etc.). If someone gets your key, they can use the service on your account — and you get the bill.

### Bad Example

```python
# NEVER do this — anyone who sees your code gets your key
api_key = "sk-abc123secretkey456"
client = OpenAI(api_key=api_key)
```

### Good Example

```python
# The key lives outside your code
import os
api_key = os.environ.get("OPENAI_API_KEY")
client = OpenAI(api_key=api_key)
```

### How to Set Environment Variables

**Mac / Linux** — add to your shell config (~/.zshrc or ~/.bashrc):

```bash
export OPENAI_API_KEY="sk-abc123secretkey456"
```

Then restart your terminal or run: `source ~/.zshrc`

**Windows (Command Prompt):**

```cmd
set OPENAI_API_KEY=sk-abc123secretkey456
```

> **Note:** `set` is temporary — it only lasts for that terminal session. For a permanent variable, use `setx OPENAI_API_KEY "sk-abc123secretkey456"` instead.

**Windows (PowerShell):**

```powershell
$env:OPENAI_API_KEY = "sk-abc123secretkey456"
```

### Using .env Files

A common pattern is to store your keys in a file called `.env` in your project folder:

```
OPENAI_API_KEY=sk-abc123secretkey456
DATABASE_URL=postgres://user:pass@localhost/mydb
```

The critical rule: **never upload your .env file to GitHub.** The next section explains how to prevent that.

### Cost and Rate Limiting Awareness

If you are using a paid API (OpenAI, Google Cloud, etc.):

- **Set usage limits** on your API account dashboard to avoid surprise bills
- **Start with free tiers** — most services offer them for students
- **Be careful with loops** — one bad `while` loop can make hundreds of API calls in seconds and drain your credits
- **Monitor your usage** regularly on the provider's dashboard

---

## 2. Using .gitignore

A `.gitignore` file tells Git which files to ignore — they will never be committed or pushed to GitHub.

### Recommended .gitignore for Python Projects

```
# Secrets and environment files
.env
.env.local
*.key
*.pem

# Python cache
__pycache__/
*.pyc
*.pyo

# Jupyter Notebook checkpoints
.ipynb_checkpoints/

# Virtual environments
venv/
env/
.venv/

# IDE settings
.vscode/
.idea/

# OS files
.DS_Store
Thumbs.db
```

> **Warning:** If you already committed a secret before adding .gitignore, the file is still in your Git history. Adding .gitignore later does NOT remove it. You must **change or revoke the key immediately**.

### What to Do If You Accidentally Leak a Key

1. **Revoke the key immediately** — go to the service's dashboard (OpenAI, Google Cloud, AWS, etc.) and delete or regenerate the key
2. **Check your usage and billing** — look for any unauthorized activity that happened while the key was exposed
3. **Generate a new key** and update your environment variables

> The most important step is **revoking the key**. Even if you delete the file in a new commit, the old commit still contains it. Someone may have already copied it — so always revoke first.

### GitHub Secret Scanning

GitHub automatically scans public repositories for known API key patterns (OpenAI, AWS, Google Cloud, etc.). If it detects a leaked key, it will send you an email alert and may notify the service provider to revoke it automatically. However, do not rely on this — it only catches known patterns and may not detect every key format. Always follow the steps above yourself.

---

## 3. What You Share with LLMs

When you paste code into an AI tool, that text is sent to a server. Understanding what happens to your data is essential.

### What You Should NEVER Paste into an LLM

- **API keys, tokens, or passwords** — even if you ask the AI to "hide" them
- **Database connection strings** — they contain credentials
- **Personal data** — names, emails, phone numbers, student IDs
- **Proprietary or company code** — check your employer's AI policy first
- **Medical, financial, or legal information**

### Safe Alternatives

- Replace real keys with placeholders: `your-api-key-here`
- Use fake or dummy data instead of real personal information
- For sensitive projects, use **local models** (Ollama, LM Studio) that never send data online
- Read the tool's privacy policy before using it for work

---

## 4. Personal Safety and Privacy

LLMs generate code based on patterns — they do not understand security. Always review AI-generated code before running it.

### Red Flags in Generated Code

- **Hardcoded credentials** — the AI might put example keys that look real
- **File system access** — watch for `os.remove()`, `shutil.rmtree()`, or writing to unexpected paths
- **Network requests** — check where `requests.get()` or `urllib` calls are pointing
- **SQL injection** — never use f-strings to build SQL queries
- **eval() / exec()** — these run arbitrary code and are almost always dangerous
- **Unknown pip packages** — verify the package exists on [pypi.org](https://pypi.org) before installing

### SQL Injection Example

**Bad — user input goes directly into the query:**

```python
# DANGEROUS - SQL injection vulnerability
query = f"SELECT * FROM users WHERE name = '{user_input}'"
cursor.execute(query)
```

**Good — use parameterized queries:**

```python
# SAFE - the database handles escaping
query = "SELECT * FROM users WHERE name = ?"
cursor.execute(query, (user_input,))
```

### General Safety Rules

- Never run code you do not understand — read it first, line by line
- Test AI-generated code in an isolated environment before using it in your project
- Do not install packages the AI suggests without checking them on pypi.org first
- If the AI suggests disabling security features (SSL verification, firewalls), do not do it

---

## 5. Using LLMs Effectively

The quality of the output depends directly on the quality of your prompt.

### Good vs Bad Prompts

**Bad prompt:**

```
Make me a chart
```

**Good prompt:**

```
Using Python with matplotlib, create a bar chart showing the top 10 most
popular track genres from a pandas DataFrame called df. The genre column
is track_genre and use the popularity column for values. Add a title and
axis labels.
```

### Key Elements of a Good Prompt

- **Be specific** — state the language, library, and version
- **Give context** — describe your data, variable names, what you have so far
- **State the goal** — what should the output look like?
- **Set constraints** — "do not use external libraries" or "keep it under 20 lines"
- **Ask for explanations** — "explain each line" helps you learn, not just copy

### How to Iterate

You will rarely get perfect code on the first try. Treat it as a conversation:

1. Start with your main request
2. Review the output, then ask for specific changes
3. If something breaks, share the error message with context
4. Refine step by step rather than rewriting your entire prompt

### Verifying LLM Outputs

> **LLMs can hallucinate — they may invent functions, libraries, or APIs that do not exist.**

- **Read the code line by line** — do you understand what each line does?
- **Check function names** — search the official documentation to confirm they exist
- **Run it in a safe environment** — a Jupyter notebook or a test script
- **Test with known inputs** — if you know the expected output, verify it matches
- **Check the version** — the AI might use syntax from an older or newer Python version

### When to Use LLMs vs When to Be Careful

| Great for | Be careful with |
|-----------|----------------|
| Explaining error messages | Security-critical code |
| Boilerplate / starter code | Database queries with user input |
| Learning new libraries | Code handling passwords or payments |
| Debugging — paste the error | Production systems |
| Converting between formats | When you cannot verify the output |
| Writing tests | Blindly copying without reading |

---

## 6. Academic Integrity

Using LLMs as a student comes with ethical responsibilities. AI can help you learn, but it should never replace your own thinking.

### Rules to Follow

- **Check your university's AI policy** before using LLMs for any assignment — policies vary by course and instructor
- **Always disclose AI usage** when required — if your instructor asks whether you used AI, be honest
- **Submitting AI-generated code as your own work is plagiarism** unless your instructor explicitly allows it
- **Using AI to learn is not the same as using AI to cheat** — asking an LLM to explain a concept is learning; asking it to solve your homework is not
- **You are responsible for the code you submit** — if AI-generated code has bugs or errors, you will be the one who is graded on it

### The Learning Test

Before submitting any AI-assisted work, ask yourself:

> *"If my instructor asked me to explain every line of this code on the spot, could I do it?"*

If the answer is no, you do not understand the code well enough to submit it.

---

## 7. Quick Reference Checklist

### Before Writing Code

- [ ] I have a `.gitignore` file in my project
- [ ] My `.env` file is listed in `.gitignore`
- [ ] API keys are stored in environment variables, not in my code

### Before Using an LLM

- [ ] I have removed all real API keys, passwords, and personal data from my prompt
- [ ] I am using placeholder values instead of real credentials
- [ ] I understand the privacy policy of the AI tool I am using
- [ ] I am not pasting exam questions or graded assignment prompts into an LLM
- [ ] I have checked my course's AI usage policy

### Before Running AI-Generated Code

- [ ] I have read the code line by line and understand what it does
- [ ] I have checked that imported packages actually exist on pypi.org
- [ ] There are no hardcoded credentials in the generated code
- [ ] There are no dangerous functions (eval, exec, os.remove) used unnecessarily
- [ ] SQL queries use parameterized inputs, not f-strings
- [ ] File and network operations point to expected locations

### Before Pushing to GitHub

- [ ] I have run `git status` to check what files will be committed
- [ ] No `.env`, `.key`, or credential files are in the commit
- [ ] No API keys or passwords are hardcoded anywhere in my code
- [ ] My `.gitignore` is working correctly

### General Habits

- [ ] I never copy-paste LLM code without reading it first
- [ ] I test code in a safe environment before using it in my project
- [ ] I ask the LLM to explain code I do not understand
- [ ] I verify that functions and libraries suggested by the LLM actually exist
- [ ] I keep my API keys rotated and revoke any that may have been exposed

---

*Remember: LLMs are tools, not experts. You are the developer — the AI is your assistant, not your replacement. Always think critically about what it produces.*
