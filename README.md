# 🚀 CI/CD for Python Web Apps with GitHub Actions, Docker, and AWS EC2

Setting up an automated CI/CD pipeline is one of the best ways to make your Python web application production-ready. In this guide, we’ll walk through building a complete pipeline using **GitHub Actions**, **Poetry** for dependency management, **pytest** and **flake8** for testing and linting, and **Docker** for containerization. The final image will be deployed to an AWS EC2 instance using SSH.

By the end, you’ll have a setup that:
- Installs Python dependencies via Poetry
- Runs unit tests and style checks
- Builds and pushes a Docker image to GitHub Container Registry (GHCR)
- Deploys your app to a running EC2 instance

Let’s dive in. 🧑‍💻

---

## 🧱 1. Set Up a Basic Python Project with Poetry

Start by creating a basic Python project structure:

```bash
mkdir python-webapp && cd python-webapp
poetry init --name python-webapp --dependency flask --dev-dependency pytest --dev-dependency flake8 --dev-dependency black
```

Structure:

```
python-webapp/
├── app/
│   └── main.py
├── tests/
│   └── test_main.py
├── pyproject.toml
└── README.md
```

Sample `main.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, World!"
```

Sample test:

```python
# tests/test_main.py
def test_home():
    assert 1 + 1 == 2  # Replace with real tests later
```

---

## 🧪 2. Add a GitHub Actions Workflow

Inside your project root, create a workflow file:

```bash
mkdir -p .github/workflows
touch .github/workflows/ci-cd.yml
```

Now let’s define the CI/CD pipeline.

---

## 🛠️ 3. The Full `ci-cd.yml` Breakdown

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install Poetry
        run: curl -sSL https://install.python-poetry.org | python3 -

      - name: Add Poetry to PATH
        run: echo "$HOME/.local/bin" >> $GITHUB_PATH

      - name: Install dependencies
        run: poetry install

      - name: Run tests
        run: poetry run pytest

      - name: Lint with flake8
        run: poetry run flake8 app tests

      - name: Format check with black
        run: poetry run black --check app tests
```

### 🔍 What's happening here?

- **Set up Python & Poetry**: We're installing Poetry to manage dependencies and running it in the GitHub Actions runner.
- **Run tests**: `pytest` checks your app logic.
- **Lint & format**: `flake8` and `black` ensure your code is readable and clean.

---

## 🐳 4. Add a Dockerfile

Here’s a simple `Dockerfile` for the Flask app:

```Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY pyproject.toml poetry.lock* /app/

RUN pip install --upgrade pip \
    && pip install poetry \
    && poetry config virtualenvs.create false \
    && poetry install --no-interaction --no-ansi

COPY . /app

CMD ["python", "app/main.py"]
```

Make sure your app runs without Flask's development server in production. Use `gunicorn` in real apps.

---
