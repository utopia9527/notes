# 一、基本概念

AWS Signature Version 4 是一种用于对 AWS 请求进行身份验证的标准方式。在应用层面上，通常通过 HTTP 请求头中的 `Authorization` 字段来实现。下面是其格式及一个示例。

官方文档： https://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html

# 二、Authrization Header 格式

- 当使用 AWS Signature Version 4 时，`Authorization` 头的格式如下


```shell
# 一、标准格式
Authorization: AWS4-HMAC-SHA256 Credential=<AccessKeyId>/<CredentialScope>, SignedHeaders=<SignedHeaders>, Signature=<Signature>
```

## 2.1 Credential 生成

- **AccessKeyId**：您的 AWS 访问密钥 ID。
- **CredentialScope**：<font color=yellow>包括日期（YYYYMMDD）、区域、服务和终端标识符（例如，`aws4_request`是终端提供的标识）</font>

## 2.2 SignedHeaders 生成

- `SignedHeaders` **是用于计算签名的请求标头名称列表。这些名称必须小写，并以分号分隔**。简单来说就是将所有需要的请求头用分号分隔拼接，有如下的必须的请求头

  ```shell
  # 1、Host：必需。包含主机名和端口号（如果非标准端口），如 example.amazonaws.com。这通常是所有 HTTP 请求都需要的基本头
  
  # 2、X-Amz-Date：必需。表示请求的时间戳，以 ISO8601 格式提供，如 20231104T120000Z。如果不使用该头，则可以在查询字符串中提供一个 X-Amz-Date 参数
  
  # 3、X-Amz-Content-Sha256：通常需要。用于传递请求正文的 SHA256 哈希值。这在传输数据时尤为重要，因为它确保内容在传输途中未被篡改(对于某些服务，特别是对 PUT 和 POST 方法，需要包括请求体的 SHA256 哈希。在无请求主体时，可以使用空字符串的哈希值)
  
  
  ## 最终得到的结构就是
  # 'host;x-amz-content-sha256;x-amz-date'
  ```

## 2.3 Signature

- 生成的签名会被附加到 HTTP 请求中，通常放在 `Authorization` 头的 `Signature` 字段中。这确保了请求的完整性和来源可验证

  所有字符串在计算 HMAC 和 SHA256 时，都应该使用二进制格式进行处理。

- 时间戳和区域信息需要准确无误，以确保签名匹配成功。

- 使用 AWS SDK 可以自动处理这些细节，因此推荐在生产环境中尽量使用 SDK 而不是手动实施。

- 签名密钥是通过一系列 HMAC SHA256 运算生成的 ,签名主要包括以下几个步骤

### 2.3.1、创建 Canonical Request

- Canonical Request 是对请求进行标准化处理后的字符串，格式如下：

```shell
HTTPRequestMethod
CanonicalURI
CanonicalQueryString
CanonicalHeaders
SignedHeaders
HashedPayload

# 字段含义
HTTPRequestMethod：HTTP 方法（如 GET、POST 等）。
CanonicalURI：请求 URI 的标准化版本。
CanonicalQueryString：查询字符串参数按字母顺序排序，并进行 URL 编码。
CanonicalHeaders：请求头以小写排序，每个头之间用换行符分隔。
SignedHeaders：参与签名的请求头，用分号分隔。
HashedPayload：请求主体的 SHA256 哈希值。如果没有请求主体，可以使用空字符串的 SHA256。
```

### 2.3.2 、创建StringToString

- String to Sign 是通过结合当前日期、Credential Scope 和 Canonical Request 的哈希来构建的。格式如下：

```shell
AWS4-HMAC-SHA256
<TimeStamp>
<CredentialScope>
<SHA256HashOfCanonicalRequest>

# 字段含义
TimeStamp：用于签名的时间戳，格式为 YYYYMMDD'T'HHMMSS'Z'。
CredentialScope：格式为 YYYYMMDD/region/service/aws4_request。
SHA256HashOfCanonicalRequest：Canonical Request 的 SHA256 哈希值。
```

### 2.3.3 计算签名密钥

- 签名密钥是通过一系列 HMAC SHA256 运算生成的：

