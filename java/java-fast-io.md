# Java Fast I/O

### 1. Scanner & System.out.println

* **Concept:** Java의 기본 입출력 방식. 정규식 검사 등 내부 파싱 과정이 많아 속도가 매우 느림.
* **Constraint:** 입력 데이터가 많은 문제에서는 시간 초과 확률이 높음.

<img src = "https://miro.medium.com/v2/resize:fit:1200/1*XQjeodKw0dCIeIwiypsHMw.png" width = "80%">

<br/>

### 2. BufferedReader

* **Concept:** 입력 스트림에서 문자를 읽어올 때 내부 버퍼(Buffer)를 거쳐서 한 번에 덩어리째로 읽어오는 방식
* **Best for:** 데이터 입력량이 많은 문제에서 사용
* **Key Methods & Syntax:**
  * **`BufferedReader br = new BufferedReader(new InputStreamReader(System.in));`** 
    *  `System.in`: 키보드로부터 입력받는 원시적인 **바이트 스트림**
    * `InputStreamReader`: 바이트 스트림을 사람이 읽을 수 있는 **문자 스트림**으로 변환하는 다리 역할
    * `BufferedReader`: 문자 스트림을 **버퍼(메모리)에 모아두었다가 한 번에 전달**하여 입출력 병목을 줄이는 역할


  * **`readLine()`**: 엔터(`\n`)를 기준으로 한 줄을 통째로 읽어와 `String`으로 반환. 더 이상 읽을 데이터가 없으면(EOF) `null`을 반환함.


* **Constraint:** 
  * 반환값이 무조건 `String`이므로 `Integer.parseInt()` 등을 통한 형변환이 필요하다.

  * **반드시 `throws IOException` 예외 처리를 해주어야 한다.** <br/>
    자바는 파일이나 네트워크, 콘솔 등의 입출력(I/O) 과정에서 발생할 수 있는 오류(예: 스트림 끊김, 하드웨어 문제)를 개발자가 반드시 대처하도록 강제한다(Checked Exception). <br/>
    코딩테스트에서는 일일이 `try-catch`로 감싸기 번거로우므로, `main` 메서드 선언부에 `throws IOException`을 붙여 예외 처리를 시스템에 던져버리는 방식을 사용한다.

<br/>

### 3. StringTokenizer (String Parsing)

* **Concept:** `BufferedReader`로 읽어온 문자열 한 줄을 특정 구분자(기본값: 공백)를 기준으로 빠르게 잘라주는(Tokenize) 도구.
* **Key Methods:**
  * **`nextToken()`**: 잘라낸 단어(토큰)를 앞에서부터 하나씩 꺼내어 반환(`String` 타입).
  * **`hasMoreTokens()`**: 썰어둔 단어(토큰)가 아직 남아있는지 확인하여 `true` 또는 `false`를 반환. 꺼낼 개수를 정확히 모를 때 `while`문의 조건식으로 활용하기 좋음.

<img src = "https://www.scaler.com/topics/images/stringtokenizer-in-java_thumbnail.webp" width = "80%">

<br/>

### 4. StringBuilder (Fast Output)

* **Concept:** 매번 콘솔에 출력하지 않고, 변경 가능한 문자열 버퍼에 정답을 계속 이어 붙인(`append`) 뒤 마지막에 한 번에 출력하는 방식
* **Best for:** 반복문 안에서 `System.out.println`이 빈번하게 호출되어야 할 때 출력 속도를 단축시킴.

<br/>

### 5. Example Code

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;
import java.util.StringTokenizer;

public class Main {
    public static void main(String[] args) throws IOException { // I/O 예외 던지기 필수!

        // 바이트 스트림 -> 문자 스트림 -> 버퍼링을 거쳐 빠른 입력 세팅
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        
        // ====================================================
        // Case 1. 기본 구분자 (공백) 처리
        // ====================================================
        // 가상의 입력 1: "10 20 30"
        String line1 = br.readLine(); // 한 줄 읽어오기
        
        // 구분자 생략 시 기본적으로 '공백(스페이스, 탭 등)'을 기준으로 자름
        StringTokenizer st1 = new StringTokenizer(line1);
        
        int A = Integer.parseInt(st1.nextToken()); // A = 10
        int B = Integer.parseInt(st1.nextToken()); // B = 20
        int C = Integer.parseInt(st1.nextToken()); // C = 30
        
        
        // ====================================================
        // Case 2. 특정 구분자 지정하여 자르기 (쉼표 `,`)
        // ====================================================
        // 가상의 입력 2: "Apple,Banana,Cherry"
        String line2 = br.readLine();
        
        // 두 번째 파라미터에 기준이 될 문자(구분자)를 지정
        StringTokenizer st2 = new StringTokenizer(line2, ",");
        
        String first = st2.nextToken();  // first = "Apple"
        String second = st2.nextToken(); // second = "Banana"
        String third = st2.nextToken();  // third = "Cherry"
        
        
        // ====================================================
        // Case 3. 특정 구분자 + 개수를 모를 때 (while문 활용)
        // ====================================================
        // 가상의 입력 3: "2026/02/24" 
        String line3 = br.readLine();
        
        // 슬래시("/")를 기준으로 자름
        StringTokenizer st3 = new StringTokenizer(line3, "/");
        
        // 꺼낼 토큰이 남아있는 동안만 반복
        while (st3.hasMoreTokens()) {
            String token = st3.nextToken(); 
            // 1번째 루프: token = "2026"
            // 2번째 루프: token = "02"
            // 3번째 루프: token = "24"
        }
    }
}

```



<br/>

### References
[StringTokenizer in Java](https://www.scaler.com/topics/stringtokenizer-in-java/)

---
