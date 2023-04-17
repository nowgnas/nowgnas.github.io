---
title: 사용자 정보 암호화 하기 (with java)
categories: java
date: 2022-10-09 16:21 +0900
description: 자바에서 암호화 알고리즘 사용에 대해 알아본다
lastmod: 2022-10-09 16:21 +0900
tags: SHA256 encrypt AES128 decrypt java
---

# User Security VO(Value Object)

사용자가 회원가입 시 저장되는 비밀번호는 암호화 되어야 한다. 사용자 비밀번호 암호화를 위한 security VO를 이용해 사용자 id, salt, secret key를 저장한다.

# 암호화 (Encryption)

사용자 비밀번호 암호화를 하기 전 암호화에 대해 간단히 알아본다.

암호화는 크게 양방향 암호화, 단방향 암호화가 있다. 양방향은 키를 이용한 복호화가 가능하다는 차이점이 있다.

## 양방향 암호화

양방향 암호화에는 대칭키 방식과 비대칭키 방식으로 나눌 수 있다. 대칭 키 방식은 암호화와 복호화에 사용하는 키가 같은 것이고, 비대칭키 방식은 암호화 하는 키와 복호화 하는 키가 다른 것이다.

대칭키 방식에는 AES, DES, SEED 등이 있다. 이번 포스팅에서는 AES128 방식을 사용할 것이다.

비대칭키 방식에는 DAS, RSA가 있다. 대칭키에 비해 느리지만, 키 생성 시 private key와 public key가 생성된다.

## 단방향 암호화

단방향 암호화는 hash를 이용해서 암호화 하는 방법이다. 평문을 암호화 할 수 있지만, 복호화는 불가능하다.

단방향 암호화는 사용자 인증, 인가에 사용되는 JWT(json web token)에 사용되는 방식이다.

# 사용자 정보 암호화 하기

이번 예시에서는 `AES128` 방식으로 평문을 암호화 해볼 것이다. 암호화가 사용되는 시점은 사용자 회원가입 시 발생하게 된다.

비밀번호는 단방향 해시 알고리즘인 `SHA256`을 사용해 암호화 할 것이다. 단방향 암호화 알고리즘을 사용해 복호화가 불가능하게 한다.

`java.security`와 `javax.crypto` 클래스에서 import 하여 사용한다.

### 1. Generate key

AES 128 방식으로 secret key를 발급한다.

```java
import javax.crypto.KeyGenerator;

public class Encrypt{
...
	public static byte[] generateKey(String algorithm, int keySize) throws
            NoSuchAlgorithmException {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(algorithm);
        keyGenerator.init(keySize);
        SecretKey key = keyGenerator.generateKey();
        return key.getEncoded();
    }
...
}
```

`generateKey` 메서드에 `algorithm`에는 암호화의 방식, `keySize`는 키의 크기가 들어간다. `algorithm`은 AES, `keySize`는 128로 한다.

javax.crypto의 KeyGenerator를 사용해 알고리즘 종류와 키의 크기로 초기화 해준다. KeyGenerator의 generateKey()를 이용해 키를 생성한 후 getEncoded()로 키를 반환한다.

```java
public static String byteArrayToHex(byte[] ba) {
    if (ba == null || ba.length == 0) {
        return null;
    }
    StringBuffer sb = new StringBuffer(ba.length * 2);
    String hexNumber;
    for (int x = 0; x < ba.length; x++) {
        hexNumber = "0" + Integer.toHexString(0xff & ba[x]);
        sb.append(hexNumber.substring(hexNumber.length() - 2));
    }
    return sb.toString();
}
```

`generateKey()`메서드에서 얻어진 byte 배열을 16진수의 문자열로 변환하는 `byteArrayToHex` 메서드이다.

```java
byte[] key = generateKey("AES", 128);
System.out.println("key length: " + key.length);
String hexKey = byteArrayToHex(key);
System.out.println(hexKey);

/*
key length: 16
9880ac1ecc316c9929b700a95b38b372
*/
```

