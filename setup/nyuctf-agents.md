# NYU baseline · D-CIPHER 설치 기록

- 담당: 이효관
- 대상: [NYU-LLM-CTF/nyuctf_agents](https://github.com/NYU-LLM-CTF/nyuctf_agents) (baseline과 D-CIPHER가 같은 레포)
- 환경: Windows + WSL2(Ubuntu) + Docker, Python 3.10 이상
- 실제 설치 환경 (2026-11-22): macOS 27.2.1 (Apple Silicon, arm64), Docker 29.6.2, Python 3.12.10, nyuctf_agents 커밋 `612190f`

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
| setup_dcipher.sh | 성공 | 약 24분. `ctfenv:multiagent` 이미지(amd64, 2.67GB), `ctfnet` 네트워크 생성. 빌드 후 `together<2` 재설치 필요 (아래 2번) |
| setup_baseline.sh | 성공 | 약 28분. `ctfenv` 이미지(amd64, 2.91GB), Ghidra 11.0.1은 `docker/baseline/`에 풀림. 빠진 의존성 추가 설치 필요 (아래 3번) |
| nyuctf.download | 성공 | 약 3분 30초. `~/.nyuctf/v20250206` (3.7GB), test 200문제 |
| 최신 모델 등록 (baseline) | 완료 | `model_info.json`에 `claude-haiku-4-5-20251001` 추가, `base_config.yaml`을 anthropic/haiku로 변경. 비용 계산 코드 수정 필요했음 (아래 4번) |
| 모델 설정 (D-CIPHER) | 완료 | 모델은 이미 등록돼 있음. `base_planner_executor.yaml`의 planner·executor·autoprompter 모델을 haiku로 변경 |
| 스모크 테스트 | 미실시 | 이번에는 하지 않음. 두 에이전트 모두 import와 `--help` 실행까지만 확인 |

### 실행 전 준비: API 키

`nyuctf_agents/keys.cfg`에 아래 한 줄을 넣으면 baseline과 D-CIPHER 둘 다 읽는다. 이 파일은 nyuctf_agents의 `.gitignore`에 들어 있고, 팀 레포에는 절대 넣지 않는다.

```
ANTHROPIC=<API 키>
```

- D-CIPHER는 `keys["ANTHROPIC"]`로 찾는다 (`run_dcipher.py`, `nyuctf_multiagent/utils.py`의 `APIKeys`). `ANTHROPIC_API_KEY=`로 쓰면 못 읽는다.
- baseline은 `nyuctf_baseline/backends/utils.py`의 `parse_keys()`가 왼쪽 이름 뒤에 `_API_KEY`를 붙여서 `ANTHROPIC_API_KEY`로 읽는다. 키 파일이 없으면 환경변수 `ANTHROPIC_API_KEY`, `~/.config/anthropic/api_key` 순서로 찾는다.

### 실행 시 참고

- D-CIPHER는 `--config`를 안 주면 `configs/dcipher/<카테고리>_planner_executor.yaml`을 자동으로 쓴다. `base_planner_executor.yaml`만 바꾸면 카테고리 설정에는 반영되지 않는다. 모델과 비용은 옵션으로 덮어쓴다:

  ```bash
  python3 run_dcipher.py --split test --challenge 2018f-msc-leaked_flag \
    --planner-model claude-haiku-4-5-20251001 --executor-model claude-haiku-4-5-20251001 \
    --autoprompter-model claude-haiku-4-5-20251001 --max-cost 0.5 --keys keys.cfg
  ```

- baseline은 설정 파일의 `max_cost`가 명령줄 `--max-cost`보다 우선한다 (`run_baseline.py`가 config 값으로 덮어씀). 비용 상한은 `base_config.yaml`에서 바꿔야 한다.

## 겪은 문제와 해결

1. **macOS(arm64)에서 이미지 빌드**: 두 Dockerfile 모두 x86_64 전용(radare2 amd64 deb, i386 라이브러리, `/usr/lib/x86_64-linux-gnu` 경로)이다. `DOCKER_DEFAULT_PLATFORM=linux/amd64`를 설정하고 setup 스크립트를 실행해서 amd64 이미지로 빌드했다. 에뮬레이션이라 빌드가 느리다. `setup_baseline.sh`는 `wget`을 쓰는데 macOS에는 기본으로 없어서 `brew install wget`으로 설치했다.
   - 주의: amd64 에뮬레이션에서는 gdb 같은 ptrace 기반 도구가 제대로 안 될 수 있다. pwn·rev 본 실험은 팀 공용 x86 PC(WSL2)에서 돌리는 게 안전하다.
2. **D-CIPHER import 실패 (`No module named 'together.error'`)**: `pip install --editable .`이 최신 `together` 3.x를 설치하는데, 3.x에는 `together.error` 모듈이 없다. `pip install "together<2"`로 1.6.0을 설치해서 해결했다.
3. **baseline 의존성 누락**: `requirements.txt`에서 `ToolDefGenerator`, `jinja2`, `bs4`, `lxml`, `ruamel.yaml`이 주석 처리돼 있는데 코드가 import한다. 직접 설치했다:

   ```bash
   pip install "ToolDefGenerator @ git+https://github.com/moyix/ToolDefGenerator@main" jinja2 bs4 lxml ruamel.yaml
   ```

4. **baseline 비용 계산 코드가 최신 SDK와 안 맞음**: `anthropic_backend.py`의 `send()`가 `self.client.count_tokens()`를 호출하는데, 설치된 anthropic 2.12.0에는 이 함수가 없어서 첫 응답 직후 죽는다. API 응답의 `response.usage`(input/output 토큰)로 비용을 계산하도록 고쳤다. 원래 코드는 마지막 메시지만 세서 비용이 실제보다 적게 나왔는데, 수정 후에는 매 호출의 전체 프롬프트 토큰이 반영된다. 로그 JSON에 `input_tokens`, `output_tokens`도 남기게 했다 (`conversation.py`).
5. **baseline은 `keys.cfg`가 없으면 import 단계에서 죽는다**: `parse_keys()`가 `FileNotFoundError`가 아니라 `FileExistsError`를 잡고 있어서다. 환경변수로만 키를 줄 때도 빈 `keys.cfg`는 있어야 한다.
6. **baseline 응답 길이 고정**: `anthropic_backend.py`의 `_call_model()`이 `max_tokens=1024`로 고정돼 있어서 `model_info.json`의 `max_tokens`(8192)는 실제로 쓰이지 않는다. D-CIPHER는 설정 파일의 `max_tokens: 4096`을 쓴다. 본 실험 전에 맞출지 팀에서 정해야 한다.
