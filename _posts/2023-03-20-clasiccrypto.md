## 고전 암호 기술

- Symmetric Cipher Model >> 송신자 & 수신자가 동등하다
- Substitution Techniques >> 대체 기술 ex) a를 다른걸로
- Transposition Techniques >> 순서 바꾸기 - reordering
- Rotor Machines >> 
- Steganography

---

Symmetric Encryption (대칭 암호)
- Referred to as
	- Conventional/private-key/single-key encryption (전통적인/개인/단일 키 암호라고도 함)
- Sender and recipient share a common key (송신자와 수신자가 공통 키를 공유함)
	– Encryption and decryption with the same key (encryption과 decryption이 하나의 키로 이뤄짐!)
- All classical encryption algorithms are **private-key** (모든 고전 암호 기법은 **비공개 키**임)

- 퍼블릭키와 상호 보완적임
	- Was only type prior to invention of publickey in 1970’s (1970년대 공개 키 발명 이전에는 유일한 유형이었음)


---
### Some Basic Terminology

- Plaintext - original message
- Ciphertext - coded message**
- Cipher - algorithm for transforming plaintext to ciphertext
- Key - info used in cipher known only to sender/receiver**
• Encipher (encrypt) - converting plaintext to ciphertext
• Decipher (decrypt) - recovering ciphertext from plaintext
• Cryptography - study of encryption principles/methods
• Cryptanalysis (codebreaking) - study of principles/ methods of deciphering ciphertext without knowing key >> 
• Cryptology - field of both cryptography and cryptanalysis >> 위에 두개 포함


•  평문 - 원본 메시지
• 암호문 - 암호화된 메시지
• 암호 - 평문을 암호문으로 변환하는 알고리즘
•  키 - 송신자/수신자만 알고 있는 정보
• 암호화 - 평문을 암호문으로 변환
• 복호화 - 암호문을 평문으로 복원
• 암호학 - 암호화 원리/방법 연구
• 암호 해석(암호 분석) - 키를 모르고 암호문을 해독하는 원리/방법 연구
• 암호학 - 암호학과 암호 해석(암호 분석)의 분야 >> 위 두 가지 포함

![](https://velog.velcdn.com/images/yuran3391/post/dc40cf06-77f4-41f9-8bc9-a1d05b79dd42/image.png)

5 ingredients of symmetric cipher model
1. Plaintext
2. Encryption algorithm
3. Secret key > k : 핵심 >> 비밀키 k 이외에 모두 공격자가 알고 있다고 가정함, ex) AES 알고리즘
4. Ciphertext
5. Decryption algorithm

대칭 암호 모델
• 안전한 사용을 위한 2가지 요구 사항

1. 강력한 암호화 알고리즘
• 상대방은 암호문을 해독하거나 키를 발견할 수 없어야 함
• 공격 모델 - 보안이 강화될수록 뒤로 갈수록 강해짐, 알고리즘이 안전한가?를 고려하려면 어디까지 권한을 갖을 수 있나도 고려해서 고민
– 암호문만 알고 있는 공격 >> 공격자는 암호문만 볼 수 있다고 가정, 암호문에서 평문을 알아내는 것이 목적 / 알려진 평문 공격(수동) >> 암호화된 데이터와 그것에 대한 평문을 알고 있는 경우, 공격자는 암호화에 사용된 키를 찾을 수 있음
– 선택된 평문 공격 >> 선택된 값의 암호문을 볼 수 있음 / 선택된 암호문 공격(적극적) >> 암호문과 선택된 텍스트를 모두 알 수 있음.
     a. 알고리즘이 안전해야 하고
    b. 키가 안전해야 함 (키 공유 방식은 안전하다고 가정)
2. 송신자/수신자만 알고 있는 비밀 키
• 암호화 알고리즘이 알려져 있다고 가정
• 키 배포를 위한 안전한 채널이 필요함


