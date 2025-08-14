# Git & GitHub — Hướng Dẫn Sử Dụng Chi Tiết

> Cẩm nang thực chiến cho cá nhân & nhóm: cài đặt và cấu hình Git, SSH/GPG/SSH-signing, GitHub Flow (branch/PR), GitHub CLI, Branch protection & CODEOWNERS, Git LFS, rebase/cherry-pick/stash/revert/bisect, Actions & Secrets, `.gitignore`/`.gitattributes`, cùng xử lý lỗi phổ biến.

---

## Mục lục
- [1) Chuẩn bị nhanh](#1-chuẩn-bị-nhanh)
- [2) Cài đặt & cấu hình Git](#2-cài-đặt--cấu-hình-git)
- [3) Kết nối GitHub (SSH & ký commit)](#3-kết-nối-github-ssh--ký-commit)
- [4) Tạo repo & đồng bộ với GitHub](#4-tạo-repo--đồng-bộ-với-github)
- [5) Quy trình làm việc nhóm (GitHub Flow)](#5-quy-trình-làm-việc-nhóm-github-flow)
- [6) Bảng lệnh thường dùng](#6-bảng-lệnh-thường-dùng)
- [7) Nâng cao: rebase/cherry-pick/stash/revert/bisect](#7-nâng-cao-rebasecherry-pickstashirevertbisect)
- [8) .gitignore & .gitattributes (EOL/CRLF)](#8-gitignore--gitattributes-eolcrlf)
- [9) Git LFS (file lớn) & Submodule](#9-git-lfs-file-lớn--submodule)
- [10) GitHub CLI (gh)](#10-github-cli-gh)
- [11) GitHub Actions & Secrets](#11-github-actions--secrets)
- [12) Branch protection & CODEOWNERS](#12-branch-protection--codeowners)
- [13) Xử lý lỗi hay gặp](#13-xử-lý-lỗi-hay-gặp)
- [14) Alias & mẹo](#14-alias--mẹo)
- [Tài liệu](#tài-liệu)
- [Giấy phép](#giấy-phép)

---

## 1) Chuẩn bị nhanh
```bash
# Đặt nhánh mặc định là main (Git >= 2.28)
git config --global init.defaultBranch main

# Danh tính
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"

# SSH cho GitHub (khuyên dùng)
ssh-keygen -t ed25519 -C "you@example.com"
eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub  # dán lên GitHub → Settings → SSH and GPG keys

# Khởi tạo & đẩy lên GitHub
git init -b main
echo "# My Project" > README.md
git add . && git commit -m "chore: init"
git remote add origin git@github.com:<owner>/<repo>.git
git push -u origin main
```

---

## 2) Cài đặt & cấu hình Git
```bash
git --version

git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main   # nhánh mặc định khi git init
git config --global log.date relative
git config --global color.ui auto
```
> `init.defaultBranch` giúp `git init` tạo nhánh đầu tiên là **main** thay vì **master**.

---

## 3) Kết nối GitHub (SSH & ký commit)

### SSH
```bash
ssh-keygen -t ed25519 -C "you@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub   # thêm vào GitHub
```

### Ký commit (Verified)
- **GPG**: cài `gpg`, tạo key → `git config --global user.signingkey <KEYID>` → `git config --global commit.gpgsign true`.
- **SSH signing** (Git ≥ 2.34): có thể dùng khóa SSH để ký commit/tag.
- Có thể bật **Require signed commits** trong `Branch protection` để bắt buộc Verified.

---

## 4) Tạo repo & đồng bộ với GitHub

### A) Tạo local rồi push
```bash
mkdir my-app && cd my-app
git init -b main
echo "# My App" > README.md
git add . && git commit -m "chore: initial commit"
git remote add origin git@github.com:<owner>/<repo>.git
git push -u origin main
```

### B) Clone repo có sẵn
```bash
git clone git@github.com:<owner>/<repo>.git
cd <repo>
```

### C) Đồng bộ thường ngày
```bash
git fetch --all --prune
git pull --rebase origin main
git push
```

---

## 5) Quy trình làm việc nhóm (GitHub Flow)

1. **Branch** từ `main`: `git switch -c feat/login`
2. Commit nhỏ, rõ ràng (*Conventional Commits* khuyến nghị)
3. Push & mở **Pull Request**
4. Review + CI xanh, rồi merge (squash/rebase)
5. Xoá nhánh tính năng sau khi merge

**Conventional Commits ví dụ:**
```
feat(auth): add OAuth login
fix(api): handle 401 refresh
docs(readme): installation steps
chore(ci): bump node to 20.x
```

---

## 6) Bảng lệnh thường dùng
```bash
# Nhánh
git branch
git switch -c feat/x
git branch -M main

# Trạng thái & lịch sử
git status
git diff
git diff --staged
git log --oneline --graph --decorate -n 20

# Staging & commit
git add .              # hoặc: git add -p (chọn hunk)
git commit -m "feat: ..."
git commit --amend     # sửa commit gần nhất

# Đồng bộ
git fetch --all --prune
git pull --rebase origin main
git push -u origin <branch>

# Tag
git tag v1.0.0
git push origin v1.0.0
```

---

## 7) Nâng cao: rebase/cherry-pick/stash/revert/bisect

**Rebase (sạch lịch sử)**
```bash
git pull --rebase origin main
git rebase -i origin/main
# sửa xung đột → git add <file> → git rebase --continue
# hủy rebase: git rebase --abort
```

**Cherry-pick (lấy commit lẻ)**
```bash
git cherry-pick <commit> [<commit2> ...]
```

**Stash (cất tạm)**
```bash
git stash push -u -m "WIP"
git stash list
git stash pop     # hoặc: git stash apply
```

**Revert (đảo commit bằng commit mới)**
```bash
git revert <commit>
```

**Bisect (tìm commit gây lỗi)**
```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
# lặp lại test → good/bad
git bisect reset
```

---

## 8) .gitignore & .gitattributes (EOL/CRLF)

**`.gitignore` ví dụ:**
```
# deps/build
node_modules/
dist/
.build/

# env & secrets
.env
.env.*

# IDE
.vscode/
.idea/
```

**`.gitattributes` chuẩn hoá EOL:**
```
* text=auto
```

---

## 9) Git LFS (file lớn) & Submodule

**Git LFS**
```bash
git lfs install
git lfs track "*.psd" "*.mp4" "*.zip"
git add .gitattributes
git add . && git commit -m "chore(lfs): track large assets"
git push
```

**Submodule**
```bash
git submodule add https://github.com/org/lib lib/
git submodule update --init --recursive
# sau này:
git submodule update --remote --merge
```

---

## 10) GitHub CLI (gh)
```bash
# Đăng nhập (PAT classic: cần scopes tối thiểu repo, read:org, gist)
gh auth login
gh auth status

# Tạo repo & đẩy source hiện tại
gh repo create <owner>/<repo> --public --source=. --push

# Pull Request
gh pr create --fill --base main --head feat/login
gh pr view --web
gh pr merge --squash --delete-branch

# Topics & mô tả
gh repo edit <owner>/<repo> \
  --description "My project" \
  --add-topic git --add-topic github
```

---

## 11) GitHub Actions & Secrets

**Secrets:** Repo → Settings → *Secrets and variables* → *Actions*.  
**Workflow Node.js tối giản:**
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm test
```

---

## 12) Branch protection & CODEOWNERS

**Branch protection** (Settings → Branches → Add rule):
- Require pull request reviews
- Require status checks to pass
- (Tuỳ chọn) Require signed commits
- Restrict who can push

**CODEOWNERS** (tự yêu cầu reviewer đúng người):
```
# .github/CODEOWNERS
*           @team/core
docs/*      @team/docs
src/api/*   @alice @bob
```

---

## 13) Xử lý lỗi hay gặp

**`error: src refspec main does not match any`**  
→ Chưa có commit đầu tiên hoặc nhánh chưa là `main`  
```
git add . && git commit -m "init"
git branch -M main
git push -u origin main
```

**`non-fast-forward` khi push**  
→ Repo GitHub đã có lịch sử; pull trước:  
```
git pull --rebase origin main
git push
```

**Thiếu scope khi dùng gh CLI**  
```
gh auth refresh -s repo -s read:org -s gist -s workflow
```

**File > 100MB** → dùng Git LFS.

**Lỗi CRLF/LF** → thêm `.gitattributes` với `* text=auto` rồi re-checkout nếu cần.

---

## 14) Alias & mẹo

**Alias hữu ích:**
```bash
git config --global alias.co "checkout"
git config --global alias.sw "switch"
git config --global alias.br "branch"
git config --global alias.cm "commit -m"
git config --global alias.st "status -sb"
git config --global alias.lg "log --graph --oneline --decorate --all"
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"
```

**Mẹo:**
- `git add -p` để stage theo hunk → commit gọn và chuẩn.
- Đặt `pull.rebase=true` để luôn rebase khi pull:
  ```bash
  git config --global pull.rebase true
  ```

---

## Tài liệu
- Pro Git (sách miễn phí): https://git-scm.com/book
- Git docs: https://git-scm.com/docs
- GitHub Docs: https://docs.github.com/
- Git LFS: https://git-lfs.com/
- GitHub CLI Manual: https://cli.github.com/manual/

---

## Giấy phép
Phát hành theo **MIT License** — xem [LICENSE](./LICENSE).  

> Nội dung & tên thương hiệu của bên thứ ba giữ nguyên giấy phép và quyền sở hữu tương ứng.

