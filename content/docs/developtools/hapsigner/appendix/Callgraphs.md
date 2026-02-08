# 调用链附录

## 目的

本文档记录 hapsigner 的关键调用链，帮助理解代码执行流程。

---

## 调用链 1: HAP 签名流程

```
main() [main.cpp:18]
└── ParamsRunTool::ProcessCmd() [params_run_tool.cpp:59]
    └── ParamsRunTool::DispatchParams()
        └── ParamsRunTool::RunSignApp() [params_run_tool.cpp:49]
            └── SignToolServiceImpl::SignHap() [sign_tool_service_impl.cpp]
                └── LocalSignProvider::Sign() [local_sign_provider.cpp]
                    └── SignHap::Sign() [sign_hap.cpp]
                        ├── ZipUtils::ParseZip() [zip_utils.cpp]
                        ├── DigestCommon::GetDigest() [digest_common.cpp]
                        ├── CodeSigning::GenerateCodeSign() [code_signing.cpp] (可选)
                        │   ├── MerkleTreeBuilder::GenerateMerkleTree() [merkle_tree_builder.cpp]
                        │   └── FsVerityGenerator::GenerateDescriptor() [fs_verity_generator.cpp]
                        └── BcPkcs7Generator::GeneratePkcs7() [bc_pkcs7_generator.cpp]
```

---

## 调用链 2: 证书生成流程

```
main() [main.cpp:18]
└── ParamsRunTool::ProcessCmd() [params_run_tool.cpp:59]
    └── ParamsRunTool::CallGenerators()
        └── ParamsRunTool::RunCa() [params_run_tool.cpp:52]
            └── SignToolServiceImpl::GenerateCA() [sign_tool_service_impl.cpp:37]
                ├── SignToolServiceImpl::HandleIssuerKeyAliasEmpty() [sign_tool_service_impl.cpp:146]
                ├── CertTools::GenerateCsr() [cert_tools.cpp]
                │   └── OpenSSL: X509_REQ_new()
                ├── CertTools::GenerateRootCertificate() [cert_tools.cpp]
                │   └── OpenSSL: X509_new(), X509_sign()
                └── SignToolServiceImpl::X509CertVerify() [sign_tool_service_impl.cpp]
```

---

## 调用链 3: HAP 验证流程

```
main() [main.cpp:18]
└── ParamsRunTool::ProcessCmd() [params_run_tool.cpp:59]
    └── ParamsRunTool::RunVerifyApp()
        └── SignToolServiceImpl::VerifyHapSigner() [sign_tool_service_impl.cpp]
            └── VerifyHap::Verify() [verify_hap.cpp]
                ├── HapSignerBlockUtils::FindEocdInHap() [hap_signer_block_utils.cpp]
                ├── VerifyHap::VerifyAppPkcs7() [verify_hap.cpp:347]
                │   ├── OpenSSL: PKCS7_parse()
                │   └── OpenSSL: PKCS7_verify()
                ├── VerifyHap::VerifyHapIntegrity() [verify_hap.cpp]
                │   └── DigestCommon::GetDigest() [digest_common.cpp]
                └── VerifyCodeSignature::Verify() [verify_code_signature.cpp] (可选)
```

---

## 调用链 4: Profile 签名流程

```
main() [main.cpp:18]
└── ParamsRunTool::ProcessCmd() [params_run_tool.cpp:59]
    └── ParamsRunTool::RunSignProfile()
        └── SignToolServiceImpl::SignProfile() [sign_tool_service_impl.cpp]
            └── ProfileSignTool::SignProfile() [profile_sign_tool.cpp]
                ├── ProfileInfo::FromJson() [profile_info.cpp]
                ├── ProfileInfo::ToDer() [profile_info.cpp]
                └── Pkcs7Data::GeneratePkcs7() [pkcs7_data.cpp]
                    └── OpenSSL: PKCS7_sign()
```

---

## 调用链 5: 密钥生成流程

```
main() [main.cpp:18]
└── ParamsRunTool::ProcessCmd() [params_run_tool.cpp:59]
    └── ParamsRunTool::CallGenerators()
        └── ParamsRunTool::RunKeypair() [params_run_tool.cpp:50]
            └── SignToolServiceImpl::GenerateKeyStore() [sign_tool_service_impl.cpp]
                └── KeyStoreHelper::GenerateKeyStore() [key_store_helper.cpp]
                    ├── OpenSSL: EVP_PKEY_keygen() (RSA/ECC)
                    └── OpenSSL: PKCS12_create()
```

---

## 关键函数索引

| 函数 | 文件 | 功能 |
|------|------|------|
| `ParamsRunTool::ProcessCmd` | cmd/src/params_run_tool.cpp | 命令处理主入口 |
| `SignToolServiceImpl::SignHap` | api/src/sign_tool_service_impl.cpp | HAP 签名服务 |
| `SignToolServiceImpl::VerifyHapSigner` | api/src/sign_tool_service_impl.cpp | HAP 验证服务 |
| `SignHap::Sign` | hap/sign/src/sign_hap.cpp | HAP 签名实现 |
| `VerifyHap::Verify` | hap/verify/src/verify_hap.cpp | HAP 验证实现 |
| `CodeSigning::GenerateCodeSign` | codesigning/sign/src/code_signing.cpp | 代码签名生成 |
| `MerkleTreeBuilder::GenerateMerkleTree` | codesigning/fsverity/src/merkle_tree_builder.cpp | Merkle 树构建 |
| `CertTools::GenerateCsr` | api/src/cert_tools.cpp | CSR 生成 |
| `KeyStoreHelper::GenerateKeyStore` | utils/src/key_store_helper.cpp | 密钥库生成 |
| `ProfileSignTool::SignProfile` | profile/src/profile_sign_tool.cpp | Profile 签名 |

---

## 相关链接

- [架构说明](../02_Architecture.md) - 了解系统设计
- [API 参考](../03_API_Reference.md) - 查看接口详情
