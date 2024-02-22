# Secure Key Manager

## PKCS(Public Key Cryptography Standards)

공개 키 암호에 대한 사용 방식을 정의한 표준 프로토콜

- 개인 키 및 공개 키의 생성, 저장, 교환 및 사용에 대한 다양한 프로토콜 및 형식 정의

```
PKCS #1: RSA 암호화 및 서명에 대한 기본 표준.
PKCS #7: 데이터 압축, 암호화, 디지털 서명, 인증서, 인증서 체인 등을 포함하는 메시지 형식 정의.
PKCS #11: 암호화 토큰 및 하드웨어 보안 모듈(HSM)과 상호 작용하기 위한 API 정의.
PKCS #12: 개인 키, 공개 키 및 인증서와 같은 개인 보안 정보의 포맷 정의.
PKCS #15: 개인 키 저장 장치와의 상호 작용을 위한 인터페이스와 알고리즘 정의.
```

### PKCS #12

![img_8.png](resources/img_8.png)

사용자의 개인정보(사용자의 개인 키, 인증 등)의 저장과 교환을 위한 포맷

- 인증서, 개인키 내용을 파일 하나에 모두 담고 있음
- 확장자는 주로 `.pfx` 또는 `.p12` 를 가짐


## 1. Secure Key Manager

사용자의 중요 데이터를 안전하게 보관하고 접근 권한을 제어하는 서비스 (By NHN)

### 기능

![img.png](resources/img_9.png)

### 구조

![img_1.png](resources/img_1.png)

## 실습

### 1.1 - NHN Security Key Manager

![img_2.png](resources/img_2.png)

### 1.2 - 인증서 등록

![img_3.png](resources/img_3.png)

### 1.3 - 키 관리

![img_4.png](resources/img_4.png)

![img_5.png](resources/img_5.png)

```
이름, 설명 -> 별로 중요하지 않음(식별용)
아이디, 데이터 -> 매우 중요
```

데이터 필드에 암호화할 데이터를 넣어준다

> **저는 다음과 같은 정보들을 넣어 주었습니다**

- 데이터베이스 접근 URL
- 데이터베이스 클래스 이름
- 데이터베이스 사용자 이름
- 데이터베이스 패스워드

### 1.4 - 인증서 관리

다운로드 시 `.p12` 파일로 다운로드 되는데, 이는 위에서 언급했던 PKCS #12 형식을 사용하는 인증서입니다.

키 값을 보려면 해당 인증서가 있는 디렉토리로 이동하여 다음 명령어를 입력하면 된다.


```
openssl pkcs12 -in ckin.p12 -nodes
```

```
...

Error outputting keys and certificates
409CD3D601000000:error:0308010C:digital envelope routines:inner_evp_generic_fetch:unsupported:crypto/evp/evp_fetch.c:355:Global default library context, 
Algorithm (RC2-40-CBC : 0), Properties ()
```

![img.png](resources/img.png)

<br>


```
openssl pkcs12 -in ckin.p12 -nodes -legacy
```

