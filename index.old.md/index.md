1. Инициализация пустого git-репозитория```

bash```
git add .
git commit -m "Update analysis"
git push
```
Определение путей проекта, имя на github, имя репозиторияб путь к html документу страницы:
```bash
# Установите эти переменные в начале
export REPO_NAME="mcp-stdio-tpl-rs-site"
export HTML_FILE="path/to/your/file.html"
export GITHUB_USERNAME="your-username"
```

2. **Создание и настройка репозитория:**
```bash
# Создать репозиторий
gh repo create $REPO_NAME --public --clone
cd $REPO_NAME

# Копировать HTML файл
cp "$HTML_FILE" index.html

# Добавить базовые файлы
git add .
git commit -m "Add MCP analysis page"
git push origin main
```

3. **Включение GitHub Pages:**
```bash
# Через GitHub CLI (если поддерживается)
gh api repos/$GITHUB_USERNAME/$REPO_NAME/pages \
  --method POST \
  --field source='{"branch":"main","path":"/"}'

# Или вручную: Settings → Pages → Source: Deploy from branch → main
```

## Вариант 3: Python скрипт с полной автоматизацией

```python
import os
import subprocess
import requests
import time
from pathlib import Path

class GitHubPagesDeployer:
    def __init__(self, username, token, repo_name, html_file):
        self.username = username
        self.token = token
        self.repo_name = repo_name
        self.html_file = html_file
        self.repo_url = f"https://github.com/{username}/{repo_name}"
        
    def create_repository(self):
        """Создать репозиторий через GitHub API"""
        url = "https://api.github.com/user/repos"
        headers = {
            "Authorization": f"token {self.token}",
            "Accept": "application/vnd.github.v3+json"
        }
        data = {
            "name": self.repo_name,
            "description": "MCP Analysis Site",
            "private": False,
            "auto_init": True
        }
        
        response = requests.post(url, headers=headers, json=data)
        if response.status_code == 201:
            print("✅ Репозиторий создан успешно")
            return True
        else:
            print(f"❌ Ошибка создания репозитория: {response.text}")
            return False
    
    def setup_local_repo(self):
        """Настроить локальный репозиторий"""
        commands = [
            f"git clone {self.repo_url}",
            f"cd {self.repo_name}",
            f"cp {self.html_file} index.html",
            "git add .",
            'git commit -m "Add MCP analysis page"',
            "git push origin main"
        ]
        
        for cmd in commands:
            result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
            if result.returncode != 0:
                print(f"❌ Ошибка выполнения команды: {cmd}")
                print(f"Ошибка: {result.stderr}")
                return False
        
        print("✅ Файлы загружены в репозиторий")
        return True
    
    def enable_github_pages(self):
        """Включить GitHub Pages"""
        url = f"https://api.github.com/repos/{self.username}/{self.repo_name}/pages"
        headers = {
            "Authorization": f"token {self.token}",
            "Accept": "application/vnd.github.v3+json"
        }
        data = {
            "source": {
                "branch": "master",
                "path": "/"
            }
        }
        
        response = requests.post(url, headers=headers, json=data)
        if response.status_code == 201:
            print("✅ GitHub Pages включен")
            return True
        else:
            print(f"❌ Ошибка включения GitHub Pages: {response.text}")
            return False
    
    def deploy(self):
        """Полный цикл деплоя"""
        print("🚀 Начинаем деплой...")
        
        if not self.create_repository():
            return False
            
        time.sleep(5)  # Ждем создания репозитория
        
        if not self.setup_local_repo():
            return False
            
        if not self.enable_github_pages():
            return False
        
        site_url = f"https://{self.username}.github.io/{self.repo_name}/"
        print(f"✅ Деплой завершен!")
        print(f"🔗 Сайт будет доступен по адресу: {site_url}")
        print("⏱️  Активация может занять несколько минут")
        
        return True

# Использование
if __name__ == "__main__":
    deployer = GitHubPagesDeployer(
        username="your-username",
        token="your-github-token",
        repo_name="mcp-analysis-site",
        html_file="path/to/your/file.html"
    )
    deployer.deploy()
```

## Вариант 4: Использование GitHub Actions для автоматического деплоя

### .github/workflows/deploy.yml

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Setup Pages
        uses: actions/configure-pages@v4
        
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
          
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## Выбор оптимального варианта

### Для разовых задач:
- **Вариант 2** (пошаговые команды) - простой и понятный

### Для регулярного использования:
- **Вариант 1** (bash-скрипт) - быстрый и эффективный

### Для команды разработчиков:
- **Вариант 4** (GitHub Actions) - автоматизация при каждом коммите

### Для сложных проектов:
- **Вариант 3** (Python скрипт) - максимальный контроль и гибкость

## Требования для автоматизации

1. **GitHub CLI** установлен и настроен
2. **Git** настроен с авторизацией
3. **GitHub Token** с правами на создание репозиториев (для API)
4. **Переменные окружения** настроены правильно
