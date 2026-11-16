# Лабораторна робота №1. Git, GitLab, SSH, merge-конфлікт, релізи

Структура:

```
lab1/
├── .gitignore
├── README.md
├── notes.txt        # створюється під час роботи
└── src/
    ├── app.py
    └── index.html
```

Позначка 📸 означає, що тут треба зробити скріншот.

---

## 0. Налаштування Git і SSH

```bash
git config --global user.name "Volodymyr Kimachuk"
git config --global user.email "volodymyr.kimachuk.23@pnu.edu.ua"

# SSH-ключ
ssh-keygen -t ed25519 -C "volodymyr.kimachuk.23@pnu.edu.ua"   # Enter на всі питання
cat ~/.ssh/id_ed25519.pub                                       # скопіювати
```

GitLab → аватар → **Preferences → SSH Keys → Add new key** → вставити ключ → **Add key**.

```bash
ssh -T git@gitlab.com      # очікується: Welcome to GitLab, @username!
```
📸 Вивід `ssh -T git@gitlab.com`.

## 1. Локальний репозиторій і .gitignore

```bash
cd lab1
git init -b main
echo "debug" > debug.log        # перевірка .gitignore: цей файл не має потрапити в git
git status                      # debug.log не видно
git add .
git commit -m "Initial commit: add app, html and .gitignore"
```
📸 `git status`, на якому видно, що `debug.log` ігнорується.

## 2. Віддалений репозиторій на GitLab

GitLab → **New project → Create blank project** → назва `devops-lab1`,
**без** README (зняти галочку *Initialize repository with a README*).

```bash
git remote add origin git@gitlab.com:<username>/devops-lab1.git
git remote -v
git push -u origin main
```
📸 `git remote -v` і успішний `git push`.

## 3. Кілька комітів і відкат останнього

```bash
echo "Нотатки до лабораторної" > notes.txt
git add notes.txt && git commit -m "Add notes"

echo "<footer>Kimachuk 2026</footer>" >> src/index.html
git commit -am "Add footer"

echo "print('bad change')" >> src/app.py
git commit -am "Bad commit to be reverted"

git log --oneline
```

Відкат останнього коміту. Варіант A, безпечний для вже запушеної історії:
```bash
git revert HEAD --no-edit
git log --oneline
```
Варіант B, якщо коміт ще не пушили (стирає коміт з історії):
```bash
git reset --hard HEAD~1
```
```bash
git push
```
📸 `git log --oneline` до і після відкату.

## 4. Імітація merge-конфлікту

```bash
git checkout -b feature/title
sed -i '' 's/Лабораторна робота №1/Lab 1 — feature branch/' src/index.html   # macOS
# (на Linux: sed -i 's/.../.../' src/index.html)
git commit -am "Change title in feature branch"

git checkout main
sed -i '' 's/Лабораторна робота №1/Lab 1 — main branch/' src/index.html
git commit -am "Change title in main branch"

git merge feature/title
# CONFLICT (content): Merge conflict in src/index.html
```
📸 Повідомлення про конфлікт і `git status`.

```bash
cat src/index.html
```
📸 Маркери конфлікту у файлі:
```
<<<<<<< HEAD
  <h1>Lab 1 — main branch</h1>
=======
  <h1>Lab 1 — feature branch</h1>
>>>>>>> feature/title
```

Виправлення: відкрити файл (VS Code покаже кнопки *Accept Current / Incoming / Both*)
і залишити, наприклад:
```html
  <h1>Lab 1 — main + feature</h1>
```
📸 Файл під час редагування у VS Code.

```bash
git add src/index.html
git commit -m "Resolve merge conflict in index.html"
git log --oneline --graph --all
git push origin main feature/title
```
📸 `git log --graph` з merge-комітом.

## 5. Реліз через теги

```bash
git tag -a v1.0.0 -m "Release 1.0.0"
git tag
git show v1.0.0
git push origin v1.0.0
```
GitLab → **Deploy → Releases → New release** → вибрати тег `v1.0.0` →
назва `v1.0.0`, опис → **Create release**.

📸 `git tag` / `git show v1.0.0` і сторінка Releases у GitLab.
