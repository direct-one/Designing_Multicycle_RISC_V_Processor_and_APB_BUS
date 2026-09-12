# Define Code

이 폴더에는 Multicycle RISC-V CPU의 명령어 해석과 제어 신호 생성에 공통으로 사용하는 Verilog 매크로 정의 파일이 저장되어 있습니다.

현재 구성 파일은 다음과 같습니다.

| FILE | EXPLANATION |
|---|---|
| `define.vh` | RV32I Opcode, ALU 연산, Branch 조건 등에 사용하는 공통 매크로를 정의합니다. |

## `define.vh`

`define.vh`는 명령어 디코딩 과정에서 반복해서 사용하는 비트 값을 의미 있는 이름으로 표현합니다. CPU 제어부와 데이터패스가 숫자 상수를 직접 비교하지 않고 `R_TYPE`, `ADD`, `BEQ` 등의 이름을 사용하게 하므로 코드의 가독성과 유지보수성이 높아집니다.

예를 들어 다음 코드는 명령어의 Opcode가 R-Type인지 확인합니다.

```systemverilog
if (opcode == `R_TYPE) begin
    // R-Type instruction control
end
```

## Opcode 정의

| 매크로 | Opcode | 명령어 형식 또는 기능 |
|---|---|---|
| `R_TYPE` | `7'b011_0011` | 레지스터 간 산술·논리 연산 |
| `S_TYPE` | `7'b010_0011` | Store 명령 |
| `I_TYPE` | `7'b001_0011` | Immediate 산술·논리 연산 |
| `IL_TYPE` | `7'b000_0011` | Load 명령 |
| `B_TYPE` | `7'b110_0011` | 조건 분기 명령 |
| `LUI_TYPE` | `7'b011_0111` | Load Upper Immediate |
| `AUIPC_TYPE` | `7'b001_0111` | PC 기준 Upper Immediate 연산 |
| `JAL_TYPE` | `7'b110_1111` | Jump and Link |
| `JALR_TYPE` | `7'b110_0111` | Jump and Link Register |

## ALU 연산 정의

ALU 제어 값은 RISC-V 명령어의 `funct3`와 `funct7` 조합을 구분하기 위해 사용합니다.

| 매크로 | 제어 값 | 기능 |
|---|---|---|
| `ADD` | `4'b0_000` | 덧셈 |
| `SUB` | `4'b1_000` | 뺄셈 |
| `SLL` | `4'b0_001` | 논리 왼쪽 시프트 |
| `SLT` | `4'b0_010` | Signed 비교 |
| `SLTU` | `4'b0_011` | Unsigned 비교 |
| `XOR` | `4'b0_100` | XOR |
| `SRL` | `4'b0_101` | 논리 오른쪽 시프트 |
| `SRA` | `4'b1_101` | 산술 오른쪽 시프트 |
| `OR` | `4'b0_110` | OR |
| `AND` | `4'b0_111` | AND |

## Branch 조건 정의

| 매크로 | 제어 값 | 기능 |
|---|---|---|
| `BEQ` | `4'b0_000` | 같으면 분기 |
| `BNE` | `4'b0_001` | 다르면 분기 |
| `BLT` | `4'b0_100` | Signed 값이 작으면 분기 |
| `BGE` | `4'b0_101` | Signed 값이 크거나 같으면 분기 |
| `BLTU` | `4'b0_110` | Unsigned 값이 작으면 분기 |
| `BGEU` | `4'b0_111` | Unsigned 값이 크거나 같으면 분기 |

ALU 연산과 Branch 조건은 서로 다른 디코딩 문맥에서 사용되므로 일부 제어 값이 같을 수 있습니다.

## CPU 모듈과의 연결

`define.vh`는 CPU 및 데이터패스 소스에서 다음과 같이 포함하여 사용합니다.

```systemverilog
`include "define.vh"
```

현재 `define.vh`가 `CPU_APB/Define code` 폴더에 있으므로 컴파일 시 이 폴더를 Verilog include 경로에 추가해야 합니다. include 경로를 사용하지 않는 경우에는 소스 코드에서 상대 경로를 지정할 수 있습니다.

```systemverilog
`include "Define code/define.vh"
```

시뮬레이터에 include 디렉터리를 지정하는 방식을 권장합니다. 예를 들어 사용하는 도구에 따라 `CPU_APB/Define code`를 include directory로 등록하면 기존 `` `include "define.vh" `` 구문을 유지할 수 있습니다.

이 폴더의 정의 코드는 Assembly Code 폴더에 있는 프로그램을 CPU가 올바르게 해석하고 실행하기 위한 공통 기준 역할을 합니다.

