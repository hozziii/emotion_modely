# emotion_modely

KoBERT 기반 한국어 감정 분류의 Python 추론 코드를 담은 저장소입니다.
문장을 입력하면 가장 높은 점수의 감정 라벨과 softmax 점수를 반환합니다.

## 구현 내용

- Hugging Face의 `monologg/kobert` 토크나이저 로딩
- 로컬 `kobert_emotion_final/`에서 분류 모델 로딩
- 최대 128토큰으로 입력을 처리하고 `model.eval()` 및 `torch.no_grad()`로 추론
- 모델 설정의 라벨 매핑에 따라 `anger`, `sad`, `fear` 중 하나와 점수 반환

반환 점수는 모델의 softmax 출력이며, 실제 정답 확률로 검증된 값은 아닙니다.
현재 코드는 중립 클래스나 신뢰도 임계값을 구현하지 않습니다.

## 파일 구성

| File | Purpose |
| --- | --- |
| `emotion_infer.py` | 재사용 가능한 `predict_emotion(text)` 함수 |
| `test_inference.py` | 네 개의 예문을 출력하는 수동 실행 스크립트 |
| `kobert_emotion_final/config.json` | 모델 구조 및 세 개 감정 라벨 설정 |
| `tokenization_kobert.py` | 저장소에 포함된 토크나이저 구현 |

현재 추론 스크립트는 로컬 토크나이저 파일을 직접 지정하지 않고 Hugging Face의 토크나이저를 불러옵니다.

## 실행 전 준비

**공개 저장소에는 모델 가중치가 포함되어 있지 않습니다.**
현재 파일만 복제하면 모델 로딩 단계에서 실패합니다.
호환되는 학습 완료 가중치를 별도로 확보해 `kobert_emotion_final/`에 배치해야 합니다
(예: `model.safetensors` 또는 `pytorch_model.bin`). 이 저장소는 가중치 다운로드 경로를 제공하지 않습니다.

Python 환경에 PyTorch, Transformers 및 KoBERT 토크나이저가 요구하는 의존성이 필요합니다.
고정된 의존성 목록과 재현 검증된 설치 환경은 제공되지 않습니다.
모델 설정에는 `transformers_version: 4.57.3`이 기록되어 있지만, 이를 실행 환경 검증으로 볼 수는 없습니다.

토크나이저 최초 로딩에는 네트워크 연결 또는 사전 캐시가 필요합니다.
코드의 `trust_remote_code=True`는 원격 토크나이저 코드를 실행할 수 있으므로 해당 코드를 검토한 환경에서 사용하세요.

## 사용 예시

가중치와 의존성을 준비한 뒤 저장소 루트에서 실행합니다.

```python
from emotion_infer import predict_emotion

label, score = predict_emotion("배송이 너무 늦어요")
print(label, score)
```

샘플 문장 출력:

```bash
python test_inference.py
```

`test_inference.py`는 정답 비교나 assertion이 없는 수동 확인용 스크립트입니다.
이 문서에서는 실행 결과나 정확도 수치를 주장하지 않습니다.

## 문제 확인

| 증상 | 확인할 내용 |
| --- | --- |
| 모델을 불러오지 못함 | `kobert_emotion_final/` 내 가중치 존재 여부와 설정 호환성 |
| 토크나이저 로딩 실패 | 네트워크·캐시, 토크나이저 의존성 및 Transformers 호환성 |
| 중립 문장도 감정으로 분류됨 | 현재 모델은 세 클래스 중 하나를 선택하며 중립 처리가 없음 |

## 범위와 관련 프로젝트

이 저장소는 감정 추론 코드를 보여주며, REST API 서버나 Carely 전체 서비스는 포함하지 않습니다.
학습 데이터, 학습 절차, 평가 지표도 이 README의 검증 범위에 포함되지 않습니다.

Carely의 서비스 연동 경험은 [개인 포트폴리오](https://github.com/hozziii/Carely)와
[팀 AI 백엔드](https://github.com/sswu-gen-ai-18/AI-BE)에서 확인할 수 있습니다.

