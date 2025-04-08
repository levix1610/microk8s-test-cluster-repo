# 1. Navigate to directory
cd /path/to/your/directory

# 2. Initialize Git repo
git init

# 3. (Optional) Configure Git user
# git config user.name "Your Name"
# git config user.email "your.email@example.com"

# 4. Add your file
git add your_file_name.yml

# 5. Commit your file
git commit -m "Add my YAML file"

# 6. Add the remote GitHub repo
git remote add origin <repository_url>

# 7. Push the file to GitHub
git push -u origin main

# 8. to see the active git branch
git branch

# 9. See what the current URL for origin is:
git remote -v


