## Classical Encryption Techniques - 고전 암호 기술

Symmetric Cipher Model >> 송신자 & 수신자가 동등하다
• Substitution Techniques >> 대체 기술 ex) a를 다른걸로
• Transposition Techniques >> 순서 바꾸기 - reordering
• Rotor Machines >> 
• Steganography

---

Symmetric Encryption
• Referred to as
– Conventional/private-key/single-key encryption
• Sender and recipient share a common key
– Encryption and decryption with the same key
• All classical encryption algorithms are **private-key**

>> 퍼블릭키와 상호 보완적임
****• Was only type prior to invention of publickey in 1970’s
• By far most widely used

encryption과 decryption이 하나의 키로 이뤄짐!

---

### Some Basic Terminology

**- Plaintext - original message
• Ciphertext - coded message**
• Cipher - algorithm for transforming plaintext to ciphertext
**• Key - info used in cipher known only to sender/receiver**
• Encipher (encrypt) - converting plaintext to ciphertext
• Decipher (decrypt) - recovering ciphertext from plaintext
• Cryptography - study of encryption principles/methods
• Cryptanalysis (codebreaking) - study of principles/ methods of deciphering ciphertext without knowing key >> 
• Cryptology - field of both cryptography and cryptanalysis >> 위에 두개 포함

---

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f4359573-90da-4ba5-9a23-115242ce6ed5/Untitled.png)

Y: X에 대응되는 Cyper text

Encryption과 Decryption에 같은 키가 사용됨.

5가지 요소

x : 

k : 핵심 >> 비밀키 k 이외에 모두 공격자가 알고 있다고 가정함, ex) AES 알고리즘 

y

encryption

decryption

---

Symmetric Cipher Model
• Two requirements for secure use

1. Strong encryption algorithm
• Opponent should be unable to decrypt
ciphertext or discover the key
• Attack model - 뒤로 갈 수록 보안이 강해짐, 알고리즘이 안전한가?를 고려하려면 어디까지 권한을 갖을 수 있나도 고려해서 고민
– Ciphertext-only attack >> 공격자는 Ciphertext만 볼 수 있다고 가정, Ciphertext로 부터 plain text를 아는게 목적  / Known-plaintext attack (passive) >> key는 모르지만(복호화 불가), 그 정보를 갖고 공격 및 추정 가능
– Chosen-plaintext attack >> 내가 선택한 값에 대해서는 알 수 있음 /  Chosen-ciphertext attack (active) >> Ciphertext와 chosen text를 알 수 있음.
    1. 알고리즘이 안전해야하고
    2. 키가 안전해야함 (키 공유 방식은 안전하다고 가정)
2. Secret key known only to sender / receiver
• Assume encryption algorithm is known
• Implies a secure channel to distribute key

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef6101b4-e756-4e39-b762-b2c167924b76/Untitled.png)

- 공격자: x나 k 를 guessing 하는 것이 목표

---

### Cryptography

Characterize cryptographic system by:

1. Type of encryption operations
• Substitution
• Transposition
• Product (multiple stages of them)
2. Number of keys
• Single-key or private
• Two-key or public
3. Way in which plaintext is processed
• Block cipher >> 멀티플 비트의 정보를 하나의 블락으로 봄 ex) AES
• Stream cipher >> 비트나 바이트 단위로 

Objective of attacking and encryption
system - Recover key not just message
• General approaches:

1. Cryptanalytic attack >> 알고리즘이 갖고 있는 취약점을 찾음 , 알고리즘에 대한 디펜던시가 있음
• Exploit characteristics of algorithms
2. Brute-force attack >> 될때까지 다해보는 방법, 2의 n 승 까지 , but 시간과 컴퓨팅 파워가 많이 든다는 단점이 있음, 알고리즘을 타지 않음
• Try every possible keys
    1. 
    
    Always possible to simply try every key
    • Most basic attack, proportional to key size
    • Assume either know / recognise plaintext
    
    자료: [https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42f74806-d057-4140-affc-b75425c50102/ch02.pdf](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42f74806-d057-4140-affc-b75425c50102/ch02.pdf)
    검토: 아니요
    유형: 강의
    
    ## 
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94e2a365-a256-4be1-8611-5e5f173ce234/Untitled.png)
    

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/511134af-5546-4eac-b062-7c5691562dfd/Untitled.png)

