# Reservation

- Introduction

  A core reservation service that solves the problem of reserving a resource for a period time.





# 【项目】预定系统

- Q：如何用rust处理资源预定类的问题？

  (有意思的、使用 database 和 gRPC 解决真实世界问题的 live coding)



- 需求分析

  会议室预定

- 服务接口 (资源预定层)

  [excalidraw 画图](https://excalidraw.com/)
  
  gRPC interface：tonic
  
  DB connection pool：sqlx (非orm 轻)
  
  ![Snipaste_2024-04-06_21-27-54](res\Snipaste_2024-04-06_21-27-54.png)





## 项目初始化

- 环境搭建

  初始化仓库
  
  ```
  cd /d/code2/rust-code
  git clone https://github.com/tyrchen/simple-dns.git
  
  mkdir reservation/ && cd reservation/
  code .
  
  touch Cargo.toml README.md
  cp -r ../simple-dns/deny.toml .
  cp -r ../simple-dns/.github .
  cp -r ../simple-dns/.pre-commit-config.yaml .
  cp -r ../simple-dns/_typos.toml .
  
  ```

  D:\code2\rust-code\reservation\.github\workflows\build.yml
  
  ```yml
  name: build
  
  on:
    push:
      branches:
        - master
    pull_request:
      branches:
        - master
  
  jobs:
    build-rust:
      strategy:
        matrix:
          platform: [ubuntu-latest]
      runs-on: ${{ matrix.platform }}
      steps:
        - uses: actions/checkout@v3
          with:
            fetch-depth: 0
        - name: Install Rust
          run: rustup toolchain install stable --component llvm-tools-preview
        - name: Install cargo-llvm-cov
          uses: taiki-e/install-action@cargo-llvm-cov
        - name: install nextest
          uses: taiki-e/install-action@nextest
        - uses: Swatinem/rust-cache@v1
        - name: Check code format
          run: cargo fmt -- --check
        - name: Check the package for errors
          run: cargo check --all
        - name: Lint rust sources
          run: cargo clippy --all-targets --all-features --tests --benches -- -D warnings
        - name: Execute rust tests
          run: cargo nextest run --all-features
  ```

  D:\code2\rust-code\reservation\.pre-commit-config.yaml
  
  ```yaml
  fail_fast: false
  repos:
    - repo: https://github.com/pre-commit/pre-commit-hooks
      rev: v2.3.0
      hooks:
        - id: check-byte-order-marker
        - id: check-case-conflict
        - id: check-merge-conflict
        - id: check-symlinks
        - id: check-yaml
        - id: end-of-file-fixer
        - id: mixed-line-ending
        - id: trailing-whitespace
    - repo: https://github.com/psf/black
      rev: 19.3b0
      hooks:
        - id: black
    - repo: https://github.com/crate-ci/typos
      rev: v1.8.1
      hooks:
        - id: typos
    - repo: local
      hooks:
        - id: cargo-fmt
          name: cargo fmt
          description: Format files with rustfmt.
          entry: bash -c 'cargo fmt -- --check'
          language: rust
          files: \.rs$
          args: [ ]
        - id: cargo-deny
          name: cargo deny check
          description: Check cargo dependencies
          entry: bash -c 'cargo deny check'
          language: rust
          files: \.rs$
          args: [ ]
        - id: cargo-check
          name: cargo check
          description: Check the package for errors.
          entry: bash -c 'cargo check --all'
          language: rust
          files: \.rs$
          pass_filenames: false
        - id: cargo-clippy
          name: cargo clippy
          description: Lint rust sources
          entry: bash -c 'cargo clippy --all-targets --all-features --tests --benches -- -D warnings'
          language: rust
          files: \.rs$
          pass_filenames: false
        - id: cargo-test
          name: cargo test
          description: unit test for the project
          entry: bash -c 'cargo nextest run --all-features'
          language: rust
          files: \.rs$
          pass_filenames: false
  ```

  一个小工具 (暂时不用)

  [pre-commit](https://pre-commit.com/)：在 `git commit` 时检查是否符合 `.pre-commit-config.yaml` 的规范
  
  ```
  pip install pre-commit
  pre-commit --version  # pre-commit 3.5.0
  
  ```

  初始化仓库
  
  ```
  git init
  pre-commit install
  git status
  
  cargo new abi --lib
  cargo new reservation --lib
  cargo new service
  
  ```

  D:\code2\rust-code\reservation\service\Cargo.toml
  
  ```toml
  [package]
  name = "reservation-service"
  version = "0.1.0"
  edition = "2021"
  
  # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
  
  [dependencies]
  
  ```

  D:\code2\rust-code\reservation\Cargo.toml
  
  ```toml
  [workspace]
  resolver = "2"
  members = ["abi", "reservation", "service"]
  
  ```
  
  初始化仓库 

  ```
  cargo build  # ok
  
  git add .
  git status
  git commit -a
  
  ```
  
  遇到问题 (暂时不用)
  
  [LF和CRLF](https://blog.csdn.net/qq_36727756/article/details/105164225)
  
  ```
  cargo install cargo-deny
  cargo install cargo-nextest
  
  # 文件末尾的空行，这将禁用end-of-file-fixer钩子
  git config --unset hook.endoffile
  # 尾部的空白字符，这将禁用trailing-whitespace钩子
  git config --unset hook.trailingwhitespace
  
  # 配置Git以跟踪LF作为行尾序列
  git config --global core.autocrlf input
  git config --local core.autocrlf false
  # 如果你想要Git在提交时将CRLF转换为LF，并在检出时保持LF
  git config --global core.autocrlf true
  # 清除缓存
  git rm --cached -r .
  git commit -m "Rewrite history to use LF line endings"
  
  
  # 编码风格问题
  cargo install clippy
  cargo clippy
  # Clippy已经不再通过crates.io提供，而是作为一个Rust工具链组件可用
  rustup component add clippy-preview
  cargo clippy --version
  ```
  
  远程仓库
  
  ```
  git remote add origin https://github.com/Time1043/reservation.git
  git branch -M main
  git push -u origin main
  ```
  
  
  
  1







































