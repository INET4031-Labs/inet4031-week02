# Update .gitignore at Repo Root

The `.env` file for this week contains database credentials and must never be committed to version control.

Add this line to `.gitignore` at your repository root:

```
week-2/.env
```

You can do this with:

```bash
echo "week-2/.env" >> .gitignore
```

Verify the rule is working:

```bash
git check-ignore -v week-2/.env
```

Expected output: a line showing which `.gitignore` rule matches `week-2/.env`.

Commit the `.gitignore` update along with `.env.example`:

```bash
git add .gitignore week-2/.env.example
git commit -m "chore: ignore .env files, add .env.example for week-2"
git push origin main
```
