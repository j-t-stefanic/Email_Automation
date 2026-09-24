# Optimum_Email_Automation

I created this automation after getting very frustrated with my overloaded (extremely overloaded) Inbox of my Optimum Email account.  The spam emails that arrive not daily, but seemingly every 15 minutes finally clogged my Inbox and seemingly locked down my Outbox thus preventing me from sending emails.  It was so bad that trying to manually filter and delete just didn't cut it.  If you have had these types of frustrations, perhaps this project can assist you! 

## Optimum Email Automation — User Guide

This guide describes `Optimum_Email_Automation.py` as supplied. The script connects to an Optimum mailbox over IMAP, scans one mailbox, chooses actions from a JSON rules file, and optionally moves messages. Start with `analyze` to inspect the proposed actions. `apply` makes mailbox changes.

### Before you start

- Python **3.10 or newer**. The script uses only Python's standard library; there is no `pip install` step.
- An Optimum email account with IMAP access and its valid sign-in credentials. The script defaults to `mail.optimum.net` on port `993` using SSL. If your account requires a special app password, use the credential your email provider supplies for IMAP.
- A copy of `Optimum_Email_Automation.py` in your working directory. The commands below assume that filename. Replace it if you renamed the script.
- A local place to store `email_rules.json` and the generated CSV. The CSV contains senders and subjects, so keep it private.

Use `python` on Windows or `python3` on macOS/Linux in the examples below. On some Windows installations, `py` may be the appropriate command.

Check that Python and the script run:

```shell
python --version
python Optimum_Email_Automation.py --help
```

## 1. Create your rules file

```shell
python Optimum_Email_Automation.py init-rules
```

This creates `email_rules.json` in the current directory. The sample contains illustrative addresses that you **must replace** with your own matching criteria. The command refuses to overwrite an existing rules file. It does not connect to the mailbox and does not need `--username`.

Edit the file in a text editor. Here is a small example:

```json
{
  "rules": [
    {
      "name": "Keep tax documents",
      "match": {"subject": {"contains": "tax document"}},
      "action": "keep"
    },
    {
      "name": "Move store receipts",
      "match": {"sender_email": {"equals": "receipts@example.com"}},
      "action": "move",
      "folder": "Receipts"
    },
    {
      "name": "Send daily newsletter to Trash",
      "match": {
        "sender_email": {"ends_with": "@newsletter.example"},
        "subject": {"contains": "daily update"}
      },
      "action": "delete"
    }
  ]
}
```

Rules are evaluated **from top to bottom**. The first matching rule wins. Place a specific `keep` rule above a broader `move` or `delete` rule when you need that exception. Every condition within one rule must match. The recognized fields are `sender` (the full displayed From header), `sender_email` (the parsed email address), and `subject`. Matching is case-insensitive.

| Operator | Example | Meaning |
| --- | --- | --- |
| `equals` | `{"sender_email": {"equals": "person@example.com"}}` | The entire field equals the value. |
| `contains` | `{"subject": {"contains": "receipt"}}` | The field contains the text. |
| `ends_with` | `{"sender_email": {"ends_with": "@example.com"}}` | The field ends with the text. |
| `regex` | `{"subject": {"regex": "invoice [0-9]+"}}` | A Python regular expression matches somewhere in the field. |

Each rule's `action` is `move`, `delete`, or `keep`. A `move` rule also needs a nonempty `folder`. `delete` moves a message to the configured Trash folder; it is **not** a permanent deletion in this script. `keep` leaves it in the selected mailbox. A message with no matching rule receives the action `none` and stays in place.

Avoid empty `match` objects: they do not match anything. Use valid JSON with double quotes and no trailing commas. A malformed rule may fail during analysis rather than at file load time, so inspect the preview carefully.

## 2. Provide your username and password

The simplest option is to pass the email address as a global option. The script prompts for a password without displaying what you type:

```shell
python Optimum_Email_Automation.py --username you@optimum.net analyze
```

Alternatively set `OPTIMUM_EMAIL` for the username. You may also set `OPTIMUM_EMAIL_PASSWORD` for the password, but a password environment variable can be exposed to other local processes or retained in shell history if entered directly. The interactive prompt is a sensible default. Do not put a password into the rules file or commit credentials to GitHub.

**PowerShell (current session):**

```powershell
$env:OPTIMUM_EMAIL = 'you@optimum.net'
python .\Optimum_Email_Automation.py analyze
```

**macOS/Linux (current shell):**

```shell
export OPTIMUM_EMAIL='you@optimum.net'
python3 Optimum_Email_Automation.py analyze
```

If necessary, `--host` and `--port` override `mail.optimum.net` and `993`; verify changes with your provider before using them. `--mailbox` defaults to `INBOX`. These global options go **before** the command name.

## 3. Analyze the mailbox without changing it

```shell
python Optimum_Email_Automation.py --username you@optimum.net analyze --rules email_rules.json --output preview.csv
```

The script opens the chosen mailbox read-only, searches all messages in it, fetches their From, Subject, and Date headers, chooses the first matching rule, and writes the proposed actions to `preview.csv`. It does not fetch message bodies or attachments. If `--output` is omitted, it creates a timestamped file such as `optimum_email_plan_20260924_143000.csv` in the current directory. An existing file at the chosen `--output` path is overwritten.

The CSV columns are `uid`, `date`, `sender`, `sender_email`, `subject`, `rule`, `action`, and `destination`. UIDs identify messages **within the selected mailbox**, not across folders. Check especially the `delete` and `move` rows, the destination names, unexpected matches, and the printed counts for `move`, `delete`, `keep`, and `none`. A `none` count simply means messages did not match any rule.

To scan a different mailbox:

```shell
python Optimum_Email_Automation.py --username you@optimum.net --mailbox Archive analyze --output archive_preview.csv
```

This command analyzes **one selected mailbox**. It does not recursively scan every folder.
