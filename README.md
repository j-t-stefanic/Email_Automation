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
