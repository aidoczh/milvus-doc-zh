---
id: tls.md
title: Encryption in Transit
summary: Learn how to enable TLS proxy in Milvus.
---

# 在传输中的加密

TLS（传输层安全）是一种加密协议，用于确保通信安全。Milvus代理使用TLS单向和双向认证。

本主题描述了如何在Milvus中启用TLS代理。

<div class="alert note">

TLS和用户认证是两种不同的安全方法。如果在Milvus系统中同时启用了用户认证和TLS，您将需要提供用户名、密码和证书文件路径。有关如何启用用户认证的信息，请参考[用户认证](authenticate.md)。

</div>

## 创建您自己的证书

### 先决条件

确保已安装OpenSSL。如果尚未安装，请先[构建和安装](https://github.com/openssl/openssl/blob/master/INSTALL.md) OpenSSL。

```shell
openssl version
```

如果未安装OpenSSL，可以使用以下命令在Ubuntu中安装。

```shell
sudo apt install openssl
```

### 创建文件

1. 创建 `openssl.cnf` 和 `gen.sh` 文件。

```
mkdir cert && cd cert
touch openssl.cnf gen.sh
```

2. 分别将以下配置复制到相应的文件中。

<details><summary><code>openssl.cnf</code></summary>

```ini
#
# OpenSSL example configuration file.
# This is mostly being used for generation of certificate requests.
#

# This definition stops the following lines choking if HOME isn't
# defined.
HOME= .
RANDFILE= $ENV::HOME/.rnd

# Extra OBJECT IDENTIFIER info:
#oid_file= $ENV::HOME/.oid
oid_section= new_oids

# To use this configuration file with the "-extfile" option of the
# "openssl x509" utility, name here the section containing the
# X.509v3 extensions to use:
# extensions= 
# (Alternatively, use a configuration file that has only
# X.509v3 extensions in its main [= default] section.)

[ new_oids ]

# We can add new OIDs in here for use by 'ca', 'req' and 'ts'.
# Add a simple OID like this:
# testoid1=1.2.3.4
# Or use config file substitution like this:
# testoid2=${testoid1}.5.6

# Policies used by the TSA examples.
tsa_policy1 = 1.2.3.4.1
tsa_policy2 = 1.2.3.4.5.6
tsa_policy3 = 1.2.3.4.5.7

####################################################################
[ ca ]
default_ca= CA_default# The default ca section

####################################################################
[ CA_default ]

dir= ./demoCA# Where everything is kept
certs= $dir/certs# Where the issued certs are kept
crl_dir= $dir/crl# Where the issued crl are kept
database= $dir/index.txt# database index file.
#unique_subject= no# Set to 'no' to allow creation of
# several ctificates with same subject.
new_certs_dir= $dir/newcerts# default place for new certs.

certificate= $dir/cacert.pem # The CA certificate
serial= $dir/serial # The current serial number
crlnumber= $dir/crlnumber# the current crl number
# must be commented out to leave a V1 CRL
crl= $dir/crl.pem # The current CRL
private_key= $dir/private/cakey.pem# The private key
RANDFILE= $dir/private/.rand# private random number file

x509_extensions= usr_cert# The extentions to add to the cert

# Comment out the following two lines for the "traditional"
# (and highly broken) format.
name_opt = ca_default# Subject Name options
cert_opt = ca_default# Certificate field options

# Extension copying option: use with caution.
copy_extensions = copy

# Extensions to add to a CRL. Note: Netscape communicator chokes on V2 CRLs
# so this is commented out by default to leave a V1 CRL.
# crlnumber must also be commented out to leave a V1 CRL.
# crl_extensions= crl_ext

default_days= 365# how long to certify for
default_crl_days= 30# how long before next CRL
default_md= default# use public key default MD
preserve= no# keep passed DN ordering

# A few difference way of specifying how similar the request should look
# For type CA, the listed attributes must be the same, and the optional
# and supplied fields are just that :-)
policy= policy_match

# For the CA policy
[ policy_match ]
countryName= match
stateOrProvinceName= match
organizationName= match
organizationalUnitName= optional
commonName= supplied
emailAddress= optional

# For the 'anything' policy
# At this point in time, you must list all acceptable 'object'
# types.
[ policy_anything ]
countryName= optional
stateOrProvinceName= optional
localityName= optional
organizationName= optional
organizationalUnitName= optional
commonName= supplied
emailAddress= optional

####################################################################
[ req ]
default_bits= 2048
default_keyfile = privkey.pem
distinguished_name= req_distinguished_name
attributes= req_attributes
x509_extensions= v3_ca# The extentions to add to the self signed cert

# Passwords for private keys if not present they will be prompted for
# input_password = secret
# output_password = secret

# This sets a mask for permitted string types. There are several options. 
# default: PrintableString, T61String, BMPString.
# pkix : PrintableString, BMPString (PKIX recommendation before 2004)
# utf8only: only UTF8Strings (PKIX recommendation after 2004).
# nombstr : PrintableString, T61String (no BMPStrings or UTF8Strings).
# MASK:XXXX a literal mask value.
# WARNING: ancient versions of Netscape crash on BMPStrings or UTF8Strings.
string_mask = utf8only

req_extensions = v3_req # The extensions to add to a certificate request

[ req_distinguished_name ]
countryName= Country Name (2 letter code)
countryName_default= AU
countryName_min= 2
countryName_max= 2

stateOrProvinceName= State or Province Name (full name)
stateOrProvinceName_default= Some-State

localityName= Locality Name (eg, city)

0.organizationName= Organization Name (eg, company)
0.organizationName_default= Internet Widgits Pty Ltd

# we can do this but it is not needed normally :-)
#1.organizationName= Second Organization Name (eg, company)
#1.organizationName_default= World Wide Web Pty Ltd

organizationalUnitName= Organizational Unit Name (eg, section)
#organizationalUnitName_default=

commonName= Common Name (e.g. server FQDN or YOUR name)
commonName_max= 64

emailAddress= Email Address
emailAddress_max= 64

# SET-ex3= SET extension number 3

[ req_attributes ]
challengePassword= A challenge password
challengePassword_min= 4
challengePassword_max= 20

unstructuredName= An optional company name

[ usr_cert ]

# These extensions are added when 'ca' signs a request.

# This goes against PKIX guidelines but some CAs do it and some software
# requires this to avoid interpreting an end user certificate as a CA.

basicConstraints=CA:FALSE

# Here are some examples of the usage of nsCertType. If it is omitted
# the certificate can be used for anything *except* object signing.

# This is OK for an SSL server.
# nsCertType= server

# For an object signing certificate this would be used.
# nsCertType = objsign

# For normal client use this is typical
# nsCertType = client, email

# and for everything including object signing:
# nsCertType = client, email, objsign

# This is typical in keyUsage for a client certificate.
# keyUsage = nonRepudiation, digitalSignature, keyEncipherment

# This will be displayed in Netscape's comment listbox.
nsComment= "OpenSSL Generated Certificate"

# PKIX recommendations harmless if included in all certificates.
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid,issuer

# This stuff is for subjectAltName and issuerAltname.
# Import the email address.
# subjectAltName=email:copy
# An alternative to produce certificates that aren't
# deprecated according to PKIX.
# subjectAltName=email:move

# Copy subject details
# issuerAltName=issuer:copy

#nsCaRevocationUrl= http://www.domain.dom/ca-crl.pem
#nsBaseUrl
#nsRevocationUrl
#nsRenewalUrl
#nsCaPolicyUrl
#nsSslServerName

# This is required for TSA certificates.
# extendedKeyUsage = critical,timeStamping

[ v3_req ]

# Extensions to add to a certificate request

basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment


[ v3_ca ]


# Extensions for a typical CA


# PKIX recommendation.

subjectKeyIdentifier=hash

authorityKeyIdentifier=keyid:always,issuer

# This is what PKIX recommends but some broken software chokes on critical
# extensions.
#basicConstraints = critical,CA:true
# So we do this instead.
basicConstraints = CA:true

# Key usage: this is typical for a CA certificate. However since it will
# prevent it being used as an test self-signed certificate it is best
# left out by default.
# keyUsage = cRLSign, keyCertSign

# Some might want this also
# nsCertType = sslCA, emailCA

# Include email address in subject alt name: another PKIX recommendation
# subjectAltName=email:copy
# Copy issuer details
# issuerAltName=issuer:copy

# DER hex encoding of an extension: beware experts only!
# obj=DER:02:03
# Where 'obj' is a standard or added object
# You can even override a supported extension:
# basicConstraints= critical, DER:30:03:01:01:FF

[ crl_ext ]

# CRL extensions.
# Only issuerAltName and authorityKeyIdentifier make any sense in a CRL.

# issuerAltName=issuer:copy
authorityKeyIdentifier=keyid:always

[ proxy_cert_ext ]
# These extensions should be added when creating a proxy certificate

# This goes against PKIX guidelines but some CAs do it and some software
# requires this to avoid interpreting an end user certificate as a CA.

basicConstraints=CA:FALSE

# Here are some examples of the usage of nsCertType. If it is omitted
# the certificate can be used for anything *except* object signing.

# This is OK for an SSL server.
# nsCertType= server

# For an object signing certificate this would be used.
# nsCertType = objsign

# For normal client use this is typical
# nsCertType = client, email

# and for everything including object signing:
# nsCertType = client, email, objsign

# This is typical in keyUsage for a client certificate.
# keyUsage = nonRepudiation, digitalSignature, keyEncipherment

# This will be displayed in Netscape's comment listbox.
nsComment= "OpenSSL Generated Certificate"

# PKIX recommendations harmless if included in all certificates.
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid,issuer

# This stuff is for subjectAltName and issuerAltname.
# Import the email address.
# subjectAltName=email:copy
# An alternative to produce certificates that aren't
# deprecated according to PKIX.
# subjectAltName=email:move

# Copy subject details
# issuerAltName=issuer:copy

#nsCaRevocationUrl= http://www.domain.dom/ca-crl.pem
#nsBaseUrl
#nsRevocationUrl
#nsRenewalUrl
#nsCaPolicyUrl
#nsSslServerName

# This really needs to be in place for it to be a proxy certificate.
proxyCertInfo=critical,language:id-ppl-anyLanguage,pathlen:3,policy:foo

####################################################################
[ tsa ]

default_tsa = tsa_config1# the default TSA section

[ tsa_config1 ]

# These are used by the TSA reply generation only.
dir= ./demoCA# TSA root directory
serial= $dir/tsaserial# The current serial number (mandatory)
crypto_device= builtin# OpenSSL engine to use for signing
signer_cert= $dir/tsacert.pem # The TSA signing certificate
# (optional)
certs= $dir/cacert.pem# Certificate chain to include in reply
# (optional)
signer_key= $dir/private/tsakey.pem # The TSA private key (optional)

default_policy= tsa_policy1# Policy if request did not specify it
# (optional)
other_policies= tsa_policy2, tsa_policy3# acceptable policies (optional)
digests= md5, sha1# Acceptable message digests (mandatory)
accuracy= secs:1, millisecs:500, microsecs:100# (optional)
clock_precision_digits  = 0# number of digits after dot. (optional)
ordering= yes# Is ordering defined for timestamps?
# (optional, default: no)
tsa_name= yes# Must the TSA name be included in the reply?
# (optional, default: no)
ess_cert_id_chain= no# Must the ESS cert id chain be included?
# (optional, default: no)
```
</details>

`openssl.cnf` 文件是默认的 OpenSSL 配置文件。更多信息请参阅[手册页面](https://www.openssl.org/docs/manmaster/man5/config.html)。`gen.sh` 文件用于生成相关证书文件。您可以修改 `gen.sh` 文件以实现不同的目的，比如更改证书文件的有效期、证书密钥的长度或证书文件的名称。

在 `gen.sh` 文件中配置 `CommonName` 是必要的。`CommonName` 指的是客户端在连接时应指定的服务器名称。

<details><summary><code>gen.sh</code></summary>

```shell
#!/usr/bin/env sh
# 你的变量
Country="CN"
State="上海"
Location="上海"
Organization="milvus"
Organizational="milvus"
CommonName="localhost"

echo "生成 ca.key"
openssl genrsa -out ca.key 2048

echo "生成 ca.pem"
openssl req -new -x509 -key ca.key -out ca.pem -days 3650 -subj "/C=$Country/ST=$State/L=$Location/O=$Organization/OU=$Organizational/CN=$CommonName"

echo "生成服务器 SAN 证书"
openssl genpkey -algorithm RSA -out server.key
openssl req -new -nodes -key server.key -out server.csr -days 3650 -subj "/C=$Country/O=$Organization/OU=$Organizational/CN=$CommonName" -config ./openssl.cnf -extensions v3_req
openssl x509 -req -days 3650 -in server.csr -out server.pem -CA ca.pem -CAkey ca.key -CAcreateserial -extfile ./openssl.cnf -extensions v3_req

echo "生成客户端 SAN 证书"
openssl genpkey -algorithm RSA -out client.key
openssl req -new -nodes -key client.key -out client.csr -days 3650 -subj "/C=$Country/O=$Organization/OU=$Organizational/CN=$CommonName" -config ./openssl.cnf -extensions v3_req
openssl x509 -req -days 3650 -in client.csr -out client.pem -CA ca.pem -CAkey ca.key -CAcreateserial -extfile ./openssl.cnf -extensions v3_req

```
</details>

`gen.sh` 文件中的变量对于创建证书签名请求文件至关重要。前五个变量是基本的签名信息，包括国家、州、位置、组织、组织单位。在配置 `CommonName` 时需要小心，因为在客户端与服务器通信期间将对其进行验证。

### 运行 `gen.sh` 生成证书

运行 `gen.sh` 文件以创建证书。

```
chmod +x gen.sh
./gen.sh
```

将创建以下九个文件：`ca.key`、`ca.pem`、`ca.srl`、`server.key`、`server.pem`、`server.csr`、`client.key`、`client.pem`、`client.csr`。

### 修改证书文件的详细信息（可选）

生成证书后，您可以根据自己的需求修改证书文件的详细信息。

SSL 或 TSL 互相认证的实现涉及客户端、服务器和证书颁发机构（CA）。CA 用于确保客户端和服务器之间的证书合法。

运行 `man openssl` 或参阅[openssl手册页面](https://www.openssl.org/docs/)以获取有关使用 OpenSSL 命令的更多信息。
1. 为 CA 生成 RSA 私钥。

```
openssl genpkey -algorithm RSA -out ca.key
```

2. 请求 CA 证书生成。

在这一步中，您需要提供有关 CA 的基本信息。选择 `x509` 选项以跳过请求并直接生成自签名证书。

```
openssl req -new -x509 -key ca.key -out ca.pem -days 3650 -subj "/C=$Country/ST=$State/L=$Location/O=$Organization/OU=$Organizational/CN=$CommonName"
```

在这一步之后，您将得到一个 `ca.pem` 文件，这是一个 CA 证书，可用于生成客户端-服务器证书。

3. 生成服务器私钥。

```
openssl genpkey -algorithm RSA -out server.key
```

在这一步之后，您将得到一个 `server.key` 文件。

4. 生成证书签名请求文件。

您需要提供有关服务器的必要信息以生成证书签名请求文件。

```
openssl req -new -nodes -key server.key -out server.csr -days 3650 -subj "/C=$Country/O=$Organization/OU=$Organizational/CN=$CommonName" -config ./openssl.cnf -extensions v3_req
```

在这一步之后，您将得到一个 `server.csr` 文件。

5. 签署证书。

打开 `server.csr`、`ca.key` 和 `ca.pem` 文件以签署证书。如果不存在，`CAcreateserial` 命令选项用于创建 CA 序列号文件。选择此命令选项后，您将得到一个 `aca.srl` 文件。

```
openssl x509 -req -days 3650 -in server.csr -out server.pem -CA ca.pem -CAkey ca.key -CAcreateserial -extfile ./openssl.cnf -extensions v3_req
```

## 使用 TLS 配置 Milvus 服务器

本节概述了如何配置带有 TLS 加密的 Milvus 服务器的步骤。

<div class="alert note">

由于 milvus-helm 和 milvus-operator 目前在证书文件路径配置方面存在限制，本指南将重点介绍使用 Docker Compose 进行部署。

</div>

### 1. 修改 Milvus 服务器配置

要启用 TLS，请将 `milvus.yaml` 中的 `common.security.tlsMode` 设置为 `1`（单向 TLS）或 `2`（双向 TLS）。

```yaml
tls:
  serverPemPath: /milvus/tls/server.pem
  serverKeyPath: /milvus/tls/server.key
  caPemPath: /milvus/tls/ca.pem

common:
  security:
    tlsMode: 1
```

参数：

- `serverPemPath`：服务器证书文件的路径。
- `serverKeyPath`：服务器密钥文件的路径。
- `caPemPath`：CA 证书文件的路径。
- `tlsMode`：加密的 TLS 模式。有效值：
  - `1`：单向身份验证，仅服务器需要证书，客户端验证它。此模式需要服务器端的 `server.pem` 和 `server.key`，以及客户端端的 `server.pem`。
  - `2`：双向身份验证，服务器和客户端都需要证书才能建立安全连接。此模式需要服务器端的 `server.pem`、`server.key` 和 `ca.pem`，以及客户端端的 `client.pem`、`client.key` 和 `ca.pem`。

### 2. 将证书文件映射到容器中
#### 准备证书文件

在与您的 `docker-compose.yaml` 相同的目录中创建一个名为 `tls` 的新文件夹。将 `server.pem`、`server.key` 和 `ca.pem` 复制到 `tls` 文件夹中。将它们放置在以下目录结构中：

```
├── docker-compose.yml
├── milvus.yaml
└── tls
     ├── server.pem
     ├── server.key
     └── ca.pem
```

#### 更新 Docker Compose 配置

编辑 `docker-compose.yaml` 文件，将证书文件路径映射到容器内，如下所示：

```yaml
  standalone:
    container_name: milvus-standalone
    image: milvusdb/milvus:latest
    command: ["milvus", "run", "standalone"]
    security_opt:
    - seccomp:unconfined
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
      - ${DOCKER_VOLUME_DIRECTORY:-.}/tls:/milvus/tls
      - ${DOCKER_VOLUME_DIRECTORY:-.}/milvus.yaml:/milvus/configs/milvus.yaml
```

#### 使用 Docker Compose 部署 Milvus

执行以下命令以部署 Milvus：

```bash
sudo docker compose up -d
```

## 使用 TLS 连接到 Milvus 服务器

对于 SDK 交互，请根据 TLS 模式使用以下设置。

### 单向 TLS 连接

提供 `server.pem` 的路径，并确保 `server_name` 与证书中配置的 `CommonName` 匹配。

```python
from pymilvus import MilvusClient

client = MilvusClient(
    uri="http://localhost:19530",
    secure=True,
    server_pem_path="path_to/server.pem",
    server_name="localhost"
)
```

### 双向 TLS 连接

提供 `client.pem`、`client.key` 和 `ca.pem` 的路径，并确保 `server_name` 与证书中配置的 `CommonName` 匹配。

```python
from pymilvus import MilvusClient

client = MilvusClient(
    uri="http://localhost:19530",
    secure=True,
    client_pem_path="path_to/client.pem",
    client_key_path="path_to/client.key",
    ca_pem_path="path_to/ca.pem",
    server_name="localhost"
)
```

查看 [example_tls1.py](https://github.com/milvus-io/pymilvus/blob/master/examples/example_tls1.py) 和 [example_tls2.py](https://github.com/milvus-io/pymilvus/blob/master/examples/example_tls2.py) 获取更多信息。