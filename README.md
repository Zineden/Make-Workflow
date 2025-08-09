## 📌 사용한 기술 스택
- **Make.com** – 워크플로우 자동화 시나리오 구성
- **Google Sheets API** – 데이터 저장 및 검색
- **HTTP Request 모듈** – 외부 API 호출
- **LLM (gpt-oss-120b)** – 메시지 카테고리 분류 및 멀티라벨링
- **Postman** – API 테스트
- **Webhook / Iterator 모듈** – 다중 JSON 객체 처리
- **Router / Variable 모듈** – 조건 분기 및 변수 관리

---

## 🏗 시스템 아키텍처
본 시스템은 사용자의 메시지를 웹훅(Webhook)으로 수신하여, LLM을 통해 분류하고 Google Sheets에 기록 및 이메일 발송까지 자동화하는 구조로 설계되었습니다.

**간단한 동작 흐름**
1. **Webhook 모듈**에서 요청 수신
2. **Iterator 모듈**로 다중 객체 처리
3. **Google Sheets Search Rows** – 필수 필드 확인 & 중복 방지
4. **LLM API 호출 (gpt-oss-120b)** – 메시지 카테고리 분류
5. **변수 설정 모듈** – 분류 결과 저장
6. **Router 모듈** – 카테고리에 따른 이메일 발송 경로 분기
7. **Google Sheets Update Rows** – 발송 여부 업데이트

**Flowchart**

<p align="center">
  <img width="2860" height="3794" alt="Flowchart" src="https://github.com/user-attachments/assets/f72f3858-3336-43ed-a52c-8a86cd696092" />
  <br>
  <em>Fig 1. Flowchart</em>
</p>

첫 번째 단계로 웹훅(Webhook) 모듈 바로 뒤에 이터레이터(Iterator) 모듈을 배치했습니다. 

<p align="center">
  <img width="543" height="411" alt="Iterator" src="https://github.com/user-attachments/assets/d24bff3a-c85c-4e37-b347-f8093bf9d80d" />
  <br>
  <em>Fig 2. Iterator module</em>
</p>

이터레이터 모듈은 하나의 JSON 안에 있는 여러 객체를 하나씩 받아 처리할 수 있도록 했습니다. 이를 통해 입력 형식을 다양화하고 테스트 과정을 단순화했습니다. 테스트에는 Postman을 사용했고, 여러 상황에 맞춘 테스트 값을 생성하여 적용했습니다.

<p align="center">
  <img width="509" height="455" alt="JSON string" src="https://github.com/user-attachments/assets/542aa827-ed9a-4636-a249-f213f039e8f4" />
  <br>
  <em>Fig 3. JSON string example</em>
</p>

다음으로, Google Sheets의 행 검색(Search Rows) 모듈을 두고 “이름(name)”, “이메일(email)”, “메시지(message)” 중 하나라도 비어 있으면 해당 반복을 건너뛰도록 필터를 설정했습니다. 

<p align="center">
  <img width="450" height="618" alt="Filter settings" src="https://github.com/user-attachments/assets/b7a26e25-8484-46ee-b20d-7dc72bd9c556" />
  <br>
  <em>Fig 4. Filter settings</em>
</p>

데이터가 모두 채워져 있으면, 동일 이메일을 가진 사용자가 동일한 메시지를 이미 보낸 적이 있는지 검색해 중복 문의를 방지했습니다.

<p align="center">
  <img width="856" height="589" alt="Search filter" src="https://github.com/user-attachments/assets/c999ee6b-9336-42bd-a67c-1eb868c91a0b" />
  <br>
  <em>Fig 5. Google Sheets search filter</em>
</p>

<p align="center">
  <img width="737" height="266" alt="Duplicate filter" src="https://github.com/user-attachments/assets/a85b7d7d-753a-4a19-99a0-9336ce0b7131" />
  <br>
  <em>Fig 6. Duplicate Filter</em>
</p>

필요한 데이터는 Google Sheets의 행 추가(Add Rows) 모듈로 전달해 기록했습니다. 이때 “영업 문의”, “기술 지원”, “일반 문의” 필드는 FALSE로 설정했습니다.

