# Generative AI Disclosure

Using AI tools for this challenge is allowed. Be honest here — an accurate
"I used AI heavily" is worth more to us than an inaccurate "I did not".

**Did you use AI tools?** (yes / no): yes

**Which ones?** (e.g. ChatGPT, Claude, Copilot, Cursor): ChatGPT

> If you answered **no**, write "N/A" in sections 1 and 2 and skip to
> section 3. That is a perfectly good answer.

---

## 1. Key prompts

List the main prompts you used. Paraphrasing is fine.

1. `Explain the F5 DevOps challenge step by step, including Docker, Docker Compose, Nginx, networking, integration tests, and GitHub Actions.`
2. `Help me diagnose the Docker build failure using the complete error logs, identify the root cause, and explain how to fix it.`
3. `Analyze the Nginx 502 Bad Gateway error and explain how Docker Compose service names and internal networking work.`
4. `Explain why integration tests fail during backend startup and how to implement a reliable Docker healthcheck.`
5. `Help me diagnose the GitHub Actions executable permission issue and verify the Git file mode.`
6. `Guide me through reducing the proxy image size, running Nginx as a non-root user, and implementing additional security improvements.`


---

## 2. Where the AI got it wrong

Describe **one concrete case** where the AI's answer was wrong, outdated, or
sub-optimal. Cover: what it suggested, how you noticed it was wrong, and what
you did instead. 3–5 sentences.
ChatGPT suggested running docker compose run --rm --no-deps --entrypoint nginx proxy -t to validate the Nginx configuration. However, the command failed with host not found in upstream "backend:8080" because the backend service was not running and its hostname could not be resolved. I identified the issue by reading the Nginx error output and recognizing that the --no-deps flag prevented Docker Compose from starting the backend dependency. I then started the backend container and reran the configuration test successfully.

> ...

---

## 3. Anything in your submission you could not fully explain?

We would rather know. Write "nothing" if that is the case.
Nothing. I reviewed the changes, understood the root causes of the issues, and verified the fixes through Docker builds, integration tests, and configuration checks.
> ...
