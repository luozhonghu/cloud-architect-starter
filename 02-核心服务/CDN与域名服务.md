# 核心服务 - CDN 与域名服务

## 一、CDN 概述

### 1.1 什么是 CDN？

CDN（Content Delivery Network，内容分发网络）是构建在现有网络基础之上的智能虚拟网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容。

### 1.2 CDN 工作原理

```
用户访问流程：
1. 用户访问域名（如：www.example.com）
   ↓
2. DNS 解析到 CDN 节点 IP
   ↓
3. 用户请求最近的 CDN 节点
   ↓
4. CDN 节点检查缓存
   - 有缓存：直接返回（边缘命中）
   - 无缓存：回源获取 → 缓存 → 返回
   ↓
5. 用户获得加速后的响应
```

### 1.3 CDN 架构

```
┌─────────────────────────────────────┐
│           用户                      │
└────────────┬────────────────────────┘
             ↓
┌─────────────────────────────────────┐
│           DNS                       │
│  ┌─────────────────────────────┐   │
│  │  CNAME 解析                │   │
│  │  www.example.com →         │   │
│  │  www.example.com.cdn.com   │   │
│  └─────────────────────────────┘   │
└────────────┬────────────────────────┘
             ↓
┌─────────────────────────────────────┐
│         GSLB（全局负载均衡）         │
│  • 就近路由                        │
│  • 健康检查                        │
│  • 故障切换                        │
└────────────┬────────────────────────┘
      ↙       ↓       ↘
┌────────┐ ┌────────┐ ┌────────┐
│北京节点 │ │上海节点 │ │广州节点 │
└────┬───┘ └────┬───┘ └────┬───┘
     │          │          │
     └────┬─────┴──────────┘
          ↓
    ┌──────────┐
    │  源站    │
    └──────────┘
```

### 1.4 CDN 优势

| 优势 | 说明 | 收益 |
|------|------|------|
| **加速访问** | 就近访问，降低延迟 | 响应时间降低 50%-80% |
| **降低带宽** | 边缘缓存，减少回源 | 节省带宽成本 30%-70% |
| **提升可用性** | 多节点冗余 | 可用性提升到 99.9%+ |
| **安全防护** | DDoS 防护、WAF | 提升安全性 |
| **减轻源站** | 请求分发到边缘 | 降低源站压力 80%+ |

## 二、阿里云 CDN 服务

### 2.1 开通与配置

#### 开通 CDN
```bash
# 阿里云控制台开通 CDN
访问：https://cdn.console.aliyun.com/

# 获取账号 ID 和 AccessKey
Settings -> Account Info -> Account ID
```

#### 添加域名
```bash
# 添加加速域名
aliyun cdn AddCdnDomain \
  --DomainName www.example.com \
  --CdnType web \
  --SourceType oss \
  --SourcePort "80" \
  --SourceIps oss-cn-hangzhou.aliyuncs.com \
  --SourceProtocol http
```

#### 配置回源
```bash
# 配置回源信息
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "domain_name", "argValue": "origin.example.com"},
        {"argName": "port", "argValue": "80"}
      ],
      "functionName": "origin_origin"
    },
    {
      "functionArgs": [
        {"argName": "http_port", "argValue": "80"},
        {"argName": "http_is_enable", "argValue": "true"}
      ],
      "functionName": "origin_protocol"
    }
  ]'
```

### 2.2 缓存配置

#### 缓存规则
```bash
# 配置缓存过期时间
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "cache_content", "argValue": "/path/*"},
        {"argName": "ttl", "argValue": "3600"}
      ],
      "functionName": "cache_config"
    }
  ]'
```

#### 缓存策略
| 文件类型 | 缓存时间 | 说明 |
|---------|---------|------|
| **静态图片** | 7 天 | jpg、png、gif、webp |
| **CSS/JS** | 1 天 | css、js |
| **HTML** | 不缓存 | 频繁更新 |
| **视频文件** | 30 天 | mp4、flv、m3u8 |
| **字体文件** | 30 天 | ttf、woff、woff2 |