```
Bag Attributes
    friendlyName: kmsclient
    localKeyID: 54 69 6D 65 20 31 37 30 38 33 32 35 35 32 33 30 35 30
Key Attributes: <No Attributes>
-----BEGIN PRIVATE KEY-----
MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQCDPwHiVDt/lHkQ
+CjDI9y4BlFeg88XYmKD1GTavzMSrP1GIppZLkiEAAxmCH9p4+EOKt2/icVlHOkx
mJF5TfY3/4Gf/pOOfHcP1oiGtWNp8noS13DtRdihaPA0/BlhW4Wjv87et0U9o+cl
GTv7HHdMiB5C4HtAQcYz8wUz+wtpWYJGcV8A0rgbVV6GrPQYnMa8ihbswh1Ry1xI
aUxATblhNrFWWtii1gmHbiP+LaF02gPI+aAMSex/kNtv+LXRM/uoENOy10sMYibY
p7xb3lRjxDS9BxOzshZA1h0pfAl2gYTzt+d7TTgpj2Ms/MbCqrWGmdHKGGnYNlPG
Wpn+ZcknAgMBAAECggEAGl3YFQ3y/ghn4f7YwYs81kN+qUw1mG8Nd4p90GxE6GWU
ZDijZXMk4k/YLc9bOXysQ9pcvhQBcfTC7FHIN+RvqIROGERehGuAx8/DkbxzAl/S
ImLhrukvQg4A48QZnTNzv5fTCW8sG9wHna5aHHJ1jQxzd9/9IsWT+e/ChdTTUBB/
rZbP4jmvMk6JTtCOqRfSCgQ/asDek+7Aa0bM7kEhanxCbllPi3IWUDpgh6GdFM7G
LhLQfXM0fc2CRKX5JME3yca4mlp9VviiMXgCtfIMbE65yeQ8R4pT6z4cp2gheX69
1rcs0CNFyKhPZnAMw+a2MHTpE3Qy8B06AHXSDTwdIQKBgQC74+98g1XiqQv8pzKl
prOlwDxPTBIAWfiqQ0dXUp8Ku254dIaA3YbGhXYlvGwzIfXiO3Uwq6uzVb06raUD
9H8nu7eggAgpdDZOJGdqWssihQk81t7XkLs9f3Rd0BFROhPPeeYHAwwUzQg+uPTE
oaNaEa5Fe0925ois4+IpxnpgyQKBgQCy0ojHNaGENaBh4rVFWhoRH1Yc48Ol4JS9
aeggTpcVZbNwK0tDuot3EplxDeBLUP0KAlFTQZdhUK0CwQADAeBVPtvlNQElsCps
g/y6uwXZWpr2LQyy56/i89GrVNAVa0jRLLlY/0WmkjELC3jWiVYuertf9+JfyR1p
9AMuUlJCbwKBgHqc9Nw8mP+hccEfG14eu32fRbw52EWCcVQrM5M4x7EpP5477MqL
xjgnQELAA5wavWs1gfSREyuk+CgiyV0bYR980QhchPcIb6fu+c3eElIknzKkxifG
u2oWksnlzmdDj0oGMqbAxlGMu6Sy7oijMwbEIwYUwnbFWsv4qs64R4SBAoGAILRk
k5p5acatCHoAESkDb3I9u5bkZ2gFjAbcj1wQQbeR88fXQhHSA/sA2S5igSPUw4nc
+c8bMo8gtUo7CFHFh5GxgeAJz6zCdYpGJJij3za09KUUl3vMLUPOavld/Szb6nW1
iFNXl7lRAuYcOZBwlJuZvVmGq2IK4R/hnQ0tQj0CgYAXLNqZJy/TL7lYVznOZIYj
H49FuAc+qz0fE/OV3dL93IWAG0rmaURyjDg6LKX6q01qWinggd/ip4PPeTiruDMh
5PrJLNl//go4kYP7DQnPjlSQt7epcoHYHJpKn+PnTOJsY5eRuOXddxoXqC98ptzt
BR1a4ScN5naY8MpcAyZH5A==
-----END PRIVATE KEY-----
Bag Attributes
    friendlyName: kmsclient
    localKeyID: 54 69 6D 65 20 31 37 30 38 33 32 35 35 32 33 30 35 30
subject=C=KR, O=NHN, OU=TOAST, CN=Secure Key Manager Client
issuer=C=KR, O=NHN CLOUD, OU=NHN CLOUD, CN=Secure Key Manager
-----BEGIN CERTIFICATE-----
MIIDLzCCAhegAwIBAgIEKml9JTANBgkqhkiG9w0BAQsFADBSMQswCQYDVQQGEwJL
UjESMBAGA1UECgwJTkhOIENMT1VEMRIwEAYDVQQLDAlOSE4gQ0xPVUQxGzAZBgNV
BAMMElNlY3VyZSBLZXkgTWFuYWdlcjAeFw0yNDAyMTkwNjUyMDNaFw0yNDA0MTkw
NjUyMDNaME8xCzAJBgNVBAYTAktSMQwwCgYDVQQKEwNOSE4xDjAMBgNVBAsTBVRP
QVNUMSIwIAYDVQQDExlTZWN1cmUgS2V5IE1hbmFnZXIgQ2xpZW50MIIBIjANBgkq
hkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAgz8B4lQ7f5R5EPgowyPcuAZRXoPPF2Ji
g9Rk2r8zEqz9RiKaWS5IhAAMZgh/aePhDirdv4nFZRzpMZiReU32N/+Bn/6Tjnx3
D9aIhrVjafJ6Etdw7UXYoWjwNPwZYVuFo7/O3rdFPaPnJRk7+xx3TIgeQuB7QEHG
M/MFM/sLaVmCRnFfANK4G1Vehqz0GJzGvIoW7MIdUctcSGlMQE25YTaxVlrYotYJ
h24j/i2hdNoDyPmgDEnsf5Dbb/i10TP7qBDTstdLDGIm2Ke8W95UY8Q0vQcTs7IW
QNYdKXwJdoGE87fne004KY9jLPzGwqq1hpnRyhhp2DZTxlqZ/mXJJwIDAQABoxAw
DjAMBgNVHRMEBTADAQH/MA0GCSqGSIb3DQEBCwUAA4IBAQCJlEbGRlGlzdke8t9Y
VePG/H4DGpNr709P/zdxcKDuhhbjCwDH4wxdsG4xczgUA5VECR6CCSuu6jB0l2iO
dTAsbXo2aRF90lE9ohQMbCC5tNfY/F+sMnVFVzpOprWITeMdxJc2bBTPcBmCGWH9
fYLNdZV7d8VEpO21zSk3Xp3ec8Uj677LGUZYPZCbRuvZg1juCBSKGYyYzp5w0XDZ
ZlStuTA8sj/aTGdkl/NV7nycdAUhPIwVzVfpyGxN51kG023QH0wS+pYs71qlgw6U
qDdZmea+5PEAUuKkBTmr74oTw2fpCw9hz4hJwhMnh1SkTIkwi1QHduukfralJAMN
WO+o
-----END CERTIFICATE-----
```

## 2 - DTO 클래스 및 로직 설정

