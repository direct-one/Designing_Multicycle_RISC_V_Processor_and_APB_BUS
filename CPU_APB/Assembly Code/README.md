# Assembly Code

Multicycle RISC-V CPU에서 실행할 프로그램을 32비트 Assembly Code로 변환한 `.mem` 파일이 저장되어 있습니다.

## FLOW

```text
C 언어 프로그램
    ↓ 컴파일
RISC-V Assembly Code
    ↓ 어셈블
32-bit RV32I Machine Code (.mem)
    ↓ $readmemh
Instruction Memory
    ↓
Multicycle RISC-V CPU
    ↓
APB Master → APB Peripheral
```

각 `.mem` 파일은 한 줄에 하나의 32비트 명령어를 16진수로 기록합니다. Instruction Memory는 파일의 첫 번째 줄부터 순서대로 명령어를 읽어 CPU에 제공합니다.

## Test MEM file

| FILE | EXPLANATION |
|---|---|
| `ABP_BRAM_GPO_GPI.mem` | BRAM, GPO, GPI를 함께 확인하기 위한 APB 연동 프로그램입니다. |
| `APB_FND.mem` | APB FND 장치를 제어하기 위한 프로그램입니다. |
| `APB_GPIO_LED_BLINK.mem` | APB GPIO를 출력으로 설정하고 LED를 반복 점멸시키는 프로그램입니다. |
| `APB_GPO.mem` | APB GPO 출력 동작을 확인하기 위한 프로그램입니다. |
| `APB_TEST_C.mem` | C 코드로 정의한 APB 통합 동작을 기계어로 변환한 테스트 프로그램입니다. |
| `APB_UART.mem` | APB UART의 송수신 또는 레지스터 접근을 시험하기 위한 프로그램입니다. |
| `U_DMEM.mem` | CPU의 Data Memory 접근 동작을 확인하기 위한 프로그램입니다. |
| `riscv_rv32i_rom_data.mem` | RV32I 명령어의 기본 동작을 확인하기 위한 프로그램 이미지입니다. |
| `riscv_rv32i_rom_data_sum.mem` | 산술 연산과 누적 합 동작을 확인하기 위한 프로그램 이미지입니다. |


## 담당 구현: `APB_GPIO_LED_BLINK.mem`

`APB_GPIO_LED_BLINK.mem`은 본 프로젝트에서 직접 담당하여 제작한 실행 이미지입니다. C 언어로 LED 점멸 동작을 정의한 뒤 RISC-V Assembly Code와 RV32I 기계어로 변환하여 `.mem` 파일로 저장했습니다.

프로그램의 핵심 동작은 다음과 같습니다.

1. GPIO 제어 레지스터를 설정하여 LED 연결 핀을 출력 모드로 구성합니다.
2. GPIO 출력 데이터 레지스터에 LED ON/OFF 패턴을 기록합니다.
3. 소프트웨어 반복문으로 일정한 지연 시간을 만듭니다.
4. ON/OFF 기록과 지연 동작을 반복하여 LED가 점멸하도록 합니다.

이 프로그램을 통해 다음 전체 경로를 검증할 수 있습니다.

```text
APB_GPIO_LED_BLINK.mem
    ↓
Instruction Memory
    ↓
RISC-V CPU의 명령어 실행 및 Store 명령 처리
    ↓
APB Master의 IDLE → SETUP → ACCESS 상태 전이
    ↓
GPIO Slave 선택 및 레지스터 쓰기
    ↓
LED 출력 변화
```

GPIO의 메모리 맵은 다음과 같습니다.

| 레지스터 | 주소 | 기능 |
|---|---:|---|
| `GPIO_CTL` | `0x2000_2000` | GPIO 입출력 방향을 설정합니다. |
| `GPIO_ODATA` | `0x2000_2004` | GPIO 출력 데이터를 기록합니다. |
| `GPIO_IDATA` | `0x2000_2008` | GPIO 입력 데이터를 읽습니다. |

따라서 LED 점멸 프로그램은 주로 `GPIO_CTL`과 `GPIO_ODATA`에 대한 Store 명령으로 APB 쓰기 전송을 발생시킵니다.

## Selecting Simulation

Instruction Memory의 `$readmemh` 대상 파일을 변경하면 실행할 프로그램을 선택할 수 있습니다.

시뮬레이터의 작업 디렉터리가 `CPU_APB`일 경우:

```systemverilog
initial begin
    $readmemh("Assembly Code/APB_GPIO_LED_BLINK.mem", rom);
end
```

시뮬레이터의 작업 디렉터리가 저장소 최상위 폴더일 경우:

```systemverilog
initial begin
    $readmemh("CPU_APB/Assembly Code/APB_GPIO_LED_BLINK.mem", rom);
end
```

경로에 공백이 포함되어 있으므로 시뮬레이터가 해당 경로를 올바르게 처리하는지 확인해야 합니다. 필요하면 선택한 `.mem` 파일을 시뮬레이션 작업 디렉터리에 복사하고 파일명만 지정할 수도 있습니다.

## Verificate Signal 

`APB_GPIO_LED_BLINK.mem` 실행 시 다음 신호를 함께 관찰하면 CPU와 APB GPIO의 연동을 확인하기 쉽습니다.

- CPU: Program Counter, 현재 명령어, ALU 결과, Store 데이터
- APB: `PADDR`, `PWRITE`, `PSEL`, `PENABLE`, `PWDATA`, `PREADY`
- GPIO: 방향 제어 레지스터, 출력 데이터 레지스터, LED 출력

정상 동작에서는 GPIO 주소가 출력되고, APB SETUP/ACCESS 단계가 진행된 후 `PREADY`와 함께 쓰기가 완료되며 LED 출력값이 반복해서 변경됩니다.