```shell
# 1、计算kDate
kDate = HMAC("AWS4" + SecretAccessKey, "YYYYMMDD")
# 2、计算 kRegion
kRegion = HMAC(kDate, "region")
# 3、计算 kService：
kService = HMAC(kRegion, "service")
# 4、计算 kSigning
kSigning = HMAC(kService, "aws4_request")
# 5、计算最终的签名


```

### 2.3.4 计算最终签名

- 使用生成的签名密钥对 String to Sign 进行 HMAC SHA256 运算：

```shell
Signature = HMAC(kSigning, StringToSign)
```



# 三、Demo示例

- 以下是一个构造 HTTP 请求以调用 AWS 服务的完整示例，包括使用 AWS Signature Version 4 进行身份验证。


#### 假设场景

- 您想要获取 S3 中某个存储桶的对象列表。
- 您的区域是 `us-east-1`。
- AWS 访问密钥 ID 是 `AKIDEXAMPLE`。
- AWS 秘密访问密钥是 `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`。
- 当前日期是 `20231101`（YYYYMMDD）。

#### 示例请求

假设我们想发起一个 `GET` 请求到 `https://s3.amazonaws.com/your-bucket-name`。

```
GET /your-bucket-name HTTP/1.1
Host: s3.amazonaws.com
x-amz-date: 20231101T120000Z
Authorization: AWS4-HMAC-SHA256 Credential=AKIDEXAMPLE/20231101/us-east-1/s3/aws4_request, SignedHeaders=host;x-amz-date, Signature=fe5f80f77d5fa3beca038a248ff027d0445342fe2855ddc963176630326f1024
```

#### 生成过程解析

1. **Canonical Request**:

   ```
   GET
   /your-bucket-name
   (空查询字符串)
   host:s3.amazonaws.com
   x-amz-date:20231101T120000Z
   
   host;x-amz-date
   e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855   # SHA256 of an empty payload
   ```

2. **String to Sign**:

   ```
   plaintext复制代码AWS4-HMAC-SHA256
   20231101T120000Z
   20231101/us-east-1/s3/aws4_request
   e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
   ```

3. **计算签名密钥**:

   - `DateKey = HMAC("AWS4" + SecretAccessKey, "20231101")`
   - `DateRegionKey = HMAC(DateKey, "us-east-1")`
   - `DateRegionServiceKey = HMAC(DateRegionKey, "s3")`
   - `SigningKey = HMAC(DateRegionServiceKey, "aws4_request")`

4. **计算签名**:

   - 使用 `SigningKey` 对 `String to Sign` 进行 HMAC SHA256 运算得到 `Signature`。

5. **组合 Authorization Header**:

   - 将所有部分拼接成最终的 `Authorization` 头。

请注意，这仅是一个演示，真实的签名需要确保时间戳准确以及所有域值都经过正确编码。同时，生产环境中应使用 AWS SDK 自动处理这些细节，而不是手动实施。AWS 提供的 SDK 会自动处理签名的生成过程。

# 五、验证头C++ 代码