### 2.1 - DTO 클래스

```java
@Getter
public class KeyResponseDto {
    private Header header;
    private Body body;

    @Getter
    @NoArgsConstructor
    public static class Body {
        private String secret;
    }
 
    @Getter
    @NoArgsConstructor
    public static class Header {
        private Integer resultCode;
        private String resultMessage;
        private boolean isSuccessful;
    }
}
```

### 2.2 - API 요청

![img_6.png](resources/img_6.png)

### 2.3 - KeyManager

암호화된 `keyId` 와 인증서를 사용하여 실제 데이터를 가져와서 `return`

```java
@Component
@RequiredArgsConstructor
public class KeyManager {
    private final KeyManagerProperties properties;

    /**
     * /resources/ckin.p12 인증서를 사용하여 클라우드에서 기밀 데이터를 가져오는 메서드
     *
     * @param keyId 조회를 원하는 데이터의 Key value
     * @return key value에 해당하는 데이터
     */
    public String keyStore(String keyId) {
        try {
            // TLS 프로토콜을 사용한 HTTPS 설정
            KeyStore clientStore = KeyStore.getInstance("PKCS12");
            InputStream result = new ClassPathResource("ckin.p12").getInputStream();
            clientStore.load(result, properties.getPassword().toCharArray());

            SSLContext sslContext = SSLContextBuilder.create()
                    .setProtocol("TLS")
                    .loadKeyMaterial(clientStore, properties.getPassword().toCharArray())
                    .loadTrustMaterial(new TrustSelfSignedStrategy())
                    .build();

            SSLConnectionSocketFactory sslConnectionSocketFactory =
                    new SSLConnectionSocketFactory(sslContext);
            CloseableHttpClient httpClient = HttpClients.custom()
                    .setSSLSocketFactory(sslConnectionSocketFactory)
                    .build();

            HttpComponentsClientHttpRequestFactory requestFactory =
                    new HttpComponentsClientHttpRequestFactory(httpClient);

            // 아래는 기존 HTTP 통신에 사용하던 코드들
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            headers.setAccept(List.of(MediaType.APPLICATION_JSON));

            RestTemplate restTemplate = new RestTemplate(requestFactory);

            URI uri = UriComponentsBuilder
                    .fromUriString(properties.getUrl())
                    .path(properties.getPath())
                    .encode()
                    .build()
                    .expand(properties.getAppKey(), keyId)
                    .toUri();
            return Objects.requireNonNull(restTemplate.exchange(uri,
                                    HttpMethod.GET,
                                    new HttpEntity<>(headers),
                                    KeyResponseDto.class)
                            .getBody())
                    .getBody()
                    .getSecret();
        } catch (KeyStoreException | IOException | CertificateException
                 | NoSuchAlgorithmException
                 | UnrecoverableKeyException
                 | KeyManagementException e) {
            throw new KeyMangerException(e.getMessage());
        }
    }
}
```

### 2.4 - 저장된 키 사용

Properties 파일에는 기존에 평문으로 보여지던 값들을 암호화된 값으로 대체할 수 있음

![img_7.png](resources/img_7.png)

`dbProperties.get~~()` 를 통하여 위에 해당하는 `keyId` 값을 가져오고

이를 통하여 `keyManager.keyStore(String keyId)` 를 호출하여 실제 값을 가져와서 `set` 해줄 수 있음

```java
@Slf4j
@Configuration
@RequiredArgsConstructor
public class DataSourceConfig {

    private final DbProperties dbProperties;
    private final KeyManager keyManager;

    @Bean
    public DataSource dataSource() {
        BasicDataSource basicDataSource = new BasicDataSource();

        basicDataSource.setDriverClassName(keyManager.keyStore(dbProperties.getDriver()));
        basicDataSource.setUrl(keyManager.keyStore(dbProperties.getUrl()));
        basicDataSource.setUsername(keyManager.keyStore(dbProperties.getUsername()));
        basicDataSource.setPassword(keyManager.keyStore(dbProperties.getPassword()));

        basicDataSource.setInitialSize(dbProperties.getInitialSize());
        basicDataSource.setMaxTotal(dbProperties.getMaxTotal());
        basicDataSource.setMaxIdle(dbProperties.getMaxIdle());
        basicDataSource.setMinIdle(dbProperties.getMinIdle());

        basicDataSource.setTestOnBorrow(true);
        basicDataSource.setValidationQuery("SELECT 1");

        basicDataSource.setMaxWaitMillis(dbProperties.getMaxWait());
        return basicDataSource;
    }
}
```
## 정리

애플리케이션을 실행하는 서버가 증가하면 데이터베이스 접속 정보를 저장하는 서버가 증가함, 접속 정보가 노출될 위험 증가

데이터베이스 접속 정보를 변경하면, 설정을 수정한 후 전체 서버에 재배포 해야하는 불편함

> 데이터베이스 접속 정보를 중앙 집중적으로 안전하게 관리
>  - 서비스 시작 시, Secure Key Manager 에서 접속 정보를 가져와서 사용








## Reference

https://docs.nhncloud.com/ko/Security/Secure%20Key%20Manager/ko/overview/
