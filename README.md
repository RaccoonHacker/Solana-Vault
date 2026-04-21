# ⚓ Solana Vault - Solana PDA 金库

这是一个基于 **Anchor Framework** 开发的 Solana 智能合约（Program）。它演示了如何利用 **PDA (Program Derived Address)** 为每个用户创建私有的、去中心化的资金金库。

---

## 🛠️ 开发环境版本 (Environment)

为确保合约编译与部署成功，建议使用以下或更高版本的工具链：

- **Rust**: `1.91.1 (ed61e7d7e 2025-11-07)`
- **Solana CLI**: `3.0.10`
- **Anchor CLI**: `0.32.1`
- **Node.js**: `v24.10.0`
- **Yarn**: `1.22.1`
- **Surfpool CLI**: `0.12.0`

---

## 📖 项目简介

`Solana Vault` 允许用户将 SOL 存入一个由程序控制的账户（Vault）中，并确保只有存入者本人（Signer）能够提取这些资金。该项目是学习 Solana PDA 签名、CPI（跨程序调用）以及 Anchor 基本约束的核心案例。

## ✨ 核心特性

- **PDA 账户映射**：通过种子 `[b"vault", signer_key]` 为每个用户派生唯一的金库地址。
- **租金安全检查**：存入金额必须大于最低租金豁免额（Rent Exempt），确保账户状态在链上持续存在。
- **自动化权限校验**：利用 Anchor 的 `#[derive(Accounts)]` 宏自动验证调用者身份与金库地址的匹配性。
- **安全取款**：通过 PDA 种子在合约内部进行签名（`CpiContext::new_with_signer`），实现资金安全原路返回。

---

## 🏗️ 技术架构

### 1. 程序信息
- **Program ID**: `DHaP8mcPgrjJEgU8ZREF7ajE74XzsZrpTKXQuyJmRBsT`
- **开发语言**: Rust (Anchor Framework)
- **部署网络**: Solana Devnet

### 2. 指令说明 (Instructions)

| 函数名 | 操作 | 核心逻辑 |
| :--- | :--- | :--- |
| `deposit` | 存款 | 检查金库是否为空 -> 验证金额是否覆盖租金 -> 执行 System Program 转账 |
| `withdraw` | 取款 | 校验金库余额 -> 生成 PDA 签名种子 -> 将所有 Lamports 转回签名者账户 |

### 3. 数据校验模型 (Account Context)

```rust
#[derive(Accounts)]
pub struct VaultAction<'info> {
    #[account(mut)]
    pub signer: Signer<'info>, // 支付 Gas 费并拥有金库的钱包
    #[account(
        mut,
        seeds = [b"vault", signer.key().as_ref()], // 定义 PDA 派生路径
        bump, // 自动处理并存储偏移量 bump
    )]
    pub vault: SystemAccount<'info>,
    pub system_program: Program<'info, System>,
}
