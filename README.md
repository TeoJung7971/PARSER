## 2023 프로그래밍 언어론 1차 과제
### RECURSIVE DESCENT PARSER 구현하기
### EXTERNAL DOCUMENT

##### 20210398 예술공학부 정하영    
</br>

### [프로그램 구현 환경]
Visual Studio 2020
C Language 
</br>
### [필요 헤더 파일]
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
#include <string.h>
#include "getopt.h"
</br>

##### 이때, getopt.c와 getopt.h는 외부에서 가져와 파일 내에 추가적으로 넣어 실행해야 한다.
##### *https://gist.github.com/superwills/5815344*
</br>

### [기본적인 동작]
기본적인 동작은 과제의 요구되어지는 세부사항을 따랐음.

### [예외 처리 사항]

/*복구 불가능한 경우: ERROR RAISE*/

void ERROR_RAISE(int state_num);

    if (state_num == 1) {
        printf("ERROR: 파일을 불러올 수 없습니다\n");
    }
    else if (state_num == 2) {
        printf("ERROR: 정의되지 않은 변수를 참조했습니다\n");
    }
    else if (state_num == 3) {
        printf("ERROR: 불완전한 RHS입니다\n");
    }
    else
        printf("ERROR: 정의되지 않은 오류 발생\n");
}

- error_state = 1
  주어진 파일명으로 파일을 읽어올 수 없을 때 발생
  
      /* Open the input data file and process its contents */
    if (in_fp == NULL) {
        ERROR_STATE = 1;
        ERROR_RAISE(ERROR_STATE);
        return 0;
    }
  
- error_state = 2
  정의되지 않은 변수 참조 시 발생
  void ident_() {
    //printf("%d, %d", symbolTable->value, searchSymbolValue(lexeme, symbolTable));
  
      if (searchSymbol(lexeme, symbolTable) == NULL) {
        insertSymbol(lexeme, NULL, updateValue, &symbolTable);
        Unkown_ERROR_STATE = 1;
        ERROR_STATE = 2;
    }
    else {
        symbolTable->value = searchSymbolValue(lexeme, symbolTable);
    }
    
    lex();
}
  
- error_state = 3
  연산식의 마무리가 불완전하게 이루어졌을 때 발생
  예를 들어,
  target := 3 +
  와 같은 경우에 에러 발생

/*복구 가능한 경우: WARNING RAISE*/


void WARNING_RAISE(int state_num);

void WARNING_RAISE(int state_num) {

    if (WARNING_STATE == 1) {
        printf("WARNING: 값의 할당을 위해서는':' 다음에 '='가 와야 합니다\n");
    }
    else if (WARNING_STATE == 2) {
        printf("WARNING: 값의 할당을 위해서는'=' 이전에 ':'가 와야 합니다\n");
    }
    else if (WARNING_STATE == 3) {
        printf("WARNING:변수에 값을 할당하기 위해서는 ':=' 연산자가 필요합니다\n");
    }
    else if (WARNING_STATE == 4) {
        printf("WARNING: *|/뒤에 *|/가 연속해서 입력되었습니다: 마지막 입력으로 대체됩니다\n");
    }
    else if (WARNING_STATE == 5) {
        printf("WARNING: +|-뒤에 +|-가 연속해서 입력되었습니다: 마지막 입력으로 대체됩니다\n");
    }
}

- warning_state = 1
    ASSIGN_OP가 :만 주어졌을 때, 뒤에 =를 추가하고 파싱 이어서 진행
  
- warning_state = 2
  ASSIGN_OP가 =만 주어졌을 때, 앞에 :를 추가하고 파싱 이어서 진행
  
- warning_state = 3
  '변수명 숫자' 형식으로 주어졌을 때 := 를 추가하고 파싱 이어서 진행
  
- warning_state = 4
    **, */, //, /* 과 같이 연산자가 올 경우, 연산자 중복으로 판단, 마지막 입력을 최종적으로 파싱할 연산자로 설정한 이후 파싱 이어서 진행

    if (laterToken2 == MULT_OP) {
        if (nextToken == MULT_OP) {
            WARNING_STATE = 4;

            lexeme_list_cnt--;

        }
        else if (nextToken == DIV_OP) {
            WARNING_STATE = 4;

            lexeme_list_cnt--;

        }
    }
    if (laterToken3 == DIV_OP) {
        if (nextToken == MULT_OP) {
            WARNING_STATE = 4;

            lexeme_list_cnt--;

        }
        else if (nextToken == DIV_OP) {
            WARNING_STATE = 4;

            lexeme_list_cnt--;

        }
    }
  
- warning_state = 5
  ++, +-, --, -+ 과 같이 연산자가 올 경우, 연산자 중복으로 판단, 마지막 입력을 최종적으로 파싱할 연산자로 설정한 이후 파싱 이어서 진행

    if (laterToken0 == ADD_OP) {
        if (nextToken == ADD_OP) {
            WARNING_STATE = 5;

            lexeme_list_cnt--;

        }
        else if (nextToken == SUB_OP) {
            WARNING_STATE = 5;

            lexeme_list_cnt--;
        }
    }
     if (laterToken1 == SUB_OP) {
        if (nextToken == ADD_OP) {
            WARNING_STATE = 5;

            lexeme_list_cnt--;

        }
        else if (nextToken == SUB_OP) {
            WARNING_STATE = 5;

            lexeme_list_cnt--;

        }
    }