![](https://velog.velcdn.com/images/yuran3391/post/436dee66-3ec3-48d0-ba91-1df21999e662/image.png)

공격자는 k나 x를 찾는 것이 목적

### 암호학

암호 시스템을 특징 짓는 방법:

1. 암호화 연산의 유형
• 대체
• 순서 바꾸기
• 곱셈 (여러 단계의 곱셈)
2. 키의 수
• 단일 키 또는 비공개 키
• 두 개의 키 또는 공개 키
3. 평문 처리 방식
• 블록 암호 >> 여러 비트의 정보를 하나의 블록으로 처리 ex) AES
• 스트림 암호 >> 비트 또는 바이트 단위로 처리

공격 목적 및 암호화 시스템 - 메시지뿐만 아니라 키를 복구
• 일반적인 접근 방법:

1. 암호 해석 공격 >> 알고리즘이 갖고 있는 취약점을 찾음, 알고리즘에 대한 의존성이 있음
• 알고리즘 특성을 이용함
2. 브루트 포스 공격 >> 가능한 모든 키를 시도함, 2의 n 승까지, 그러나 시간과 컴퓨팅 파워가 많이 들어가는 단점이 있음, 알고리즘을 사용하지 않음
• 모든 가능한 키를 시도함

- Always possible to simply try every key
- Most basic attack, proportional to key size
- Assume either know / recognise plaintext

