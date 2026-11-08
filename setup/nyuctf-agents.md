# NYU baseline · D-CIPHER 설치 기록

- 담당: 이효관
- 대상: [NYU-LLM-CTF/nyuctf_agents](https://github.com/NYU-LLM-CTF/nyuctf_agents) (baseline과 D-CIPHER가 같은 레포)
- 환경: Windows + WSL2(Ubuntu) + Docker, Python 3.10 이상

## 설치 순서

```bash
git clone https://github.com/NYU-LLM-CTF/nyuctf_agents.git
cd nyuctf_agents
python3 -m venv .venv && source .venv/bin/activate

# D-CIPHER: ctfnet 네트워크 + ctfenv:multiagent 이미지 빌드 + 패키지 설치
./setup_dcipher.sh

# baseline: ctfenv 이미지 빌드 + Ghidra 11.0.1 다운로드 + 의존성 설치
./setup_baseline.sh

# NYU CTF Bench 데이터셋 다운로드 (세 에이전트 공용)
python3 -m nyuctf.download
```

## 실행 예시 (스모크 테스트)

```bash
# D-CIPHER
python3 run_dcipher.py --split test --challenge <문제이름> [--enable-autoprompt]

# baseline
python3 run_baseline.py -c configs/baseline/base_config.yaml --split test --challenge <문제이름>
```

## 코드 분석 메모

- **모델 지원 범위가 다름**: D-CIPHER(`nyuctf_multiagent/backends/anthropic_backend.py`)는 최신 Claude 모델까지 등록되어 있지만, baseline(`nyuctf_baseline/backends/model_info.json`)은 `claude-3-5-sonnet-20240620`까지만 있다. 같은 모델로 비교하려면 baseline에 모델 정보를 추가해야 한다.
- **기본 설정이 서로 다름**:
  - baseline `base_config.yaml`: `max_cost: 3`, `max_rounds: 3`, 기본 모델 `gpt-4o-2024-05-13`
  - D-CIPHER `base_planner_executor.yaml`: `max_cost: 1.0`, 계획자 30라운드 / 실행자 100라운드 / 자동 프롬프트 5라운드, 기본 모델 `gpt-4o-2024-11-20`
  - → 실험 전에 비용 상한과 모델을 `docs/experiment-rules.md` 기준으로 맞춰야 한다.
- **카테고리별 설정**: D-CIPHER는 `configs/dcipher/`에 카테고리별(pwn, rev, crypto, web, forensics, misc) 프롬프트·설정이 따로 있다.
- **ablation용 스크립트 제공**: `run_single_executor.py`는 계획자 없이 실행자만 돌린다. "역할 분할의 효과" 측정에 바로 쓸 수 있다.

## 설치 결과

| 단계 | 결과 | 비고 |
| --- | --- | --- |
| setup_dcipher.sh | | |
| setup_baseline.sh | | |
| nyuctf.download | | |
| 최신 모델 등록 (baseline) | | |
| 스모크 테스트 | | |

## 겪은 문제와 해결

-
