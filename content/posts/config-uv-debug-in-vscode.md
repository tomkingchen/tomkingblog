---
title: "How to debug Python script with uv in VS Code"
date: 2026-02-20T10:43:36+11:00
featured_image: "https://blogfilesr2.tomking.xyz/vscode-debug.jpg"
draft: false
description: "Debug Python scripts with uv dependencies in VS Code using PEP 723 inline metadata."
---

Under `.vscode` folder:

Configure your `launch.json` as below:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: My uv Debug Test",
      "type": "debugpy",
      "request": "launch",
      "program": "./path/to/python/script.py",
      "console": "integratedTerminal",
      "args": ["--arg1", "--arg2"]
    }
  ]
}
```

Copy uv project dependencies from your `pyproject.toml` file.
```toml
dependencies = [
    "gql[all]<4",
    "requests>=2.32.5",
    "slack-sdk>=3.39.0",
    "urllib3>=2.6.3",
]
```

In your Python script `script.py` paste the dependencies to the top of the file and comment it like below.
```python
# /// script
# requires-python = ">=3.11"
# dependencies = [
#    "gql[all]<4",
#    "requests>=2.32.5",
#    "slack-sdk>=3.39.0",
#    "urllib3>=2.6.3",
# ]
# ///
```

Now you should be able to kickoff debug mode for your script with all required dependencies loaded.


## Reference
https://github.com/astral-sh/uv/issues/8558