점점 공격자가 강해짐 

---

## More Definitions

### • Unconditional Security

>> 아무리 자원이 많아도 이론적으로 절대로 깰 수 없는거, only one time pad
– No matter how much computing power or time
is available, the cipher cannot be broken
– E.g., One-time pad

### • Computational security

>> 제한된 시간과 자원하에서 깨질 수 없는 거, 우리가 쓰는 시스템 
– Given limited computing resources, the cipher
cannot be broken
– Criteria:

1. **Cost of breaking cipher > value of information**
2. **Time to break cipher > useful lifetime of info.**

---

### Two basic building blocks of encryption

1. Substitution
2. Transposition
• Letters of plaintext are replaced by other
letters or by numbers or symbols

Caesar Cipher
• Earliest known, simplest substitution cipher by
Julius Caesar
• First attested use in military affairs
• Replaces each letter by 3rd letter on
• Example:
Plain: meet me after the toga party
Cipher: PHHW PH DIWHU WKH WRJD SDUWB
• Define transformation as:
Plain:
a b c d e f g h i j k l m n o p q r s t u v w x y z
Cipher:
D E F G H I J K L M N O P Q R S T U V W X Y Z A B C

Mathematically give each letter a number
Plain: a b c d e f g h i j k l m n o
Cipher: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14
Plain: p q r s t u v w x y z
Cipher: 15 16 17 18 19 20 21 22 23 24 25

• Then have Caesar cipher as:
c = E(k, p) = (p + k) mod 26
p = D(k, c) = (c – k) mod 26

>> bruteforce ㄱㄱ 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/81648744-e528-459b-a0a8-b0379ad31daa/Untitled.png)

What if all of the decrypted texts are plausible looking?
• We can make the algorithm secure
against brute-force attack
→ Main idea of “Honey Encryption”

---

### Monoalphabetic Cipher

• Shuffle the letters arbitrarily rather than just shifting the alphabet
• Each plaintext letter maps to a different random ciphertext letter
• Hence key is 26 letters long

Plain: abcdefghijklmnopqrstuvwxyz
Cipher: DKVQFIBJWPESCXHTMYAUOLRGZN
각각 더해지는 수가 다름 >> 키가 더더 많아져서 bruteforce 공격이 힘듦, but, crypt analysis가 힘듦 

Plaintext: ifwewishtoreplaceletters
Ciphertext: WIRFRWAJUHYFTSDVFSFUUFYA

Now have a total of 26! = 4 x 1026 keys
• With so many keys, might think is secure
• But would be !!!WRONG!!!
• Problem is language characteristics

BUT, 

• Human languages are redundant
• letters are not equally commonly used
– In English, E is by far the most common letter
– Followed by T,R,N,I,O,A,S
– Other letters like Z,J,K,Q,X are fairly rare
• Have tables of single, double & triple letter frequencies for various languages

어느정도 try & error 를 통해 guessing 가능

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fe15322c-bf5b-4d7a-b684-120bd08d4a51/Untitled.png)

Key concept - monoalphabetic substitution
ciphers do not change relative letter
frequencies
• Calculate letter frequencies for ciphertext
• Compare counts/plots against known values
– Peaks at: A-E-I triple, NO pair, RST triple
– Troughs at: JK, X-Z
• For monoalphabetic, must identify each letter
of similar frequency
– Tables of common double/triple letters help

>> 키를 무작정 늘리는 것도 능사가 아니다. 그래서 하나의 글자라고 하더라고 케이스에 따라 다양하게 매핑 되게 

### Playfair Cipher

Not even the large number of keys in a monoalphabetic cipher provides security
• One approach to improve security was to encrypt multiple letters
• Playfair Cipher is an example
– Invented by Charles Wheatstone in 1854,but named after his friend Baron Playfair

/

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/abeeff5f-a8ec-4419-b87c-2361feb8aeeb/Untitled.png)

5X5 matrix of letters based on a keyword
• Fill in letters of keyword (sans duplicates)
• Fill rest of matrix with other letters
• E.g., keyword MONARCHY

Encrypt two letters at a time

