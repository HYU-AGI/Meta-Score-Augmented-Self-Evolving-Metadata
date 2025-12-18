# Meta-Score-Augmented-Self-Evolving-Metadata
## 생성 결과에 대해 Meta Score를 결합하여 자기학습을 위한 Meta Data

### 💡 예시
![image](./img/example.png)

## 📁 Data Structure

이 데이터셋은 수학 문제 해결을 위한 메타 데이터를 포함하고 있습니다.

- **REF/**: Reference dataset - 모델 학습 및 참조용 데이터
- **TEST/**: Test dataset - 모델 평가용 데이터
- **AHSME/**: American High School Mathematics Examination 데이터
- **AIME/**: American Invitational Mathematics Examination 데이터
- **AMC/**: American Mathematics Competition 데이터

각 폴더는 여러 모델(Mathstral-7B, Deepseek-math-7b-rl, Qwen-2.5-math-7B, OpenMath2-Llama3.1-8B, OREAL-7B)의 결과를 포함합니다.

### 파일명 규칙

파일명은 `{DATASET}_{MODEL}.json` 형식을 따릅니다. 예: `REF_Mathstral-7B.json`
- 폴더명이 파일명에 포함되어 있어 파일 이동 시 출처를 쉽게 식별할 수 있습니다.

## 📄 JSON File Format

각 JSON 파일은 다음과 같은 필드를 포함합니다:

```json
{
  "competition_id": "문제 출처 (예: 2016_AMC_8_Problems)",
  "problem_id": "문제 번호",
  "difficulty": "난이도 (숫자)",
  "prompt": "모델에 입력된 프롬프트",
  "response": "모델이 생성한 답변",
  "CLAWS": [
    "평가 메트릭 배열 (5개 값)",
    "각 값은 특정 평가 기준을 나타냄"
  ],
  "Computed_metrics": {
    "perplexity": "언어 모델의 perplexity 값",
    "window_entropy": "윈도우 엔트로피",
    "logit_entropy": "로짓 엔트로피",
    "hidden_score": "hidden state 기반 점수",
    "attention_score": "attention 기반 점수"
  },
  "LLM_evaluator": {
    "Gemini-1.5-pro": {
      "correctness": "정답 여부 (YES/NO)",
      "coarse_grained_novelty": "창의성 여부 (YES/NO)"
    },
    "o4-mini": {
      "correctness": "정답 여부 (YES/NO)",
      "coarse_grained_novelty": "창의성 여부 (YES/NO)"
    }
  },
  "label": "최종 라벨 (Creative_Solution, Typical_Solution, Hallucinated_Solution)"
}
```

### 필드 상세 설명

- **CLAWS**: 5개 값으로 구성된 평가 메트릭 배열. 각 값은 솔루션의 품질을 다각도로 평가합니다.
- **Computed_metrics**: 자동으로 계산된 정량적 메트릭
  - `perplexity`: 모델의 언어 생성 품질
  - `window_entropy`: 텍스트의 다양성
  - `logit_entropy`: 출력 확률의 불확실성
  - `hidden_score`: 내부 표현의 품질
  - `attention_score`: 주의 메커니즘 기반 점수
- **LLM_evaluator**: LLM을 사용한 정성적 평가
  - `correctness`: 답변의 정확성
  - `coarse_grained_novelty`: 솔루션의 창의성
- **label**: 최종 분류 레이블
  - `Creative_Solution`: 창의적인 해법
  - `Typical_Solution`: 일반적인 해법
  - `Hallucinated_Solution`: 오답 또는 환각된 해법

## 💻 Usage

### Python에서 데이터 로드하기

```python
import json

# JSON 파일 로드
with open('REF/REF_Mathstral-7B.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

# 첫 번째 항목 확인
first_item = data[0]
print(f"Problem ID: {first_item['problem_id']}")
print(f"Label: {first_item['label']}")
print(f"Perplexity: {first_item['Computed_metrics']['perplexity']}")

# Creative Solution만 필터링
creative_solutions = [item for item in data if item['label'] == 'Creative_Solution']
print(f"Total creative solutions: {len(creative_solutions)}")
```

### 데이터 분석 예시

```python
import pandas as pd

# DataFrame으로 변환
df = pd.DataFrame(data)

# 레이블별 분포 확인
print(df['label'].value_counts())

# 난이도별 평균 perplexity
df['perplexity'] = df['Computed_metrics'].apply(lambda x: x['perplexity'])
print(df.groupby('difficulty')['perplexity'].mean())
```