![](https://velog.velcdn.com/images/yuran3391/post/2bd510fe-d487-40ad-a5d6-5a88e00f7f8c/image.png)

---
### 절대적인 보안

어떤 경우에도 이론적으로 깰 수 없는 것, 한 번만 쓰는 비밀 키
– 계산에 사용 가능한 자원이 얼마나 많든지 암호를 깰 수 없음
– 예: 일회용 비밀 키(One-time pad)
 

### 계산적 보안

제한된 컴퓨팅 자원하에서 깰 수 없는 것, 우리가 사용하는 시스템
– 한정된 컴퓨팅 리소스가 주어지면 암호를 깰 수 없음
– 기준:
 
1. **암호를 깨는 데 드는 비용 > 정보의 가치**
2. **암호를 깨는 데 걸리는 시간 > 정보의 유용 수명**

### 암호화의 두 가지 기본 구성 요소

1. 대체
- 평문의 문자를 다른 문자, 숫자 또는 기호로 대체함

2. 순서 바꾸기

#### 시저 암호
• Julius Caesar가 발명한 가장 오래되고 가장 간단한 대체 암호
• 군사 문제에서 최초로 사용됨
• 각 문자를 3번째 문자로 대체함

>
평문: meet me after the toga party
암호문: PHHW PH DIWHU WKH WRJD SDUWB
>
• 변환을 다음과 같이 정의함:
평문:
a b c d e f g h i j k l m n o p q r s t u v w x y z
암호문:
D E F G H I J K L M N O P Q R S T U V W X Y Z A B C
>
• 각 문자에 숫자를 할당하고 시저 암호를 적용함
평문:   a b c d e f g h i j k l m n o p q r s t u v w x y z
암호문: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25

• 시저 암호는 다음과 같음:
c = E(k, p) = (p + k) mod 26
p = D(k, c) = (c – k) mod 26

시저 암호는 brute force 공격 할만 함 왜냐하면,,,
1. 암호화 / 복호화 알고리즘이 알려져 있음
2. 시도해볼 키가 25개뿐임
3. 평문의 언어가 알려져 있으며 쉽게 인식됨

그런데 컴퓨터에게는 어려울 수 도 있음 (상대적으로 사람보다)

복호화 된 모든 텍스트가 타당해 보인다면 어떻게 될까요?
• 우리는 brute-force 공격에 대해 알고리즘을 보안 할 수 있습니다.
→ "Honey Encryption"의 주요 아이디어

#### Monoalphabetic Cipher

- 알파벳을 단순히 이동시키는 것이 아니라 임의로 섞음
• 각 평문 문자는 다른 임의의 암호 문자에 매핑됨
• 따라서 키는 26개의 문자로 구성됨

> 평문: abcdefghijklmnopqrstuvwxyz
암호문: DKVQFIBJWPESCXHTMYAUOLRGZN
각각 더해지는 수가 다름 >> 키가 더 많아져서 brute-force 공격이 힘듦, but, crypt analysis가 힘듦

>평문: ifwewishtoreplaceletters
암호문: WIRFRWAJUHYFTSDVFSFUUFYA

총 26! = 4 x 10^26 개의 키가 있게됨.
• 이렇게 많은 키가 있다면, 보안이 보장될 것 같지만, 그렇지는 않음

- 사람이 사용하는 언어는 중복되는 부분이 많고
- 문자가 모두 동일하게 자주 사용되지 않음 
예를 들어서 영어에서 E > T, R, N, I, O, A, S 가 많이 쓰이고
- Z, J, K, Q, X와 같은 다른 문자는 상대적으로 드문 편.
• 실제로 각 언어에 대한 단일, 이중 및 삼중 문자 빈도의 테이블이 있어 try & error을 통해 어느정도 추측 가능

#### Use in Cryptanalysis (암호해독)
주요 개념 - 단일 알파벳 대체
암호문의 문자 빈도를 계산합니다.

- 알려진 값과 비교합니다.
- A-E-I 트리플, NO 페어, RST 트리플에서 피크
- JK, X-Z에서 트러프
• 모노알파벳에서는 각 동일한 빈도의 문자를 식별해야 함
- 일반적인 더블 / 트리플 문자 테이블이 도움이 됩니다.

>
Given ciphertext:
UZQSOVUOHXMOPVGPOZPEVSGZWSZOPFPESXUDBMETSXAIZ
VUEPHZHMDZSHZOWSFPAPPDTSVPQUZWYMXUZUHSX
EPYEPOPDZSZUFPOMBZWPFUPZHMDJUDTMOHMQ
• Count relative letter frequencies (see text)
• Guess P & Z are ‘e’ and ‘t’
• Guess ZW is ‘th’ and hence ZWP is ‘the’
• Proceeding with trial and error finally get:
it was disclosed yesterday that several informal but
direct contacts have been made with political
representatives of the viet cong in moscow


#### Playfair Cipher
여러 문자를 암호

![](https://velog.velcdn.com/images/yuran3391/post/57130012-d68d-460a-8bb8-1fd15c482891/image.png)
키워드를 기반으로 한 5x5 문자 행렬
• 키워드의 문자를 채웁니다 (중복 제외)
• 나머지 행렬을 다른 문자로 채웁니다.
• 예: 키워드 MONARCHY

두 문자씩 암호화

1. 쌍이 반복되는 경우 'X'와 같은 채우기를 삽입합니다.
2. 두 문자가 동일한 행에있는 경우 각각 오른쪽의 문자로 바꿉니다 (끝에서 시작으로 감쌉니다).
• 예 : 'ar' → 'RM'
3. 두 문자가 동일한 열에있는 경우 각각 아래에있는 문자로 교체합니다 (끝에서 시작으로 감쌉니다).
• 예 : 'mu' → 'CM'
4. 그렇지 않으면 각 문자는 쌍의 다른 문자열의 동일한 행과 열에있는 문자로 대체됩니다.
• 예 : 'ea' → 'IM' (또는 'JM')

> 같은 단어라도 다른 출력값이 나옵니다.
>
- 모노알파벳보다 보안이 훨씬 개선됨
- 26 x 26 = 676 다이그램
- 분석을 위해 676 개의 빈도표가 필요합니다 (모노알파벳의 경우 26 개).
• 많은 연도 동안 널리 사용됨
- 예를 들어, 제1 차 세계 대전에서 미국과 영국 군대에서 사용됨
• 키 수는 몇 개입니까?
- 25! / 25 = 24!

#### Hill Cipher

![](https://velog.velcdn.com/images/yuran3391/post/73b8076b-12f2-47cb-a670-bf067e60a490/image.png)


연속적인 평문 문자 m을 m 개의 암호화 문자로 변환합니다.
– 암호문 만으로 공격? (o)
– 알려진 평문 공격? (x)

#### Polyalphabetic Ciphers
- Use different monoalphabetic substitutions as
one substitution
– Improve security using multiple cipher alphabets
• Make cryptanalysis harder with more alphabets
to guess and flatter frequency distribution 
- 서로 다른 모노알파벳 대체를 하나의 대체로 사용합니다.
	- 여러 개의 암호 문자를 사용하여 보안을 개선합니다.
- 더 많은 알파벳을 추측하면 cryptanalysis가 더 어려워집니다.
1. 각 메시지 문자에 대해 사용할 알파벳을 선택하는 키를 사용합니다.
2. 차례로 각 알파벳을 사용합니다.
3. 키의 끝에 도달하면 처음부터 다시 반복합니다.


#### Vigenère Cipher
• Simplest polyalphabetic substitution cipher
![](https://velog.velcdn.com/images/yuran3391/post/aad03001-177b-4d0b-ab75-3986b638bb93/image.png)

1. Write the plaintext out
2. Write the keyword repeated above it
3. Use each key letter as a Caesar cipher key
4. Encrypt the corresponding plaintext letter

1. 일반 텍스트를 작성하십시오.
2. 그 위에 반복되는 키워드를 적는다.
3. 각 키 문자를 카이사르 암호 키로 사용
4. 해당 평문 문자를 암호화

![](https://velog.velcdn.com/images/yuran3391/post/66d90020-fe14-4ff7-82ca-746b3613536a/image.png)



Multiple ciphertext letters for each plaintext
letter
• Hence letter frequencies are obscured, but not
totally lost
• If keyword length is m, the cipher consists of
m monoalphabetic substitution ciphers
– E.g., when m=9, 1st, 10th, 19th … letters are all
encrypted with the same monoalphabetic cipher
– Use the known frequency of the plaintext language
to attack each cipher separately


각각 다른 매핑 키가 있음?

문자 빈도가 변경되지 않기 때문에 공격이 가능합니다. 반복되기 때문입니다.

> 가정: 해커가 키 크기를 알고 있다.
> 

> 해결책: 반복성 제거 > Autokey Cipher

#### Autokey Cipher (반복 키 제거)
• 메시지 길이와 동일한 길이의 무작위 키를 사용합니다.
• 키는 메시지의 접두어로 사용됩니다.
• 예 : deception 키를 제공하면
키 : deceptivewearediscoveredsav
평문 : wearediscoveredsaveyourself
암호문 : ZICVTWQNGKZEIIGASXSTSLVVWLA
• 키워드를 알면 처음 몇 글자를 복구 할 수 있습니다.
• 이를 차례로 다른 메시지에 사용합니다.
• 여전히 공격 가능한 빈도 특성이 있습니다.

> 처음 몇 개의 시크릿이 깨지면 그에 대응하는 모든 것들의 안전성이 무너집니다.
#### Vernam Cipher
• 평문과 길이가 같은 키워드를 사용하는 최종 방어
• 키워드와 통계적 관계가 없음 >> 앞의 것을 반복하지 않음
• 1918 년 AT&T 엔지니어 Gilbert Vernam이 발명
• 문자 대신 이진 데이터 (글자)를 사용합니다.

![](https://velog.velcdn.com/images/yuran3391/post/f37ea789-d33c-41c8-b828-2a0398371326/image.png)

Key construction is the essence of Vernam cipher
• Originally proposed using a very long but eventually repeating key

키 구성은 Vernam 암호의 본질입니다.
• 원래 매우 길지만 결국 반복되는 키를 사용하도록 제안됨

#### One-Time Pad
• Vernam 암호의 개선
• 메시지와 같은 길이의 진정한 무작위 키를 사용하면 암호화가 안전합니다.

1. 메시지와 같은 길이의 무작위 키 (반복 안됨)
2. 키는 하나의 메시지에만 사용됩니다.
• 암호문은 평문과 통계적 관계가 없기 때문에 해독이 불가능합니다.
• 무조건적인 보안 또는 완벽한 보안

but 실제로 두 가지 근본적인 어려움
– 대량의 랜덤 키 생성
– 키 배포 및 보호


#### Transposition Ciphers

Classical transposition or permutation ciphers
– Hide the message by rearranging the letter order
without altering the actual letters used
• Can recognize these since have the same
frequency distribution as the original text 

고전적인 전치 또는 치환 암호
- 실제 사용된 문자를 변경하지 않고 문자 순서를 재배열하여 메시지를 숨깁니다.
• 원래 텍스트와 동일한 빈도 분포를 가지고 있기 때문에 이를 인식할 수 있습니다.


#### Rail Fence cipher
• 가장 간단한 치환 암호 중 하나

1. 메시지 문자를 대각선으로 여러 줄에 써 넣습니다.
2. 그런 다음, 한 줄씩 암호를 읽습니다.
• 예: 메시지를 다음과 같이 쓰면 (깊이 2):
m e m a t r h t g p r y
e t e f e t e o a a t
암호문
MEMATRHTGPRYETEFETEOAAT


#### Row Transposition Ciphers
• 더 복잡한 전치

1. 지정된 열 수로 메시지의 문자를 행에 작성합니다.
2. 그런 다음 어떤 키에 따라 열을 재배열 한 다음 행을 읽습니다.
키 : 4 3 1 2 5 6 7
평문 : a t t a c k p
o s t p o n e
d u n t i l t
w o a m x y z
암호문 : TTNAAPTMTSUOAODWCOIXKNLYPETZ

두 번 하면 훨씬 복잡해집니다.
전치 → 간단하지만 여러 번 시도하면 어려워집니다.

#### Product Ciphers

• 치환 또는 전치를 사용한 암호는 언어의 특성으로 인해 안전하지 않습니다.
• 따라서 여러 암호를 연속적으로 사용하여 보안을 높이는 것이 좋습니다. 하지만:
– 두 개의 치환은 더 복잡한 치환을 만듭니다.
– 두 개의 전치는 더 복잡한 전치를 만듭니다.
– 그러나 치환 후 전치를 사용하면 새로운 훨씬 더 어려운 암호가 만들어집니다.
• 이것이 고전적인 암호에서 현대적인 암호로 이어지는 다리 역할을 합니다.

#### 스테가노그래피 → 정보 숨기기

암호화 대체 방법
• 암호화
  – 메시지를 외부인이 이해할 수 없게 만듭니다.
• 스테가노그래피
– 메시지의 존재를 숨깁니다.
– 예:
• 일부 문자 / 단어 만 사용하고 어떤 방식으로 표시 된 긴 메시지
• 보이지 않는 잉크 사용
• 그래픽 이미지 또는 음성 파일의 LSB (최하위 비트)에 숨김

단점
– 상대적으로 적은 정보 비트를 숨기는 데 많은 오버 헤드가 필요합니다.
– 발견하면 가치가 없어집니다.
• 장점
– 암호화 사용을 가리 킬 수 있습니다.
– 먼저 암호화 한 다음 스테가노그래피를 사용하여 숨깁니다.

---
Summary
• Classical cipher techniques and terminology
• Substitution ciphers
– Monoalphabetic substitution ciphers
– Cryptanalysis using letter frequencies
– Playfair cipher
– Polyalphabetic ciphers
• Transposition ciphers
• Product ciphers
• Stenography