1. If a pair is a repeated letter, insert filler like 'X’
2. If both letters fall in the same row, replace
each with letter to right (wrapping back to
start from end)
• E.g., ‘
ar
’ → ‘RM’
3. If both letters fall in the same column, replace
each with the letter below it (wrapping to top
from bottom)
• E.g., ‘mu’ → ‘CM’
4. Otherwise each letter is replaced by the letter
in the same row and in the column of the
other letter of the pair
• E.g., ‘
ea
’ → ‘IM’ (or ‘JM’)

같은 단어라도 다른 아웃풋 값이 나옴 

- Security much improved over
monoalphabetic
– 26 x 26 = 676 digrams
– Need a 676 entry frequency table to analyze
(cf. 26 for a monoalphabetic)
• Widely used for many years
– E.g., by US & British military in WW1
• What is the number of keys?
– 25!/25 = 24!

### Hill Cipher

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a6a27ded-07b8-43c0-990e-edba56af9e8e/Untitled.png)

Encrypt m successive plaintext letters
into m ciphertext letters
• Secure against
– Ciphertext-only attack?
– Known plaintext attack?
yes
no

---

Polyalphabetic Ciphers
• Use different monoalphabetic substitutions as one substitution
– Improve security using multiple cipher alphabets
• Make cryptanalysis harder with more alphabets
to guess and **flatter frequency distribution**

1. Use a key to select which alphabet is used for
each letter of the message
2. Use each alphabet in turn
3. Repeat from start after end of key is reached

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07cc3ef1-04a5-4826-ba94-b1fe2c5ed685/Untitled.png)

각각 매핑되는 키가 달라짐 ? 

letter frequency가 안바뀌어서 공격 가능 → 반복해서 나옴

>> 전제: 공격자가 키사이즈를 알고 있다.

>>solution: 반복성 제거 > Autokey Cipher

---

Eliminate repeating keyword that is as long as the message itself
• Keyword is prefixed to message as key
• E.g., given key deceptive
key: deceptivewearediscoveredsav
plaintext: wearediscoveredsaveyourself
ciphertext:ZICVTWQNGKZEIIGASXSTSLVVWLA
• Knowing keyword can recover the first few
letters
• Use these in turn on the rest of the message
• Still have frequency characteristics to attack

>> 앞의 몇개의 시크릿이 깨지면 거기에 대응되는 모든 것들의 안정성이 무너짐 

---

Vernam Cipher
• Ultimate defense is
– Use a keyword as long as the plaintext
– No statistical relationship to it >> 앞에거 반복 x
• Invented by AT&T engineer Gilbert Vernam in 1918
• Works on binary data (rather than letters)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9c0629d1-a4e3-44e6-a52e-9d09745c286d/Untitled.png)

landom number generator

안정성으 핵심: keyu generator

---

One-Time Pad
• Improvement to Vernam cipher
• If a truly random key as long as the message
is used, the cipher will be secure

1. Random key that is as long as the message (no
repetition)
2. Key is only used to a single message
• Unbreakable since ciphertext bears no
statistical relationship to the plaintext
• Unconditional security, or perfect security
3. >> 예측 불기

Improvement to Vernam cipher
• If a truly random key as long as the message
is used, the cipher will be secure

1. Random key that is as long as the message (no
repetition)
2. Key is only used to a single message
• Unbreakable since ciphertext bears no
statistical relationship to the plaintext
• Unconditional security, or perfect security
One-Time Pad

—

러컴다오

Row Transposition Ciphers
• More complex transposition

1. Write letters of message out in rows over a specified
number of columns
2. Then reorder the columns according to some key
before reading off the rows
Key: 4 3 1 2 5 6 7
Plaintext: a t t a c k p
o s t p o n e
d u n t i l t
w o a m x y z
Ciphertext: TTNAAPTMTSUOAODWCOIXKNLYPETZ

\

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/760be45c-756e-425e-a0f9-e644af6647ea/Untitled.png)

두번 하면 훨씬 복잡해짐

transposition → 단순하지만 여러번하면 돌릴수록 하기 어려워짐

---

Product Ciphers
• Ciphers using substitutions or transpositions are not secure because of language characteristics
• Hence consider using several ciphers in succession to make harder, but:
– Two substitutions make a more complex
substitution
– Two transpositions make more complex
transposition
– But a substitution followed by a transposition makes a new much harder cipher
• This is bridge from classical to modern ciphers