### 2.3 HTTPS 配置

#### 配置 SSL 证书
```bash
# 上传证书
aliyun cdn SetCdnDomainSSLCertificate \
  --DomainName www.example.com \
  --CertName my-cert \
  --CertType upload \
  --SSLPublic "-----BEGIN CERTIFICATE-----\n..." \
  --SSLPrivateKey "-----BEGIN PRIVATE KEY-----\n..."
```

#### 强制 HTTPS
```bash
# 配置 HTTPS 协议
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "https_force"
    }
  ]'
```

### 2.4 性能优化

#### 开启 Gzip 压缩
```bash
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "gzip"
    }
  ]'
```

#### 开启 Brotli 压缩
```bash
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "brotli"
    }
  ]'
```

#### 图片优化
```bash
# 开启图片自动 WebP
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "webp_auto"
    }
  ]'
```

### 2.5 监控与分析

#### 查询流量数据
```bash
aliyun cdn DescribeDomainRealTimeQpsData \
  --DomainName www.example.com \
  --StartTime 2024-01-01T00:00:00Z \
  --EndTime 2024-01-01T23:59:59Z
```

#### 查询命中率
```bash
aliyun cdn DescribeDomainHitRateData \
  --DomainName www.example.com \
  --StartTime 2024-01-01T00:00:00Z \
  --EndTime 2024-01-01T23:59:59Z
```

#### 命中率指标
| 指标 | 说明 | 目标值 |
|------|------|--------|
| **字节命中率** | 缓存字节数 / 总字节数 | > 90% |
| **请求命中率** | 缓存请求数 / 总请求数 | > 95% |
| **回源流量** | 回源到源站的流量 | < 10% |

## 三、腾讯云 CDN 服务

### 3.1 开通与配置

#### 开通 CDN
```bash
# 腾讯云控制台开通 CDN
访问：https://console.cloud.tencent.com/cdn
```

#### 添加域名
```bash
# 添加加速域名
tencentcloud cdn AddCdnDomain \
  --Domain www.example.com \
  --Type web \
  --Origin origin.example.com \
  --OriginType domain
```

### 3.2 腾讯云特有功能

#### 全站加速
```bash
# 开启全站加速（动态加速）
tencentcloud cdn SetDomainConfig \
  --Domain www.example.com \
  --CacheKey full_url \
  --CacheTime 0
```

#### 视频加速
```bash
# 开启视频加速
tencentcloud cdn SetDomainConfig \
  --Domain www.example.com \
  --VideoOptimization on
```

## 四、域名服务

### 4.1 域名注册

#### 阿里云域名注册
```bash
# 查询域名
aliyun domain QueryDomainList \
  --KeyWord example \
  --ProductDomainType NewRegistration

# 注册域名
aliyun domain SaveSingleTaskForCreatingOrderRenew \
  --DomainName example.com \
  --RegistrantProfileId 123456 \
  --SubscriptionDuration 1
```

#### 腾讯云域名注册
```bash
# 查询域名
tencentcloud domain DescribeDomainList \
  --Limit 10

# 注册域名
tencentcloud domain CreateDomainOrder \
  --DomainNames example.com \
  --Period 1
```

### 4.2 DNS 解析

#### 阿里云 DNS（Alibaba Cloud DNS）

##### 开通 DNS
```bash
# 开通 DNS 解析服务
aliyun dns AddDomain \
  --DomainName example.com
```

##### 添加解析记录
```bash
# 添加 A 记录
aliyun dns AddDomainRecord \
  --DomainName example.com \
  --RR www \
  --Type A \
  --Value 1.2.3.4 \
  --TTL 600

# 添加 CNAME 记录（CDN）
aliyun dns AddDomainRecord \
  --DomainName example.com \
  --RR www \
  --Type CNAME \
  --Value www.example.com.cdn.aliyuncs.com \
  --TTL 600

# 添加 MX 记录（邮件）
aliyun dns AddDomainRecord \
  --DomainName example.com \
  --RR @ \
  --Type MX \
  --Value 10 mx1.example.com \
  --TTL 600
```

