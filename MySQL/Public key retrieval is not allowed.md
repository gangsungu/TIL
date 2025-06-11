# [DBeaver] Public Key Retrieval is not allowed 에러

MySQL 8.0부터 기본 인증 플러그인이 변경되었다

| 버전           | 기본 인증 방식                     |
| ------------ | ---------------------------- |
| MySQL 5.x 이하 | `mysql_native_password`      |
| MySQL 8.x    | `caching_sha2_password` ✅ 기본 |

`caching_sha2_password`은 클라이언트가 서버에 로그인할 때, 비밀번호를 암호화하여 전송해야 한다.<br/>
암호화를 위해선 서버의 공개 키(public key)가 필요하다.

👉 공개키가 필요한 이유 요약
- 비밀번호 평문 전송 ❌ (보안 이슈)
- 대신 RSA 공개키로 암호화해서 서버에 보냄
- 그래서 클라이언트(DBeaver)가 서버에 공개키 달라고 요청

그런데 DBeaver에서는 allowPublicKeyRetrieval 옵션이 기본값으로 false로 되어있어 공개키를 받지못했다.

그래서 DBeaver가 공개키 요청을 허용하도록 설정하면 해결

> allowPublicKeyRetrieval=true<br/>
> useSSL=false