```cpp
// Helper functions for encoding and hashing
std::string hmacSha256(const std::string &key, const std::string &message) {
  unsigned char hash[EVP_MAX_MD_SIZE];
  unsigned int len = 0;
  HMAC(EVP_sha256(), key.data(), key.size(),
       reinterpret_cast<const unsigned char *>(message.data()), message.size(),
       hash, &len);
  std::ostringstream oss;
  for (unsigned int i = 0; i < len; i++) {
    oss << std::hex << std::setw(2) << std::setfill('0') << (int)hash[i];
  }
  return oss.str();
}

std::string sha256(const std::string &input) {
  unsigned char hash[SHA256_DIGEST_LENGTH];
  SHA256(reinterpret_cast<const unsigned char *>(input.c_str()), input.size(),
         hash);
  std::ostringstream oss;
  for (int i = 0; i < SHA256_DIGEST_LENGTH; i++) {
    oss << std::hex << std::setw(2) << std::setfill('0') << (int)hash[i];
  }
  return oss.str();
}

std::string urlEncode(const std::string &value) {
  std::ostringstream escaped;
  escaped.fill('0');
  escaped << std::hex;
  for (char c : value) {
    if (isalnum((unsigned char)c) || c == '-' || c == '_' || c == '.' ||
        c == '~') {
      escaped << c;
    } else {
      escaped << '%' << std::setw(2) << int((unsigned char)c);
    }
  }
  return escaped.str();
}

// Generate the signing key
std::string getSignatureKey(const std::string &secretKey,
                            const std::string &dateStamp,
                            const std::string &regionName,
                            const std::string &serviceName) {
  std::string kDate = hmacSha256("AWS4" + secretKey, dateStamp);
  std::string kRegion = hmacSha256(kDate, regionName);
  std::string kService = hmacSha256(kRegion, serviceName);
  return hmacSha256(kService, "aws4_request");
}

// Generate the canonical request
std::string createCanonicalRequest(const std::string &httpMethod,
                                   const std::string &canonicalUri,
                                   const std::string &canonicalQuerystring,
                                   const std::string &canonicalHeaders,
                                   const std::string &signedHeaders,
                                   const std::string &payloadHash) {
  std::ostringstream oss;
  oss << httpMethod << "\n"
      << canonicalUri << "\n"
      << canonicalQuerystring << "\n"
      << canonicalHeaders << "\n"
      << signedHeaders << "\n"
      << payloadHash;
  return oss.str();
}

// Generate the string to sign
std::string createStringToSign(const std::string &canonicalRequest,
                               const std::string &timeStamp,
                               const std::string &scope) {
  std::string hashCanonicalRequest = sha256(canonicalRequest);
  std::ostringstream oss;
  oss << "AWS4-HMAC-SHA256\n"
      << timeStamp << "\n"
      << scope << "\n"
      << hashCanonicalRequest;
  return oss.str();
}

// Generate the authorization header
std::string generateAuthorizationHeader(
    const std::string &accessKey, const std::string &secretKey,
    const std::string &region, const std::string &service,
    const std::string &httpMethod, const std::string &uri,
    const std::map<std::string, std::string> &queryParams,
    const std::map<std::string, std::string> &headers,
    const std::string &payload) {
  // 1. Canonical Request
  std::string canonicalUri = uri;  // 假定直接传入了标准化 URI
  std::ostringstream queryStringStream;
  for (const auto &[key, value] : queryParams) {
    queryStringStream << urlEncode(key) << "=" << urlEncode(value) << "&";
  }
  std::string canonicalQuerystring = queryStringStream.str();
  if (!canonicalQuerystring.empty())
    canonicalQuerystring.pop_back();  // 移除末尾的 `&`

  // Headers 排序
  std::ostringstream canonicalHeadersStream, signedHeadersStream;
  for (const auto &[key, value] : headers) {
    canonicalHeadersStream << key << ":" << value << "\n";
    signedHeadersStream << key << ";";
  }
  std::string canonicalHeaders = canonicalHeadersStream.str();
  std::string signedHeaders = signedHeadersStream.str();
  if (!signedHeaders.empty()) signedHeaders.pop_back();  // 移除末尾的 `;`

  std::string payloadHash = sha256(payload);

  std::string canonicalRequest =
      createCanonicalRequest(httpMethod, canonicalUri, canonicalQuerystring,
                             canonicalHeaders, signedHeaders, payloadHash);

  // 2. String to Sign
  std::string timeStamp = headers.at("x-amz-date");
  std::string dateStamp = timeStamp.substr(0, 8);
  std::string scope =
      dateStamp + "/" + region + "/" + service + "/aws4_request";
  std::string stringToSign =
      createStringToSign(timeStamp, scope, canonicalRequest);

  // 3. Signature
  std::string signingKey =
      getSignatureKey(secretKey, dateStamp, region, service);
  std::string signature = hmacSha256(signingKey, stringToSign);

  // 4. Authorization Header
  std::ostringstream authHeaderStream;
  authHeaderStream << "AWS4-HMAC-SHA256 Credential=" << accessKey << "/"
                   << scope << ", SignedHeaders=" << signedHeaders
                   << ", Signature=" << signature;

  return authHeaderStream.str();
}

std::string getAmzDate() {
  char buffer[17];
  std::time_t now = std::time(nullptr);
  std::strftime(buffer, sizeof(buffer), "%Y%m%dT%H%M%SZ", std::gmtime(&now));
  return std::string(buffer);
```