##### 批量添加解析
```bash
# 批量添加 DNS 解析
aliyun dns AddDomainRecords \
  --DomainName example.com \
  --Records '[
    {"RR": "www", "Type": "A", "Value": "1.2.3.4", "TTL": 600},
    {"RR": "api", "Type": "A", "Value": "1.2.3.5", "TTL": 600},
    {"RR": "cdn", "Type": "CNAME", "Value": "www.example.com.cdn.com", "TTL": 600}
  ]'
```

#### 腾讯云 DNS（Tencent Cloud DNSPod）

##### 开通 DNS
```bash
# 开通 DNS 解析服务
tencentcloud dnspod CreateDomain \
  --Domain example.com
```

##### 添加解析记录
```bash
# 添加 A 记录
tencentcloud dnspod CreateRecord \
  --Domain example.com \
  --RecordType A \
  --SubDomain www \
  --RecordLine 默认 \
  --Value 1.2.3.4 \
  --TTL 600

# 添加 CNAME 记录
tencentcloud dnspod CreateRecord \
  --Domain example.com \
  --RecordType CNAME \
  --SubDomain www \
  --RecordLine 默认 \
  --Value www.example.com.cdn.dnsv1.com \
  --TTL 600
```

### 4.3 DNS 预解析

#### 配置预解析
```bash
# 配置 DNS 预解析（智能 DNS）
aliyun dns SetDomainRecordStatus \
  --RecordId 123456 \
  --Status Enable \
  --Line hz_telecom \
  --Value 1.2.3.4

# 不同线路返回不同 IP
# 电信线路 → 1.2.3.4
# 联通线路 → 5.6.7.8
# 移动线路 → 9.10.11.12
```

#### 智能 DNS 架构
```
用户请求 DNS
   ↓
智能 DNS 判断
   ├─ 电信用户 → 1.2.3.4（电信 IP）
   ├─ 联通用户 → 5.6.7.8（联通 IP）
   ├─ 移动用户 → 9.10.11.12（移动 IP）
   └─ 海外用户 → 13.14.15.16（海外 IP）
```

### 4.4 DNS 安全

#### 开启 DNSSEC
```bash
# 开启 DNSSEC（域名系统安全扩展）
aliyun dns SetDnssec \
  --DomainName example.com \
  --Status Enabled
```

#### 开启 DDoS 防护
```bash
# 开启 DDoS 防护
aliyun dns AddDomainDdosPolicy \
  --DomainName example.com \
  --Policy Basic
```

## 五、证书服务

### 5.1 SSL/TLS 证书

#### 证书类型

| 类型 | 验证方式 | 适用场景 | 有效期 |
|------|---------|---------|--------|
| **DV** | 域名验证 | 个人网站、测试环境 | 1 年 |
| **OV** | 组织验证 | 企业网站、电商平台 | 1-2 年 |
| **EV** | 扩展验证 | 金融、支付网站 | 1-2 年 |

#### 阿里云 SSL 证书

##### 免费证书（Let's Encrypt）
```bash
# 申请免费证书
aliyun cas CreateUserCertificateRequest \
  --Domain www.example.com \
  --ProductCode free \
  --SanType 1

# 下载证书
aliyun cas GetUserCertificateDetail \
  --UserCertificateId cert-xxxxx
```

##### 付费证书（CA 机构）
```bash
# 申请付费证书
aliyun cas CreateCertificateRequest \
  --Domain www.example.com \
  --ProductCode digicert-ov \
  --SanType 1 \
  --Years 1

# 上传 CSR
aliyun cas UploadUserCertificate \
  --CSR "-----BEGIN CERTIFICATE REQUEST-----\n..." \
  --CertId cert-xxxxx
```

#### 腾讯云 SSL 证书

