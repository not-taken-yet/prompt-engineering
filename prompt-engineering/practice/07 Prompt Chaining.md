## 실습 1 - 인용물 추출하기

```
너는 나의 어시스턴트야. 너의 임누는 문서에 주어진 질문에 답하는 거야. 첫번째 단계는 ####로 구분된 문서에서 질문과 관련된 인용문을 추출하는거야.

인용문 목록을 <qutoes></quotes> 태그로 출력해줘.
관련된 인용문을 찾지 못하면 "No relevant quotes found!"라고 응답해

####

{{document}}

####

질문: 앤트로픽 회사에 대한 문장
```

## 실습 2 - (1) 결과물 이용해서 답변 생성하기

```
Here is a document, in <document></document> XML tags:

<document>
{{DOCUMENT}}
</document>

please extract, word-for-word, any qutoes relevant to the question {{QUESTION}}. Please enclose the full list of quotes in <qutoes></quotes> XML tags. If there are no qutoes in this document that seem relevant to this question, please say "I can't find any relevant quotes"
```

```
문서에서 가져온 관련 인용구를 사용하여 질문에 답변해.

여기 문서가 있어:
<document>
{{DOCUMENT}}
</document>

여기 질문과 가장 관련 있는 문서의 직접 인용구야:
<quotes>
{{QUOTES}}
</quotes>

이것들을 사용하여 "{{QUESTION}}"에 대한 답변을 구성해.
답변이 정확하고 인용구에 직접적으로 뒷받침되지 않는 정보는 포함하지 않아.

{{QUESTION}}: 앤트로픽은 어떤 회사인가요?
```

## 실습 3 

```
너는 나의 어시스턴트야. 너의 임무는 문서에 주어진 질문에 답하는거야.

첫 번째 단계는 ###로 구분된 문서에서 질문과 관련된 인용문을 추출하는거야.

인용문 목록을 <quotes></quotes> 태그로 출력해.

관련된 인용문을 찾지 못하면 "No relevant qutoes found!"라고 응답해

### 문서: 
{{
	관련된 문서의 텍스트
}}

### 질문: 앤트로픽에 대한 회사 소개

ㅡㅡㅡ
이것들을 사용하여 "{{QUESTION}}"에 대한 답변을 구성해.
답변이 정확하고 인용구에 직접적으로 뒷받침되지 않는 정보는 포함하지 말아.

{{QUESTION}}: 앤트로픽은 어떤 회사인가요?
```

## 실습4

> 프롬프트 체이닝을 사용하여 아래 텍스트의 사용자의 의도(User Intent)를 분류하고, 사용자의 프롬프트를 보완해주는 프롬프트를 생성하는 프롬프트를 작성해보세요.

(1) 사용자 질문에서 **의도 분류**하기
(2) 사용자 프롬프트 보완하는 프롬프트 제작하기

```
(1) Your task is to extract "User intent" in the given text. The Intent should be an action formation. Only extract the verb.

[Text: Python에 scipy 라이브러리에서 freqz와 welch 함수의 차이점과 사용법을 예시코드를 작성해서 설명해줘]

{Intent}: {  }
```

```
(2) By uusing the extracted intent, please elaborate on the [user prompt] to achieve a better response. When elaborating, you may use roles, specific tasks, and some contexts. Respond in Korean.

[user prompt] Python에 scipy 라이브러리에서 freqz와 weich 함수의 차이점과 사용법을 예시코드를 작성해서 설명해줘.
```