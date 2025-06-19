# Contents

1. [Introduction](#introduction)
2. [The Application](#the-application)
3. [Versioning](#versioning)
4. [Automated Testing](#automated-testing)

---

# Introduction

In this tutorial, we will build an entire CI/CD pipeline for a simple web
application.

Our goals are to:

* version code with Git
* host code on GitHub
* automatically test code with GitHub Actions
* containerize the application with Docker
* store the container in a Docker registry
* pull the container with Kubernetes
* virtualize servers with Proxmox VE
* provision virtual machines with Terraform
* configure virtual machines with Ansible

---

# The Application

The application is a simple calculator with a web GUI, built with Flask and some
HTML.

To get started, fork and/or clone its repository:
```bash
git clone https://github.com/PierrickF/flask-calc.git
```

You can look at the `README.md` file to get it running.

Note how the Python code in `app.py` has been organized so that we could run
some unit tests:
```python
def add(num1, num2):
    return float(num1) + float(num2)

def subtract(num1, num2):
    return float(num1) - float(num2)

def multiply(num1, num2):
    return float(num1) * float(num2)

def divide(num1, num2):
    return float(num1) / float(num2)
```

We provide the tests for these functions in `tests.py` in the same repository:
```python
import unittest
from app import add, subtract, multiply, divide

class TestCalculations(unittest.TestCase):

    def test_sum(self):
        self.assertEqual(add(2, 3), 5, "The sum is wrong.")

    def test_diff(self):
        self.assertEqual(subtract(3, 2), 1, "The difference is wrong.")

    def test_product(self):
        self.assertEqual(multiply(2, 3), 6, "The product is wrong.")

    def test_quotient(self):
        self.assertEqual(divide(6, 3), 2, "The quotient is wrong.")

if __name__ == "main":
    unittest.main()
```

---

# Versioning

In order to trigger the whole pipeline, we will stick to a basic Git workflow.

```bash
# Stage modifications made to the code base
git add .

# Commit changes
git commit -m "(feat) Add sum()"

# Push the commit to the remote branch on GitHub
git push origin main
```

---

# Automated Testing

While we could have ran the tests locally on our machine, let's automate them
with GitHub Actions instead.
Not only is it more convenient, it will also allow us to trigger the next step
of the pipeline afterwards.

On the application's repository on GitHub, go to the `Actions` tab.
Either pick from the list or select `New workflow`, and choose `Python application`.

The selected job will be saved as code in the repository in the
`.github/workflows/` directory:
```yaml
name: Test Python Code

on:
  push:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Check out the repo
      uses: actions/checkout@v4

    - name: Set up Python 3.10
      uses: actions/setup-python@v3
      with:
        python-version: "3.10"

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Test with unittest
      run: |
        python -m unittest tests.py
```

Notice how we trimmed down the defaults as provided by GitHub to remove linting
and replace the testing framework from `pylint` to `unittest`.

The job runs when commits are pushed to `main`, which runs multiple commands
in an isolated Ubuntu environment, including `python -m unittest tests.py`.
