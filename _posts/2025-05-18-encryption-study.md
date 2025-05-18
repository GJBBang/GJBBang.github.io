---
layout: post

title: 패스워드를 어떻게 저장할 것인가 ? (단반향 암호화)

categories: java

tag: [study]


---



개인정보, 패스워드 등 민감 데이터는 DB에 저장할 때 암호화해서 저장해야 한다.<br/>
(~~특히 개인정보 때문에 정보보호팀에서 우리팀을 엄청나게 괴롭히고 있다..~~)
암호화는 크게 복호화를 할 수 있냐 없냐에 따라서 단방향, 양방향 2가지로 나눌 수 있다.
오늘은 패스워드와 같이 복호화 필요성이 없는 데이터의 암호화 방식은 단방향 암호화에 대해서 정리해보려고 한다.



### 단방향 암호화 종류



| 알고리즘    | 자바 지원 여부  | 복호화 가능 | 특징                           | 보안성 |
| ----------- | --------------- | ----------- | ------------------------------ | ------ |
| MD5         | O (표준)        | ❌           | 빠름, 충돌 있음 (취약함)       | ❌ 낮음 |
| SHA-1       | O               | ❌           | 더 안전하지만 여전히 취약      | ❌      |
| SHA-256/512 | O               | ❌           | 비교적 안전한 일반 해시 함수   | ⚠️ 중간 |
| PBKDF2      | O (Spring)      | ❌           | 반복 횟수 설정 가능, 표준 방식 | ✅ 좋음 |
| BCrypt      | O (Spring)      | ❌           | 느림 + 솔트 내장, 실무 표준    | ✅ 좋음 |
| SCrypt      | O (Spring)      | ❌           | 메모리 요구 ↑, 병렬 공격 방지  | ✅ 좋음 |
| Argon2      | O (Spring 5.0+) | ❌           | 최신 알고리즘, 가장 안전       | ✅ 최고 |



🔑 **솔트(Salt)란 ?**

> 비밀번호를 해시하기 전에 임의로 추가하는 무작위 문자열

❌ 솔트가 없는 경우 무슨 문제가 생기나 ?

1. 동일한 비밀번호 => 동일한 해시 값

- 예를 들어 2명의 사용자가 같은 비밀번호를 사용했다면

  → 두 사람의 해시값이 **완전히 똑같음** ❗

2. 레인보우 테이블 공격

- 해커가 미리 만들어둔 **"비밀번호 → 해시값" 표** (레인보우 테이블)을 이용해 해시된 비밀번호를 대입하고 역으로 원래 값을 유추할 수 있음



#### 1. 🔓 MD5, SHA-1, SHA-256 등 일반 해시 함수

 🔧 사용 예 (자바 기본 API)

```java
MessageDigest md = MessageDigest.getInstance("SHA-256");
byte[] hash = md.digest("password".getBytes(StandardCharsets.UTF_8));
String encoded = Base64.getEncoder().encodeToString(hash);
```

✅ 간단하지만 **솔트가 없음** → 보안에 취약
 ❌ 실무에서 비밀번호 저장에는 사용 ❌

------

#### 2. 🔐 **BCryptPasswordEncoder** (가장 일반적인 선택)

- 솔트가 내장되어 있음
- 느리게 설계됨 (brute-force 방지)
- 매번 다른 결과가 나옴 (동일 비밀번호라도!)


🔧 사용 예 (Spring Security)

```java
PasswordEncoder encoder = new BCryptPasswordEncoder();
String encoded = encoder.encode("password");
boolean matches = encoder.matches("password", encoded);
```

------

#### 3. 🔐 **PBKDF2 (Password-Based Key Derivation Function 2)**

- 반복 횟수를 높여 무차별 대입 방지
- 솔트 사용 가능
- HMAC + SHA 조합

 🔧 사용 예

```java
PasswordEncoder encoder = new Pbkdf2PasswordEncoder(
    "secret",       // secret key (optional)
    200000,         // iterations
    256,            // hash width
    SecretKeyFactoryAlgorithm.PBKDF2WithHmacSHA256
);
String encoded = encoder.encode("password");
```

------

#### 4. 🔐 **SCryptPasswordEncoder**

- 메모리와 CPU를 동시에 많이 쓰게 해서 **GPU 공격 방지**
- 솔트 자동 사용

 🔧 사용 예

```java
PasswordEncoder encoder = new SCryptPasswordEncoder();
String encoded = encoder.encode("password");
```

------

#### 5. 🔐 **Argon2PasswordEncoder** (가장 강력한 단방향 암호화)

- 2015년 Password Hashing Competition 우승
- **메모리**, **병렬성**, **반복 횟수**까지 조절 가능

 🔧 사용 예

```java
PasswordEncoder encoder = new Argon2PasswordEncoder(
    16,    // salt length
    32,    // hash length
    1,     // parallelism
    1 << 16, // memory (65536 KB)
    3      // iterations
);
String encoded = encoder.encode("password");
```

------

### ✅ Spring에서 여러 방식 지원하는 구조

```java
// SecurityConfig.java

@Bean
public PasswordEncoder passwordEncoder() {

    Map<String, PasswordEncoder> encoders = new HashMap<>();
    encoders.put("bcrypt", new BCryptPasswordEncoder());
    encoders.put("pbkdf2", new Pbkdf2PasswordEncoder("", 200000, 256, Pbkdf2PasswordEncoder.SecretKeyFactoryAlgorithm.PBKDF2WithHmacSHA256));
    encoders.put("argon2", new Argon2PasswordEncoder(16, 32, 1, 65536, 3));
    encoders.put("noop", NoOpPasswordEncoder.getInstance());

    return new DelegatingPasswordEncoder("bcrypt", encoders);
}


// Member.java
public class Member extends BaseEntity {

  	...
    
    public void encryptPassword(PasswordEncoder passwordEncoder) {
        this.password = passwordEncoder.encode(this.password);
    }
}

// MemberServiceImpl.java
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;
    private final MemberMapper memberMapper;
    private final PasswordEncoder passwordEncoder;

    @Override
    public Long save(MemberRegisterRequest request) {

        Member member = memberMapper.toMember(request);
        member.encryptPassword(passwordEncoder); // 도메인 메서드를 정의해서 사용 중

        return memberRepository.save(member).getId();
    }
}
```

`{bcrypt}...`, `{argon2}...` 같은 prefix를 저장해두면 `matches()` 시 자동 분기됨.



### 🔚 결론

- **복호화할 수 없는 단방향 암호화**는 비밀번호 저장에 필수
- 자바에서는 `Spring Security`의 `PasswordEncoder`로 대부분 구현 가능
- 기본적으로는 `BCrypt`, 고급 보안이 필요하다면 `Argon2`를 고려
