// 1. 전역 변수 설정 (어떤 함수에서든 쓸 수 있는 변수들)
int targetButton = 0; // 이번 라운드 목표 버튼 (2~5)

void setup() {
  Serial.begin(9600);
  
  // 랜덤 시드 초기화 (A0 핀의 전압 잡음 이용)
  randomSeed(analogRead(A0));

  // 버튼 핀 설정
  pinMode(2, INPUT); // 빨강
  pinMode(3, INPUT); // 노랑
  pinMode(4, INPUT); // 파랑
  pinMode(5, INPUT); // 흰색

  // LED 핀 설정 (1이 켜짐, 0이 꺼짐 기준)
  pinMode(9, OUTPUT);  // 초록 LED
  pinMode(10, OUTPUT); // 빨간 LED

  // 처음 시작할 때는 LED를 모두 끄고 시작
  digitalWrite(9, LOW);
  digitalWrite(10, LOW);
}

// 2. 새로운 라운드를 시작하는 함수 (setup과 loop 사이에 위치)
void startNewRound() {
  // 2번부터 5번 핀 중 하나를 랜덤으로 선택 (최대값 설정은 +1 해서 6)
  targetButton = random(2, 6); 

  // 화면에 목표 버튼 출력
  Serial.print("활성화 해야 할 버튼: ");
  if (targetButton == 2) Serial.println("빨강 (2번)");
  else if (targetButton == 3) Serial.println("노랑 (3번)");
  else if (targetButton == 4) Serial.println("파랑 (4번)");
  else if (targetButton == 5) Serial.println("흰색 (5번)");

  // 기획서의 카운트다운 연출
  Serial.println("3..."); delay(1000);
  Serial.println("2..."); delay(1000);
  Serial.println("1..."); delay(1000);
  Serial.println("cut!!!!!!!!!!!!!!!!!!!");
}

// 3. 메인 게임 루프
void loop() {
  // 라운드 시작 함수 호출
  startNewRound();

  // 판정용 변수 (기본값은 실패인 false)
  boolean success = false; 

  // [핵심] 10ms씩 100번 반복 = 총 1000ms(1초) 동안 대기하며 감시
  for (int i = 0; i < 100; i++) {
    
    // 버튼 상태 실시간 읽기 (평소 1, 누르면 0)
    int red = digitalRead(2);
    int yellow = digitalRead(3);
    int blue = digitalRead(4);
    int white = digitalRead(5);

    // [변경점] 어떤 버튼이라도 '눌렸다면' (즉, 0(LOW)이 되었다면)
    if (red == LOW || yellow == LOW || blue == LOW || white == LOW) {
      
      // 누른 버튼이 하필 랜덤으로 지정된 targetButton과 일치하는가?
      if ((targetButton == 2 && red == LOW) ||
          (targetButton == 3 && yellow == LOW) ||
          (targetButton == 4 && blue == LOW) ||
          (targetButton == 5 && white == LOW)) {
        
        success = true; // 올바른 버튼을 0으로 만들었으므로 성공!
      } else {
        success = false; // 다른 버튼을 0으로 만들었으므로 즉시 오답(실패)!
      }
      
      break; // 버튼 입력이 감지되었으니 1초 감시 루프를 즉시 탈출!
    }

    delay(10); // 0.01초씩 쪼개서 대기
  }

  // --- 1초가 지났거나 중간에 탈출한 후 판정 결과 반영 ---
  if (success == true) {
    Serial.println("-> 성공!\n");
    
    digitalWrite(9, HIGH); // 초록 LED 켜기 (1이 켜짐)
    delay(1000);           // 1초 동안 유지
    digitalWrite(9, LOW);  // 초록 LED 끄기
  } 
  else {
    Serial.println("-> 주민(이)가 플레이어한테 폭사당했습니다.\n");
    
    digitalWrite(10, HIGH); // 빨간 LED 켜기 (1이 켜짐)
    delay(1000);            // 1초 동안 유지
    digitalWrite(10, LOW);   // 빨간 LED 끄기
  }

  // 다음 게임 시작 전 잠깐의 휴식 시간 (3초 대기 후 다음 라운드 무한 반복)
  Serial.println("다음 폭탄이 설치되는 중입니다...");
  delay(3000); 
}