<p align="center">
  <img width="735" height="81" alt="Google Sheets entries" src="https://github.com/user-attachments/assets/3c159ccf-2d63-412f-8c4e-42e1f72e3357" />
  <br>
  <em>Fig 7. Google Sheets entries</em>
</p>

그다음 메시지를 HTTP 요청(Make a Request) 모듈로 전달해 URL, 헤더, 요청 방식 등을 설정하고 프롬프트를 작성했습니다. 프롬프트에는 다국어 지원이 가능한 “gpt-oss-120b” 모델을 사용했고, AI가 메시지를 특정 카테고리로 분류하며 JSON 형식으로 출력하고 여러 라벨을 부여할 수 있도록 규칙을 설정했습니다.

<p align="center">
  <img width="1211" height="603" alt="HTTP request" src="https://github.com/user-attachments/assets/58e55ccc-5225-4653-a46d-d980f06c7f77" />
  <br>
  <em>Fig 8. Setting HTTP module request</em>
</p>

출력된 JSON은 make.com 시나리오 내에서 사용할 수 있도록 파싱했습니다. 결과 배열에는 각 카테고리에 대한 Boolean 값과 해당 라벨로 분류된 메시지 부분을 포함했습니다.

<p align="center">
  <img width="821" height="704" alt="JSON parsing" src="https://github.com/user-attachments/assets/42223e35-19bf-4cf7-acd7-11296cb10c9d" />
  <br>
  <em>Fig 9. Parsing output JSON string</em>
</p>

Google Sheets 범위 값 가져오기(Get Range Values) 모듈을 사용해 수신자 이메일 주소를 가져와 make.com의 변수로 정의했습니다. 

<p align="center">
  <img width="781" height="704" alt="Fetching Email addresses" src="https://github.com/user-attachments/assets/c81d343f-52d3-4a59-9149-b75954c96341" />
  <br>
  <em>Fig 10. Fetching Email addresses</em>
</p>

<p align="center">
  <img width="310" height="72" alt="Email addresses" src="https://github.com/user-attachments/assets/793a75ab-e5f6-4f98-bb6c-ddf3384b7df5" />
  <br>
  <em>Fig 11. Email addresses in Google Sheets</em>
</p>

그리고 ‘여러 변수 설정(Set Multiple Variables)’ 모듈에서 “영업 문의”, “기술 지원”, “일반 문의” 값을 라우터 필터 조건에 사용하기 위해 정의했고, 이메일 주소 변수도 저장했습니다.

<p align="center">
  <img width="795" height="684" alt="Setting variables" src="https://github.com/user-attachments/assets/ee925797-f3e1-427b-a9db-f91df9dd0f83" />
  <br>
  <em>Fig 12. Setting variables</em>
</p>

라우터에서는 정의한 변수 값에 따라 경로를 분기했습니다. 예를 들어 “영업 문의” 값이 true라면 영업 문의 관련 요청을 처리하는 경로로 진행했습니다. 다른 문의도 동일한 로직을 적용했습니다.

<p align="center">
  <img width="993" height="684" alt="Router redirect conditions" src="https://github.com/user-attachments/assets/ec962aae-4a10-4997-9641-f362354c37f7" />
  <br>
  <em>Fig 13. Router redirect conditions</em>
</p>

포크된 경로에서는 Google Sheets에서 해당 카테고리에 대응하는 셀을 가져왔습니다. “영업 문의” 필드가 FALSE라면 메시지를 발송하고, 해당 필드를 TRUE로 변경해 동일한 담당자가 여러 카테고리를 맡아도 중복 발송되지 않도록 했습니다.

<p align="center">
  <img width="855" height="455" alt="Email deduplication" src="https://github.com/user-attachments/assets/e1929d30-57a2-41f8-9aad-c5fb59f8e2b3" />
  <br>
  <em>Fig 14. Email deduplication</em>
</p>

