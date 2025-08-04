# 사용자의 이력서/포트폴리오 분석

> 사용자가 이력서를 올리기만 하면, 자동으로 요약 + 직무 추천까지 한 번에 받을 수 있는 구조

**서비스 플로우**:
1) 사용자가 이력서를 PDF 형태로 업로드 한다.
2) 업로드된 PDF를 로딩하고, 텍스트를 페이지 단위로 나눈 뒤 일정 길이로 청크 분할한다.
3) 각 청크를 Azure OpenAI의 Embedding API로 벡터화한 후 FAISS 벡터 DB에 저장한다.
4) `이 이력서를 요약해주세요`라는 쿼리로 RAG 방식의 요약(변수명: summary)을 생성한다.
5) 생성된 요약을 기반으로 추천 프롬프트를 구성하고, LLM이 적합한 직무 1~2개와 그 근거를 자동 추천한다.

**코드**:
```python
PROMPT_ANALYZE_RESUME = """

다음은 사용자의 이력서/포트폴리오 요약입니다:

  

{resume_summary}

  

이 문서를 바탕으로 사용자가 적합할 수 있는 직무를 1~2개 추천해주세요.

각 직무에 대해, 이 문서가 어떤 강점을 갖고 있는지도 함께 요약해주세요.

  

출력 형식:

- 추천 직무: OOO, OOO

- 강점 요약: ...

"""

  

# 1. PDF 로딩 + 벡터화

def load_resume_pdf(file_path: str):

loader = PyPDFLoader(file_path)

pages = loader.load_and_split()

splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)

docs = splitter.split_documents(pages)

return docs

  

# 2. 임베딩 + 벡터 저장

def build_vector_db(docs):

embeddings = AzureOpenAIEmbeddings(

azure_deployment=settings.EMBEDDING_DEPLOYMENT,

openai_api_key=settings.EMBEDDING_API_KEY,

openai_api_version=settings.EMBEDDING_API_VERSION,

azure_endpoint=settings.EMBEDDING_ENDPOINT,

chunk_size=1_000,

)

db = FAISS.from_documents(docs, embeddings)

return db

  

# 3. 요약 추출 (RAG)

def summarize_resume(db):

llm = AzureChatOpenAI(

deployment_name=settings.LLM_DEPLOYMENT,

openai_api_key=settings.LLM_API_KEY,

api_version=settings.LLM_API_VERSION,

azure_endpoint=settings.LLM_ENDPOINT,

temperature=settings.LLM_TEMPERATURE

)

chain = load_qa_chain(llm, chain_type="stuff")

docs = db.similarity_search("이 이력서를 간단히 요약해주세요.", k=5)

summary = chain.run(input_documents=docs, question="이 이력서를 요약해주세요.")

return summary.strip()

  

# 4. 자동 직무 추천 프롬프트

def suggest_jobs_from_summary(resume_summary):

llm = AzureChatOpenAI(

deployment_name=settings.LLM_DEPLOYMENT,

openai_api_key=settings.LLM_API_KEY,

api_version=settings.LLM_API_VERSION,

azure_endpoint=settings.LLM_ENDPOINT,

temperature=settings.LLM_TEMPERATURE

)

prompt = PROMPT_ANALYZE_RESUME.format(resume_summary=resume_summary)

return llm.predict(prompt).strip()
```

## Before

> 프롬프트 엔지니어링이 적용되기 전의 프롬프트와 산출물을 제시한다.

### Prompt

```
다음은 사용자의 이력서/포트폴리오 요약입니다:

{resume_summary}

이 문서를 바탕으로 사용자가 적합할 수 있는 직무를 1~2개 추천해주세요.
각 직무에 대해, 이 문서가 어떤 강점을 갖고 있는지도 함께 요약해주세요.

출력 형식:
- 추천 직무: 000, 000
- 강점 요약: ...
```

### Output

```

```

## After

> 프롬프트 엔지니어링이 적용된 이후의 프롬프트와 산출물을 제시한다.

```
당신은 취업 및 이직 준비생을 위한 직업 컨설턴트입니다.

요약된 이력서는 다음과 같습니다.
<이력서_요약>
{{resume_summary}}
</이력서_요약>

>> 다음의 가이드라인을 따르세요.
1. 요약된 이력서 문서를 참고하세요.
2. 문서를 바탕으로 사용자에게 적합한 직무를 3개 추천하세요.
3. 각 직무에 대해 사용자의 강점을 분석하여 제시하세요.
```
