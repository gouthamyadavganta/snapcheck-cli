Installing SnapCheck
SnapCheck is distributed via PyPI and can be installed globally in a single command using pipx (recommended).
This ensures SnapCheck runs as a global CLI without messing with your Python environments.

Quick Install (Recommended)
bash
Copy
Edit
# Install pipx if you don’t have it
python -m pip install --user pipx
pipx ensurepath

# Install SnapCheck globally
pipx install snapcheck-cli

# Verify installation
snapcheck version
Alternate: Local Virtual Environment
bash
Copy
Edit
# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate   # Linux/macOS
venv\Scripts\activate      # Windows PowerShell

# Install SnapCheck
pip install snapcheck-cli

# Verify installation
snapcheck version
Next Steps
Getting Started Guide — Set up your first profile and run your first audit.

CLI Reference — Explore all available SnapCheck commands.

Security Guide — Configure OAuth, RBAC, and secrets handling.