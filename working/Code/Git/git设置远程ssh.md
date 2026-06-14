## 步骤
- 安装 Git
- 配置用户信息
- 设置 SSH 密钥（与远程仓库通信）
- 配置 Git 的全局或仓库特定的设置

### 配置用户信息
Git 需要知道您的名字和电子邮件地址，这样它就可以在提交时记录这些信息。
```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

### 生成SSH密钥
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### 添加SSH公钥到Git服务
在 GitHub 的账户设置中找到 SSH and GPG keys 部分，然后添加您的公钥。