이메일 모듈 설정은 다양한 조건을 고려해 구성했습니다. 서로 다른 담당자이거나 동일한 담당자가 여러 문의를 처리하는 경우 모두 대응할 수 있도록 했습니다. 제목과 본문 내용은 상황에 따라 유연하게 변경되도록 했습니다. 현재 경로의 카테고리에 해당하는 내용은 항상 포함하고, 추가 내용은 담당자가 동일한지 여부에 따라 달리했습니다. 담당자가 다르면 다른 경로로 분기해 다른 수신자에게 보냈습니다. 마지막 모듈에서는 필드를 TRUE로 동적으로 변경해 중복 발송을 방지했습니다.

<p align="center">
  <img width="758" height="698" alt="Dynamic Email content" src="https://github.com/user-attachments/assets/210dab83-c704-4ea2-b8d1-2d9a22b60ae3" />
  <br>
  <em>Fig 15. Dynamic Email content</em>
</p>

예를 들어, 영업 문의와 기술 지원 두 카테고리에 모두 해당하는 메시지를 보냈을 때, 담당자의 이메일 주소가 동일하면 하나의 경로만 작동해 두 카테고리에 대한 내용이 포함된 단일 이메일을 발송했습니다. 반대로 담당자가 다르면 각 카테고리에 맞는 경로가 따로 작동해 서로 다른 수신자에게 별도의 이메일을 발송했습니다.

<p align="center">
  <img width="554" height="427" alt="Screenshot 2025-08-09 at 18 35 47" src="https://github.com/user-attachments/assets/75698e52-675e-4160-b623-ccadc5f636c3" />
  <br>
  <em>Fig 16. Same Email</em>
</p>

<p align="center">
  <img width="657" height="334" alt="Screenshot 2025-08-09 at 18 36 25" src="https://github.com/user-attachments/assets/de8e2b7d-dd33-42b6-ac20-0d8c491b8ec4" />
  <br>
  <em>Fig 16. Different Email 1</em>
</p>

<p align="center">
  <img width="555" height="423" alt="Screenshot 2025-08-09 at 18 37 12" src="https://github.com/user-attachments/assets/9dc2ddda-022f-4193-896e-e135f95a7581" />
  <br>
  <em>Fig 17. Different Email 2</em>
</p>






---

## 💡 LLM 프롬프트 설계 고려 사항
- **명확한 출력 형식 지정**  
  JSON 형태로 결과를 반환하도록 명시하여, 파싱 과정에서 오류가 없도록 설계했습니다.
- **다중 카테고리 분류 지원**  
  하나의 메시지가 복수의 카테고리에 속할 수 있도록 라벨링 규칙을 포함했습니다.
- **언어 다양성 고려**  
  한국어/영어 혼합 메시지를 처리할 수 있도록 멀티링구얼 모델 사용.
- **일관성 유지**  
  카테고리명, Boolean 값(`true/false`) 등 출력 형식을 고정.

---

## 🛠 문제와 해결 과정
1. **문제: 중복 메일 발송**  
   - 동일한 담당자가 여러 카테고리를 담당할 경우, 같은 메시지가 두 번 이상 발송됨  
   **해결:** Google Sheets의 Boolean 필드를 활용하여 발송 여부를 체크하고, 발송 후 해당 값을 `TRUE`로 업데이트.

2. **문제: LLM 응답 포맷 불일치**  
   - 모델이 JSON 형식을 완벽히 지키지 않는 경우 발생  
   **해결:** 프롬프트에 엄격한 출력 규칙을 명시하고, 응답 후 유효성 검사 로직 추가.

3. **문제: 테스트 데이터 관리 어려움**  
   - 다양한 케이스에 대한 테스트 반복이 번거로움  
   **해결:** Postman 컬렉션을 만들어 자동으로 여러 테스트 데이터를 전송할 수 있도록 구성.

4. **문제: 다중 객체 처리 시 로직 복잡도 증가**  
   - 하나의 요청에 여러 개 메시지가 들어올 때 필터와 분기 조건이 꼬이는 현상  
   **해결:** Iterator 모듈을 도입해 객체를 개별적으로 처리하고, 각 반복에서 동일한 로직 적용.
