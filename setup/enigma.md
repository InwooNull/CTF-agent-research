# EnIGMA 설치 기록

- 담당: 송인우
- 대상: SWE-agent v0.7 (EnIGMA 모드)
- 환경: Windows + WSL2(Ubuntu) + Docker

> SWE-agent 문서는 Windows 네이티브 설치에 문제가 있다고 안내하므로 WSL2 안에서 진행한다.

## 설치 순서

```bash
# 1. 코드 받기 (v0.7 고정)
git clone -b v0.7 https://github.com/SWE-agent/SWE-agent.git
cd SWE-agent

# 2. 가상환경 + 설치 (Python 3.9 이상)
python3 -m venv .venv && source .venv/bin/activate
python -m pip install --upgrade pip && pip install --editable .

# 3. EnIGMA용 Docker 이미지와 네트워크
docker pull sweagent/enigma:latest
docker network create ctfnet   # 이미 있으면 생략

# 4. API 키: 레포 루트에 keys.cfg 생성 (커밋 금지)
#    ANTHROPIC_API_KEY: '...'
```

## 실행 예시 (스모크 테스트)

NYU CTF Bench의 쉬운 문제(`leaked_flag`, 설명에 플래그가 노출된 문제)로 동작만 확인한다.

```bash
python run.py \
  --model_name <모델명> \
  --ctf \
  --image_name sweagent/enigma:latest \
  --data_path ../NYU_CTF_Bench/test/2018/CSAW-Finals/misc/leaked_flag/challenge.json \
  --repo_path ../NYU_CTF_Bench/test/2018/CSAW-Finals/misc/leaked_flag/ \
  --config_file config/default_ctf.yaml \
  --per_instance_cost_limit 1.00
```

- 실행 결과는 `trajectories/` 아래 trajectory 파일로 저장된다 (풀이 기록 분석에 사용).
- 같은 문제를 다시 돌리려면 `--skip_existing=False`.

## 코드 분석 메모

- **지원 모델이 구버전에 고정됨**: `sweagent/agent/models.py`의 `AnthropicModel.MODELS`에 등록된 Claude 모델은 `claude-3-5-sonnet-20240620`까지다. 최신 모델을 쓰려면 이 딕셔너리에 모델명, `max_context`, `max_tokens`, 토큰당 비용을 추가해야 한다.
- **비용 제한**: `--per_instance_cost_limit`(문제당), `--total_cost_limit`(전체)으로 설정. 비용은 위 딕셔너리의 단가로 계산되므로, 모델 추가 시 단가를 정확히 넣어야 비용 비교가 맞는다.
- **문제 지정 방식**: `challenge.json`(이름, 설명, 카테고리, 파일, 서버 정보)과 문제 폴더를 직접 넘긴다. 폴더에 `docker-compose.yml`이 있으면 서버형 문제(web 등)용 컨테이너를 같이 띄운다.

## 설치 결과

| 단계 | 결과 | 비고 |
| --- | --- | --- |
| clone + pip install | 완료 | v0.7 브랜치, venv 환경 (2026-11-08) |
| docker 이미지 / ctfnet | 완료 | sweagent/enigma:latest, ctfnet 네트워크 생성 |
| 라이브러리 버전 고정 | 완료 | `modal<1.0`, `anthropic[bedrock]==0.49.0` (2026-11-22, 아래 1·2번) |
| 최신 모델 등록 | 완료 | `claude-haiku-4-5-20251001`, `claude-sonnet-4-6` 추가 + `top_p` 미전송 수정. 수정 내용은 `setup/patches/enigma_v0.7.patch` |
| keys.cfg 설정 | 완료 | SWE-agent 폴더에만 둠 (팀 레포에 커밋하지 않음) |
| 스모크 테스트 | 성공 | leaked_flag, `ctf_misc.yaml` 설정, $0.08. 결과는 `runs/smoke/enigma.md` |

## 재현 방법 (설치 이후)

```bash
cd SWE-agent && source .venv/bin/activate
pip install "modal<1.0" "anthropic[bedrock]==0.49.0"
git apply ../ctf-agent-comparison/setup/patches/enigma_v0.7.patch
```

실행할 때는 반드시 카테고리별 설정(`config/ctf/ctf_<카테고리>.yaml`)을 쓴다. 기본 설정(`config/default_ctf.yaml`)에는 EnIGMA의 핵심 기능인 요약기와 풀이 예시가 없다.

```bash
python run.py --model_name claude-haiku-4-5-20251001 --ctf \
  --image_name sweagent/enigma:latest \
  --data_path <문제폴더>/challenge.json --repo_path <문제폴더>/ \
  --config_file config/ctf/ctf_misc.yaml --per_instance_cost_limit 0.50
```

## 겪은 문제와 해결

1. **`modal` 버전 충돌**: `swebench`(2.x)가 불러오는 `modal`이 최신 1.x로 설치되어 `modal.Mount`가 없다는 오류(`AttributeError`) 발생. `pip install "modal<1.0"`으로 해결. 실행할 때 나오는 `DeprecationError` 경고는 무시해도 됨.
2. **`anthropic` SDK 버전 충돌**: 최신 SDK(2.x)에서 `AI_PROMPT`, `HUMAN_PROMPT`가 삭제되어 `ImportError` 발생. `anthropic[bedrock]==0.49.0`으로 고정해 해결. 이 버전으로도 최신 모델 호출은 정상 동작함.
3. **최신 모델 미등록 + API 호환성**: `sweagent/agent/models.py`의 모델 목록이 Claude 3.5 Sonnet까지만 있음. 최신 모델과 토큰 단가를 추가했고, 최신 Claude 모델은 `temperature`와 `top_p`를 함께 보내면 오류가 날 수 있어 Claude 3 계열이 아닐 때는 `top_p`를 보내지 않도록 수정함.
4. **API 키가 빈 값으로 저장됨**: zsh에서는 `read -p`가 bash와 다르게 동작해 키가 저장되지 않았고, 인증 오류(`Could not resolve authentication method`)가 남. `keys.cfg`를 다시 작성해 해결. 키 길이를 확인하는 명령으로 검증할 것.
5. **기본 설정으로 실행하면 비용이 폭증**: `default_ctf.yaml`로 실행했을 때 에이전트가 `find /opt -type f`로 38만 자를 출력했고, 이 출력이 이후 호출마다 함께 전송되어 비용 한도를 넘김. 카테고리별 설정(`ctf_misc.yaml`)은 긴 출력을 요약하는 LM Summarizer가 있어 같은 문제를 $0.08로 풀었음. 자세한 비교는 `runs/smoke/enigma.md`.