둘다 써 >> 키 사이즈 안늘리고 더 복잡하게 할 수 있어, 릴레이션 정보도 감추면서! 

---

Steganography → 정보 심기 

Alternative to encryption
• Encryption
– Render message unintelligible to outsiders
• Steganography
– Hides existence of message
– E.g.,
• Using only a subset of letters/words in a longer
message marked in some way
• Using invisible ink
• Hiding in LSB(Least Siginificant Bit) in graphic image or sound file

Drawbacks
– High overhead to hide relatively few info bits
– Once discovered, it becomes worthless
• Advantage
– Can obscure encryption use
– First encrypted, then hidden using steganogrphy

---

Classical cipher techniques and terminology
• Substitution ciphers
– Monoalphabetic substitution ciphers
– Cryptanalysis using letter frequencies
– Playfair cipher
– Polyalphabetic ciphers
• Transposition ciphers
• Product ciphers
• Stenography

---

자료: [https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42f74806-d057-4140-affc-b75425c50102/ch02.pdf](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42f74806-d057-4140-affc-b75425c50102/ch02.pdf)
검토: 아니요
유형: 강의

## 고전 암호 기술

대칭 암호 기법
• 송신자와 수신자가 동등하다.
• 대체 기술
• 순서 바꾸기
• 로터 기계
• 스테가노그래피

---

대칭 암호
• 전통적인/개인/단일 키 암호라고도 함
• 송신자와 수신자가 공통 키를 공유함
• 암호화와 복호화에 동일한 키를 사용함
• 모든 고전 암호 기법은 **비공개 키**임

> 공개 키와 상호 보완적임
****• 1970년대 공개 키 발명 이전에는 유일한 유형이었음
• 지금까지 가장 널리 사용됨
> 

암호화와 복호화가 하나의 키로 이뤄짐!

---

### 몇 가지 기본 용어

- **평문 - 원본 메시지
• 암호문 - 암호화된 메시지**
• 암호 - 평문을 암호문으로 변환하는 알고리즘
**키 - 송신자/수신자만 알고 있는 정보**
• 암호화 - 평문을 암호문으로 변환
• 복호화 - 암호문을 평문으로 복원
• 암호학 - 암호화 원리/방법 연구
• 암호 해석(암호 분석) - 키를 모르고 암호문을 해독하는 원리/방법 연구
• 암호학 - 암호학과 암호 해석(암호 분석)의 분야 >> 위 두 가지 포함

---

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f4359573-90da-4ba5-9a23-115242ce6ed5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f4359573-90da-4ba5-9a23-115242ce6ed5/Untitled.png)

Y: X에 대응되는 암호화된 텍스트

암호화와 복호화에 같은 키가 사용됨.

5가지 요소

x :

k : 핵심 >> 비밀키 k 이외에 모두 공격자가 알고 있다고 가정함, ex) AES 알고리즘

y

암호화

복호화

---

대칭 암호 모델
• 안전한 사용을 위한 2가지 요구 사항

1. 강력한 암호화 알고리즘
• 상대방은 암호문을 해독하거나 키를 발견할 수 없어야 함
• 공격 모델 - 보안이 강화될수록 뒤로 갈수록 강해짐, 알고리즘이 안전한가?를 고려하려면 어디까지 권한을 갖을 수 있나도 고려해서 고민
– 암호문만 알고 있는 공격 >> 공격자는 암호문만 볼 수 있다고 가정, 암호문에서 평문을 알아내는 것이 목적 / 알려진 평문 공격(수동) >> 암호화된 데이터와 그것에 대한 평문을 알고 있는 경우, 공격자는 암호화에 사용된 키를 찾을 수 있음
– 선택된 평문 공격 >> 선택된 값의 암호문을 볼 수 있음 / 선택된 암호문 공격(적극적) >> 암호문과 선택된 텍스트를 모두 알 수 있음.
    1. 알고리즘이 안전해야 하고
    2. 키가 안전해야 함 (키 공유 방식은 안전하다고 가정)
2. 송신자/수신자만 알고 있는 비밀 키
• 암호화 알고리즘이 알려져 있다고 가정
• 키 배포를 위한 안전한 채널이 필요함

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef6101b4-e756-4e39-b762-b2c167924b76/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef6101b4-e756-4e39-b762-b2c167924b76/Untitled.png)