##### 申请免费证书
```bash
# 申请免费证书
tencentcloud ssl CreateCertificate \
  --DomainType 1 \
  --Domain www.example.com \
  --CertType 1 \
  --ValidPeriod 12
```

##### 上传证书
```bash
# 上传已有证书
tencentcloud ssl UploadCertificate \
  --CertificatePublicKey "-----BEGIN CERTIFICATE-----\n..." \
  --CertificatePrivateKey "-----BEGIN PRIVATE KEY-----\n..." \
  --Repeatable false \
  --Alias my-cert
```

### 5.2 证书部署

#### 部署到 CDN
```bash
# 阿里云 CDN 部署证书
aliyun cdn SetCdnDomainSSLCertificate \
  --DomainName www.example.com \
  --CertType upload \
  --CertName my-cert \
  --SSLPublic "$(cat cert.pem)" \
  --SSLPrivateKey "$(cat key.pem)"
```

#### 部署到负载均衡
```bash
# 阿里云 SLB 部署证书
aliyun slb SetLoadBalancerHTTPSListenerAttribute \
  --LoadBalancerId lb-xxxxx \
  --ListenerPort 443 \
  --ListenerProtocol https \
  --ServerCertificateId cert-xxxxx \
  --CipherPolicy https_cipher_policy_1_2

# 上传证书
aliyun slb UploadServerCertificate \
  --ServerCertificateName my-cert \
  --ServerCertificate "$(cat cert.pem)" \
  --PrivateKey "$(cat key.pem)"
```

### 5.3 证书自动续期

#### Certbot 自动续期
```bash
# 安装 Certbot
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto

# 申请证书
./certbot-auto certonly --webroot -w /var/www/html -d www.example.com

# 自动续期（添加到 crontab）
0 0,12 * * * /path/to/certbot-auto renew --quiet
```

#### Let's Encrypt 证书
```bash
# 使用 acme.sh
curl https://get.acme.sh | sh

# 申请证书
acme.sh --issue -d www.example.com -w /var/www/html

# 自动续期（默认）
acme.sh 会自动续期，无需配置
```

### 5.4 证书监控

#### 证书过期监控
```bash
# 检查证书过期时间
echo | openssl s_client -connect www.example.com:443 2>/dev/null | \
  openssl x509 -noout -dates

# 监控证书过期（Python）
import ssl
import socket
from datetime import datetime

def check_cert_expiry(domain):
    context = ssl.create_default_context()
    with socket.create_connection((domain, 443)) as sock:
        with context.wrap_socket(sock, server_hostname=domain) as ssock:
            cert = ssock.getpeercert()
            expiry_date = datetime.strptime(cert['notAfter'], '%b %d %H:%M:%S %Y %Z')
            days_left = (expiry_date - datetime.now()).days
            return days_left

# 使用
days = check_cert_expiry('www.example.com')
if days < 30:
    print(f"警告：证书将在 {days} 天后过期")
else:
    print(f"证书正常，剩余 {days} 天")
```

## 六、CDN 与域名实战

### 6.1 完整配置流程

#### 步骤 1：域名注册
```bash
# 1. 注册域名
aliyun domain SaveSingleTaskForCreatingOrderRenew \
  --DomainName example.com \
  --RegistrantProfileId 123456 \
  --SubscriptionDuration 1
```

#### 步骤 2：DNS 解析
```bash
# 2. 添加域名到 DNS
aliyun dns AddDomain \
  --DomainName example.com

# 3. 添加解析记录
aliyun dns AddDomainRecord \
  --DomainName example.com \
  --RR www \
  --Type CNAME \
  --Value www.example.com.cdn.aliyuncs.com \
  --TTL 600
```

#### 步骤 3：CDN 配置
```bash
# 4. 开通 CDN
aliyun cdn AddCdnDomain \
  --DomainName www.example.com \
  --CdnType web \
  --SourceType oss \
  --SourceIps example.oss-cn-hangzhou.aliyuncs.com
```