`byteArrayToHex`를 사용하여 secret key를 출력해 보았다. 키의 길이는 16, 아래는 키를 16진수로 변환한 것이다.

이제 암호화에 사용할 secret key는 발급되었다. security VO 모델에 사용자 id와 salt에 저장될 UUID, 발급된 AES 128 16진수 키를 저장한다.

### 2. 사용자 이름 암호화

```java
public static String aesEncrypt(String msg, byte[] key) throws Exception {
    SecretKeySpec skeySpec = new SecretKeySpec(key, "AES");
    Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
    String iv = "AAAAAAAAAAAAAAAA";
    cipher.init(Cipher.ENCRYPT_MODE,
            skeySpec,
            new IvParameterSpec(iv.getBytes()));
    byte[] encrypted = cipher.doFinal(msg.getBytes());
    return byteArrayToHex(encrypted);
}
/*
평문: lee
encrypt: 91f0742622d1defa32ca78f399cd62e7
*/
```

데이터베이스에 저장되는 사용자의 정보는 마스킹한 후 저장되어야 한다. 키를 사용해 암호화와 복호화가 가능한 Cipher 클래스를 사용해 인스턴스를 생성하고, Cipher mode, secret key, 복호화에 사용할 키를 넣어 초기화해 준다. 초기화한 Cipher에 평문을 byte로 넣어 암호화 한 후 16진수로 변환해 반환한다. 반환된 16진수 문자열은 데이터베이스에 저장된다.

### 3. 사용자 비밀번호 암호화

```java
public static byte[] getSHA256(String source, String salt) {
    byte byteData[] = null;
    try {
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        md.update(source.getBytes());
        md.update(salt.getBytes());
        byteData = md.digest();
        System.out.println("원문: " + source + " SHA-256: " +
                byteData.length + "," + byteArrayToHex(byteData));
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    }
    return byteData;
}
/*
원문: 1234
SHA-256: 4b3bed8af7b7612e8c1e25f63ba24496f5b16b2df44efb2db7ce3cb24b7e96f7
pw encrypt: K;���a.�%�;�D���k-�N�-��<�K~��
*/
```

비밀번호는 단방향 암호화 알고리즘인 SHA256 방식으로 암호화 되어 저장된다. 복호화 가능한 키가 없는 해시 알고리즘이다. `getSHA256`메서드의 두 번째 인자인 `salt`는 위에서 security VO 모델에 저장한 salt를 사용하게 된다.

MessageDigest 클래스를 사용하여 인스턴스를 생성해 준다. 평문과 salt를 byte로 변환해 추가한다. 데이터베이스에 암호화된 byte 데이터가 저장된다.

# 사용자 정보 복호화 하기

마스킹되어 저장된 사용자 이름을 복호화 해 볼 것이다.

```java
public static String aesDecrypt(String msg, byte[] key) throws Exception {
    SecretKeySpec skeySpec = new SecretKeySpec(key, "AES");
    Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
    String iv = "AAAAAAAAAAAAAAAA";
    cipher.init(Cipher.DECRYPT_MODE,
            skeySpec,
            new IvParameterSpec(iv.getBytes()));
    byte[] encrypted = hexToByteArray(msg);
    byte[] original = cipher.doFinal(encrypted);
    return new String(original);
}
```

데이터베이스에 있는 사용자 정보를 복호화 하기 위해서 암호화에 사용된 secret key와 ivParameter가 필요하다. Cipher 초기화 부분에 mode 만 `DECRYPT_MODE`로 변경해 주면 된다. 암호화된 msg를 다시 byte 배열로 변환해 준 후 복호화를 진행한다. 복호화를 하게 되면 암호화 시 넣었던 평문이 반환된다.

---

## 암호화된 비밀번호는 어떻게 사용할까?

데이터베이스에 사용자 비밀번호는 해시 함수를 사용하여 암호화 되어 있다. 사용자가 정보를 변경하거나 어떤 동작에서 비밀번호를 입력 하는 행위가 필요한 경우 사용자가 입력한 비밀번호를 동일한 방식으로 해시화 해서 데이터베이스에 저장된 값과 비교한다.
