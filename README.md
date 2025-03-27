获取认证 Token
bash
复制
编辑
curl -k -X POST https://<venafi_url>/vedauth/authorize/oauth \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'username=你的用户名&password=你的密码&client_id=vcert'
返回会包含一个 access_token，之后的请求都带这个 token。

2️⃣ 发起 Renew 请求
你需要提供证书的 CertificateId（或 GUID） 或者 Policy DN。

bash
复制
编辑
curl -k -X POST https://<venafi_url>/vedsdk/Certificates/Renew \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "CertificateDN": "\\VED\\Policy\\Your\\Cert\\Path",
    "ReuseCSR": true
  }'
字段说明：

CertificateDN：证书在 Venafi 中的完整路径
ReuseCSR：是否重用之前的 CSR，true 则使用原始私钥/CSR
如果你有 GUID，也可以用 CertificateId：

json
复制
编辑
{
  "CertificateId": "xxxx-xxxx-xxxx-xxxx",
  "ReuseCSR": true
}
3️⃣ 可选：轮询证书状态
bash
复制
编辑
curl -k -X GET "https://<venafi_url>/vedsdk/Certificates/Retrieve?CertificateDN=\\VED\\Policy\\Your\\Cert\\Path&Format=Base64" \
  -H "Authorization: Bearer <access_token>"