- 공격자: x나 k 를 추측하는 것이 목표

---

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
1.
    
    모든 키를 시도하는 것이 항상 가능합니다.
    • 가장 기본적인 공격, 키 크기에 비례합니다.
    • 평문을 알고있는 경우 / 인식하는 경우
    
    자료: [https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42f74806-d057-4140-affc-b75425c50102/ch02.pdf](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42f74806-d057-4140-affc-b75425c50102/ch02.pdf)
    

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94e2a365-a256-4be1-8611-5e5f173ce234/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94e2a365-a256-4be1-8611-5e5f173ce234/Untitled.png)

공격자가 강해짐

---

## 더 많은 정의

### 절대적인 보안

> 어떤 경우에도 이론적으로 깰 수 없는 것, 한 번만 쓰는 비밀 키
– 계산에 사용 가능한 자원이 얼마나 많든지 암호를 깰 수 없음
– 예: 일회용 비밀 키(One-time pad)
> 

### 계산적 보안

> 제한된 컴퓨팅 자원하에서 깰 수 없는 것, 우리가 사용하는 시스템
– 한정된 컴퓨팅 리소스가 주어지면 암호를 깰 수 없음
– 기준:
> 
1. **암호를 깨는 데 드는 비용 > 정보의 가치**
2. **암호를 깨는 데 걸리는 시간 > 정보의 유용 수명**

---

### 암호화의 두 가지 기본 구성 요소

1. 대체
2. 순서 바꾸기
• 평문의 문자를 다른 문자, 숫자 또는 기호로 대체함

시저 암호
• Julius Caesar가 발명한 가장 오래되고 가장 간단한 대체 암호
• 군사 문제에서 최초로 사용됨
• 각 문자를 3번째 문자로 대체함
• 예:
평문: meet me after the toga party
암호문: PHHW PH DIWHU WKH WRJD SDUWB
• 변환을 다음과 같이 정의함:
평문:
a b c d e f g h i j k l m n o p q r s t u v w x y z
암호문:
D E F G H I J K L M N O P Q R S T U V W X Y Z A B C
• 각 문자에 숫자를 할당하고 시저 암호를 적용함
평문: a b c d e f g h i j k l m n o
암호문: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14
평문: p q r s t u v w x y z
암호문: 15 16 17 18 19 20 21 22 23 24 25
• 시저 암호는 다음과 같음:
c = E(k, p) = (p + k) mod 26
p = D(k, c) = (c – k) mod 26

> 브루트 포스 ㄱㄱ
> 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/81648744-e528-459b-a0a8-b0379ad31daa/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/81648744-e528-459b-a0a8-b0379ad31daa/Untitled.png)

복호화 된 텍스트가 모두 그럴듯해 보인다면 어떻게 될까요?
• 알고리즘을 브루트 포스 공격으로부터 안전하게 만들 수 있습니다.
→ "Honey Encryption"의 주요 아이디어

---

### 단일 알파벳 암호

- 알파벳을 단순히 이동시키는 대신 문자를 임의로 섞음
• 각 평문 문자는 다른 무작위 암호문 문자로 매핑됨
• 따라서 키는 26 글자임

평문: abcdefghijklmnopqrstuvwxyz
암호문: DKVQFIBJWPESCXHTMYAUOLRGZN
각각 더해지는 수가 다름 >> 키가 더 길어져서 브루트 포스 공격이 어려워지지만, 암호 분석이 어려워짐

평문: ifwewishtoreplaceletters
암호문: WIRFRWAJUHYFTSDVFSFUUFYA

이제 총 26! = 4 x 1026 개의 키가 있음
• 너무 많은 키가 있으면 안전하다고 생각할 수 있음
• 그러나 언어 특성 때문에 문제가 발생함

하지만,

- 인간 언어는 중복성을 가짐
• 문자들은 모두 동일하게 자주 사용되지 않음
– 영어에서 E가 가장 일반적인 문자임
– 그 다음은 T, R, N, I, O, A, S
– Z, J, K, Q, X와 같은 다른 문자는 상당히 드물다.
• 다양한 언어에 대
