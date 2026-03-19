# 언리얼 엔진 멀티플레이어 숫자 야구 (Number Baseball)

## 프로젝트 소개
이 프로젝트는 언리얼 엔진(Unreal Engine) C++를 기반으로 제작된 멀티플레이어 숫자 야구 게임입니다. 클라이언트-서버 구조를 채택하여 서버가 정답을 관리하고, 클라이언트(플레이어)들이 채팅 UI를 통해 숫자를 입력하여 정답을 맞히는 방식으로 진행됩니다.

사용자는 입력한 숫자에 대한 서버의 판별 결과(Strike/Ball)를 제공받으며, 정해진 횟수 내에 먼저 정답을 맞히는 플레이어가 승리하게 됩니다.

---

## 주요 구현 과정 및 클래스 설명 (현재 구현 상태)

현재 프로젝트는 핵심적인 게임 로직과 네트워크 통신 구조의 구현이 완료된 상태입니다. 각 클래스별 구현 과정과 역할은 다음과 같습니다.

### 1. UI 및 사용자 입력 처리 (NBChatInput)
* **역할**: 사용자가 숫자를 입력하는 채팅 UI 위젯입니다.
* **구현 상세**: `UEditableTextBox`를 통해 입력을 받으며, 사용자가 Enter 키를 누르면 `OnChatInputTextCommitted` 함수에서 `ETextCommit::OnEnter` 이벤트를 감지합니다. 이후 현재 위젯을 소유 중인 플레이어 컨트롤러(`ANBPlayerController`)로 입력된 텍스트 데이터를 전달하도록 구현되었습니다.

### 2. 플레이어 컨트롤러 및 네트워크 통신 (NBPlayerController)
* **역할**: 클라이언트의 UI 입력값을 서버로 전달하고, 서버의 데이터를 클라이언트 화면과 연결합니다.
* **구현 상세**:
    * UI에서 전달받은 메시지를 서버로 전송하기 위해 Server RPC(`ServerRPCPrintChatMessageString`)를 활용합니다.
    * 서버에 전송된 메시지는 현재 게임을 관장하는 `ANBGameModeBase`로 전달됩니다.
    * 클라이언트 화면(블루프린트 UI)에 안내 문구를 띄울 수 있도록 리플리케이트(Replicated) 프로퍼티인 `NotificationText` 변수가 구현되어 있습니다.

### 3. 게임 규칙 및 핵심 로직 (NBGameModeBase)
* **역할**: 게임의 룰(정답 생성, 판별, 횟수 증가, 승리 처리)을 관장하는 서버 전용 클래스입니다.
* **구현 상세**:
    * `GenerateSecretNumber()`: 1~9 사이의 숫자를 이용해 게임의 정답인 비밀 숫자를 생성합니다.
    * `PrintChatMessageString()`: 플레이어가 입력한 문자열이 올바른 숫자인지 검사(`IsGuessNumberString`)하고, 스트라이크/볼 결과를 판별(`JudgeResult`)합니다.
    * `IncreaseGuessCount()`: 플레이어가 숫자를 입력할 때마다 해당 플레이어의 도전 횟수를 증가시킵니다.
    * `JudgeGame()`: 3 스트라이크(정답) 달성 시 전체 플레이어의 `NotificationText`를 업데이트하여 누가 승리했는지 화면에 알립니다.

### 4. 플레이어 상태 및 정보 동기화 (NBPlayerState)
* **역할**: 각 플레이어의 고유 정보와 점수, 진행 상황을 저장하고 모든 클라이언트에 동기화합니다.
* **구현 상세**:
    * 플레이어의 이름(`PlayerNameString`)과 현재 도전 횟수(`CurrentGuessCount`), 최대 도전 횟수(`MaxGuessCount`, 기본값 3)를 리플리케이트하여 클라이언트-서버 간 데이터 일관성을 유지합니다.
    * `GetPlayerInfoString()`을 통해 "플레이어이름(현재횟수/최대횟수)" 형태의 문자열 포맷팅 기능을 제공합니다.

### 5. 게임 상태 및 멀티플레이어 방송 (NBGameStateBase)
* **역할**: 게임 전반의 상태를 관리하고 공지사항을 방송(Broadcast)합니다.
* **구현 상세**: Multicast RPC(`MulticastRPCBroadcastLoginMessage`)를 사용하여, 새로운 플레이어가 게임에 접속했을 때 권한(Authority)이 없는 클라이언트 화면에도 플레이어 참여 알림 문구가 출력되도록 처리되어 있습니다.
