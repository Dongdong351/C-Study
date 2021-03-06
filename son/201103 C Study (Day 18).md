20-11-03 C Study (Day 18)
=====
교제 : 윤성우의 열혈 C 프로그래밍

chapter 21 : 문자와 문자열 관련 함수  

<hr>

## 문자열 단위 입출력 함수

printf와 scanf 함수도 문자열의 입출력 기능을 수행할 수 있으나 scanf는 공백이 포함된 문자열을 받는데 제한이 있다.  
다음의 함수들을 살펴보자  

### 문자열 출력 함수: puts. fputs

```
int puts(const char * s);
int fputs(const char * s, FILE * stream);
	-> 성공시 음수가 아닌 값을, 실패시 EOF 반환
```

둘 다 첫 번째 인자로 전달되는 주소 값의 문자열을 출력하나, 출력의 형태에 있어 차이점이 있다.  
puts 함수는 문자열 출력 후에 개행을 해주나 fputs는 개행을 수행하지 않는다.  

### 문자열 입력 함수: gets, fgets

```c
char * gets(char * s);
char * fgets(char * s, int n, FILE * stream);
	-> 파일의 끝에 도달하거나 함수호출 실패 시 NULL 포인터 반환
```

문자열을 입력 받는 것이다 보니 미리 마련된 메모리 공간을 침범하는 길이의 문자열이 입력되면 오류가 발생하게 된다.  
가급적 다음과 같이 함수를 사용하자.  

```c
char str[7];
fgets(str, sizeof(str), stdin);
```

fgets 함수에는 몇가지 기능이 있는데 기본적으로 sizeof(str)의 길이만큼만 str에 저장하며 '\n'문자를 만날 때 까지 문자열을 읽어 들인다.  
예를 들면 `123456789`를 입력 받으면 `123456\0`이러한 문자열이 str에 저장되게 된다.  
sizeof(str)의 값인 7보다 하나 작은 6만큼 읽어들인 후 끝에 \0(널 문자)를 삽임해 문자열임을 알린다.  

입력을 3번 나누어 받아 각각 str1, str2, str3에 저장한다고 하자  
`I \n like \n you \n` 이렇게 입력이 오면  
str1 = "I \n"  
str2 = " like \n"  
str3 = " you \n"  
이런식으로 저장된다 당연하게도 각 문자열의 끝에는 널 문자가 삽입되어 있다.  

## 표준 입출력과 버퍼

지금까지 배운 입출력 함수들은 ANSI C의 표준에서 정의된 '표준 입출력 함수'라고 한다.  
이러한 입출력 함수를 통해 데이터를 입출력 하는 경우에 데이터들은 운영체제가 제공하는 '메모리 버퍼'를 통과하게 된다.  

버퍼란 데이터를 임시로 모아두는 공간을 말한다.  
버퍼링이란 단어는 우리가 컴퓨터에 대해서 잘 모를 때도 많이 접하였다. 유튜브 영상을 시청하는데 영상이 끊기는 현상을 "아 버퍼링 걸렸네" 하며 불평을 한 경험이 있을 것이다.  
그렇다면 버퍼링은 왜 하는 것일까?

### 버퍼링을 하는 이유?

외부장치로의 데이터를 입출력 하는 과정은 생각보다 시간이 걸리는 작업이다. 이러한 데이터 전송, 수신과정에 버퍼링 없이 즉각적으로 데이터를 목적지에 보내는 작업 보다는 한데 묶어서 보내는 것이 효율적일 것이다.  
공사장에서 작업을 할 때 손으로 하나하나 나르는 것 보다는 수레에 실어 한번에 옮기는 편이 수월할 것이다.  

### 출력버퍼를 비우는 fflush 함수

굳이 출력 버퍼를 비우는 함수라고 명명한 것은 입력버퍼를 비우는 함수가 따로 존재하지 않기 때문이다.  
출력버퍼를 비운다는 의미는 버퍼에 있는 데이터를 바로 목적지로 전송한다는 의미다.

```c
int fflush(FILE * stream);
	-> 함수호출 성공 시 0, 실패 시 EOF 반환
```

### 입력버퍼를 비워 보자

사람의 정보를 저장하기 위해 주민번호와 이름을 입력받는 프로그램의 예제를 보자

```c
#include <stdio.h>

int main(void) {
	
	char perID[7];
	char name[10];

	fputs("주민번호 앞 6자리 입력: ", stdout);
	fgets(perID, sizeof(perID), stdin);

	fputs("이름 입력: ", stdout);
	fgets(name, sizeof(name), stdin);

	printf("주민번호: %s \n", perID);
	printf("이름: %s \n", name);
	return 0;
}
```

```
주민번호 앞 6자리 입력: 980413
이름 입력: 주민번호: 9890413
이름: 


```
주목해야 할 점은 perID의 길이가 7인 것과 `이름 입력:`뒤에 아무것도 입력되지 않을 것이다.  
입력은 `980413\n`을 입력해서 `perID`에 `"980413\0"`이 문자열로 저장되고 버퍼에 남은 `"\n\0"`이 `name`에 저장된 것이다. (문자열임을 알리기 위해 \0 문자가 삽입 되었음을 보자)  
`이름:`의 출력에 개행이 2번 된 것을 보면 알 수 있다.  

`이름 입력: `에도 입력을 받기 위해 버퍼를 비워보자.  

```
#include <stdio.h>

void ClearReadBuffer(void) {
	while(getchar() != '\n');
}

int main(void) {
	
	char perID[7];
	char name[10];

	fputs("주민번호 앞 6자리 입력: ", stdout);
	fgets(perID, sizeof(perID), stdin);
	ClearReadBuffer();	// 입력 버퍼 비움
	
	fputs("이름 입력: ", stdout);
	fgets(name, sizeof(name), stdin);

	printf("주민번호: %s \n", perID);
	printf("이름: %s \n", name);
	return 0;
}
```
이렇게 구성을 하면 아무리 첫 입력으로 주민번호 13자리를 전부 입력 받아도 앞 6자리만 출력되고 이후에 이름을 입력 받는다.