#### 步骤 4：证书部署
```bash
# 5. 申请证书
aliyun cas CreateUserCertificateRequest \
  --Domain www.example.com \
  --ProductCode free

# 6. 部署证书到 CDN
aliyun cdn SetCdnDomainSSLCertificate \
  --DomainName www.example.com \
  --CertType upload \
  --SSLPublic "$(cat cert.pem)" \
  --SSLPrivateKey "$(cat key.pem)"
```

#### 步骤 5：验证
```bash
# 7. 验证 DNS 解析
nslookup www.example.com

# 8. 验证 HTTPS
curl -I https://www.example.com

# 9. 查看 CDN 命中率
aliyun cdn DescribeDomainHitRateData \
  --DomainName www.example.com \
  --StartTime $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
  --EndTime $(date -u +%Y-%m-%dT%H:%M:%SZ)
```

### 6.2 最佳实践

#### 缓存策略优化
```json
{
  "cache_rules": [
    {
      "path": "/static/*",
      "ttl": 86400,
      "ignore_params": true
    },
    {
      "path": "/api/*",
      "ttl": 0,
      "ignore_params": false
    },
    {
      "path": "/*.js",
      "ttl": 3600,
      "ignore_params": true
    },
    {
      "path": "/*.css",
      "ttl": 3600,
      "ignore_params": true
    }
  ]
}
```

#### HTTPS 优化
```bash
# 启用 HTTP/2
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "http2"
    }
  ]'

# 启用 HSTS
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"},
        {"argName": "max_age", "argValue": "31536000"}
      ],
      "functionName": "hsts"
    }
  ]'
```

#### 安全防护
```bash
# 开启 WAF
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "waf"
    }
  ]'

# 开启 IP 黑白名单
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "type", "argValue": "black_list"},
        {"argName": "value", "argValue": "1.2.3.4,5.6.7.8"}
      ],
      "functionName": "ip_auth_list"
    }
  ]'
```

## 七、成本优化

### 7.1 CDN 成本分析

| 计费方式 | 说明 | 适用场景 | 成本 |
|---------|------|---------|------|
| **按流量计费** | 根据实际流量计费 | 流量波动大 | ¥0.24/GB |
| **按带宽计费** | 根据峰值带宽计费 | 流量稳定 | ¥50/Mbps/月 |
| **阶梯计费** | 流量越大单价越低 | 大流量 | 优惠价 |

### 7.2 成本优化策略

#### 1. 提升命中率
```bash
# 优化缓存规则，提升命中率
# 目标：命中率 > 95%

aliyun cdn DescribeDomainHitRateData \
  --DomainName www.example.com

# 调整缓存策略
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "cache_content", "argValue": "/static/*"},
        {"argName": "ttl", "argValue": "86400"}
      ],
      "functionName": "cache_config"
    }
  ]'
```

#### 2. 压缩优化
```bash
# 开启 Gzip 和 Brotli 压缩
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "gzip"
    },
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "brotli"
    }
  ]'
```

#### 3. 图片优化
```bash
# 开启图片自动 WebP
aliyun cdn BatchSetCdnDomainConfig \
  --DomainName www.example.com \
  --Functions '[
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "webp_auto"
    },
    {
      "functionArgs": [
        {"argName": "enable", "argValue": "on"}
      ],
      "functionName": "img_trimming"
    }
  ]'
```

## 八、总结

### 关键要点

1. **CDN 加速**：就近访问，降低延迟
2. **域名解析**：DNS 配置、智能解析
3. **证书管理**：SSL 证书申请、部署、续期
4. **性能优化**：缓存策略、压缩、图片优化
5. **安全防护**：HTTPS、WAF、IP 黑白名单
6. **成本优化**：提升命中率、压缩优化

### CDN ≠ 只加速

CDN 的价值：
- 加速访问
- 降低成本
- 提升可用性
- 安全防护

**记住：配置好 CDN 能让你的网站访问速度提升 10 倍！**
