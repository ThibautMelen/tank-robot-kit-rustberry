# FNK0077: Freenove Tank Robot Kit for Raspberry Pi

## Supported Communication Method

| Method    | Description                                                      |
|-----------|------------------------------------------------------------------|
| TCPSocket | Port 5003 for command transmission, Port 8003 for video transmission |

## Command Format

Format: `A#10#20#30#40#50#\n`

- Each command starts with a command character (e.g., "A") to indicate the category.
- The `#` symbol acts as a separator between the command character and parameters.
- Each command ends with `\n` to mark its termination.
- During parsing:
  1. Split commands using `\n`.
  2. For each command, split the command character and parameters using `#`.
  3. Any remaining data after splitting should be carried over to the next parsing cycle.
  4. Commands consist of 1 command character and 0 to n parameters, depending on the specific command.

### Command Parsing Flow

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart TD
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef process fill:#6366f1,stroke:#4f46e5,stroke-width:2px,color:#ffffff
    classDef decision fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#ffffff
    classDef data fill:#06b6d4,stroke:#0891b2,stroke-width:2px,color:#ffffff

    A[Receive TCP Data]:::data
    B{Contains \n?}:::decision
    C[Split by \n]:::process
    D[Store Remaining Data]:::data
    E[For Each Command]:::process
    F[Split by #]:::process
    G[Extract Command Character]:::process
    H[Extract Parameters]:::process
    I[Execute Command]:::success
    J[Wait for More Data]:::process

    A --> B
    B -->|Yes| C
    B -->|No| D
    D --> J
    J --> A
    C --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> E

    accTitle: Command Parsing Flow
    accDescr: Flow diagram showing how commands are parsed by splitting on newline then hash delimiters
```

## FNK0077 Commands

```
CMD_MOTOR  = "CMD_MOTOR"
CMD_LED    = "CMD_LED"
CMD_SERVO  = "CMD_SERVO"
CMD_ACTION = "CMD_ACTION"
CMD_SONIC  = "CMD_SONIC"
CMD_MODE   = "CMD_MODE"
```

### TCP Communication Architecture

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
sequenceDiagram
    participant C as Client<br/>(Desktop)
    participant T1 as TCP Port 5003<br/>(Commands)
    participant S as Server<br/>(Raspberry Pi)
    participant H as Hardware<br/>(Motors/LEDs/Servos)
    participant T2 as TCP Port 8003<br/>(Video)

    Note over C,S: Command Flow
    C->>T1: CMD_MOTOR#2000#2000\n
    activate T1
    T1->>S: Forward Command
    activate S
    S->>S: Parse Command
    S->>H: Control Motors
    activate H
    H-->>S: Action Executed
    deactivate H
    deactivate S
    deactivate T1

    Note over C,S: Video Stream Flow
    activate S
    S->>T2: Video Frame Stream
    activate T2
    T2->>C: Forward Video
    deactivate T2
    deactivate S

    Note over C,S: Action with Response
    C->>T1: CMD_ACTION#1\n
    activate T1
    T1->>S: Forward Command
    activate S
    S->>H: Execute Action Sequence
    activate H
    H-->>S: Action Complete
    deactivate H
    S->>T1: CMD_ACTION#10\n
    T1->>C: Completion Signal
    deactivate S
    deactivate T1

    accTitle: TCP Communication Architecture
    accDescr: Sequence diagram showing bidirectional communication between client and server over two TCP ports
```

## FNK0077 Communication Protocol

### CMD_MOTOR

Controls the basic movement of the car, value range: -4095 ~ 4095

| App Command              | Action        |
|--------------------------|---------------|
| `CMD_MOTOR#2000#2000\n`  | Move forward  |
| `CMD_MOTOR#-2000#-2000\n`| Move backward |
| `CMD_MOTOR#-2000#2000\n` | Turn left     |
| `CMD_MOTOR#2000#-2000\n` | Turn right    |
| `CMD_MOTOR#0#0\n`        | Stop          |

### CMD_LED

Controls the LED lights.

Format: `CMD_LED#mode#R#G#B#data\n`

- **mode**: LED mode.
- **R, G, B**: Color values.
- **data**: the selection of 4 LEDs, value range: 0-15, 15 means selecting 4 LED lights.

| App Command                | Action                      |
|----------------------------|-----------------------------|
| `CMD_LED#mode#R#G#B#data\n`| Light up with specified color |

#### LED Modes

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart TD
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef process fill:#6366f1,stroke:#4f46e5,stroke-width:2px,color:#ffffff
    classDef info fill:#8b5cf6,stroke:#7c3aed,stroke-width:2px,color:#ffffff
    classDef data fill:#06b6d4,stroke:#0891b2,stroke-width:2px,color:#ffffff

    CMD[CMD_LED#mode#R#G#B#data\n]:::data

    subgraph Mode0[Mode 0: Off]
        M0[Turn Off All LEDs]:::success
    end

    subgraph Mode1[Mode 1: Manual]
        M1[Set RGB Color<br/>Use R, G, B values]:::process
    end

    subgraph Mode2[Mode 2: Chasing]
        M2[LED Chasing Effect<br/>RGB values ignored]:::info
    end

    subgraph Mode3[Mode 3: Blink]
        M3[Blinking Effect<br/>Use R, G, B values]:::process
    end

    subgraph Mode4[Mode 4: Breathing]
        M4[Breathing Effect<br/>Use R, G, B values]:::process
    end

    subgraph Mode5[Mode 5: Rainbow]
        M5[Rainbow Breathing<br/>RGB values ignored]:::info
    end

    CMD --> Mode0
    CMD --> Mode1
    CMD --> Mode2
    CMD --> Mode3
    CMD --> Mode4
    CMD --> Mode5

    accTitle: LED Modes Overview
    accDescr: Block diagram showing 6 LED modes with their behaviors and RGB usage
```

| App Command                | Action                                  |
|----------------------------|-----------------------------------------|
| `CMD_LED_MOD#0#R#G#B#15\n` | Turn off                                |
| `CMD_LED_MOD#1#R#G#B#15\n` | Manual RGB control                      |
| `CMD_LED_MOD#2#R#G#B#15\n` | Chasing mode (RGB value invalid)        |
| `CMD_LED_MOD#3#R#G#B#15\n` | Blink mode (input RGB)                  |
| `CMD_LED_MOD#4#R#G#B#15\n` | Breathing mode (input RGB)              |
| `CMD_LED_MOD#5#R#G#B#15\n` | Rainbow breathing mode (RGB value invalid) |

### CMD_SERVO

Controls servo angles.

Format: `CMD_SERVO#data#angle`

- **data**: Servo number (0 = gripper servo, 1 = lift servo).
- **angle**: Servo angle (range: 90-150).

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart TD
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef process fill:#6366f1,stroke:#4f46e5,stroke-width:2px,color:#ffffff
    classDef decision fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#ffffff
    classDef error fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#ffffff
    classDef data fill:#06b6d4,stroke:#0891b2,stroke-width:2px,color:#ffffff

    START[CMD_SERVO#data#angle\n]:::data
    CHECK{Servo Number}:::decision
    SERVO0[Gripper Servo]:::process
    SERVO1[Lift Servo]:::process
    VALIDATE{Angle Valid?<br/>90-150}:::decision
    SET[Set Servo Position]:::success
    ERR[Ignore Command]:::error

    START --> CHECK
    CHECK -->|data=0| SERVO0
    CHECK -->|data=1| SERVO1
    SERVO0 --> VALIDATE
    SERVO1 --> VALIDATE
    VALIDATE -->|Yes| SET
    VALIDATE -->|No| ERR

    accTitle: Servo Control Flow
    accDescr: Flowchart showing servo selection and angle validation process
```

| App Command           | Action                          |
|-----------------------|---------------------------------|
| `CMD_SERVO#0#angle\n` | Set gripper servo angle (90-150) |
| `CMD_SERVO#1#angle\n` | Set lift servo angle (90-150)    |

#### Host Control Commands

| Direction | Command              |
|-----------|----------------------|
| Up        | `CMD_SERVO#1#angle+5`|
| Down      | `CMD_SERVO#1#angle-5`|
| Left      | `CMD_SERVO#0#angle-5`|
| Right     | `CMD_SERVO#0#angle+5`|
| Home      | `CMD_SERVO#0#90` + `CMD_SERVO#1#140` |

### CMD_ACTION

Controls object gripping and releasing.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
sequenceDiagram
    participant C as Client
    participant S as Server
    participant M as Motors
    participant G as Gripper Servo

    Note over C,G: Action 1: Grip Object
    C->>S: CMD_ACTION#1\n
    activate S
    S->>M: Move Forward
    activate M
    M-->>S: Position Reached
    deactivate M
    S->>G: Close Gripper
    activate G
    G-->>S: Gripper Closed
    deactivate G
    S->>C: CMD_ACTION#10\n
    deactivate S
    Note over C,S: Completion Signal

    Note over C,G: Action 2: Release Object
    C->>S: CMD_ACTION#2\n
    activate S
    S->>G: Open Gripper
    activate G
    G-->>S: Gripper Opened
    deactivate G
    S->>C: CMD_ACTION#20\n
    deactivate S
    Note over C,S: Completion Signal

    accTitle: Action Command Sequences
    accDescr: Sequence diagram showing grip and release action flows with completion signals
```

| App Command        | Action                    |
|--------------------|---------------------------|
| `CMD_ACTION#1\n`   | Move forward and grip object |
| `CMD_ACTION#2\n`   | Release gripped object    |

**CMD_ACTION#1**: Move forward and grip the object. After the action is completed, send the following to the host computer (client): `CMD_ACTION#10` (Used to determine whether the action is completed.)

**CMD_ACTION#2**: Release the gripped object. After the action is completed, send the following to the host computer (client): `CMD_ACTION#20` (Used to determine whether the action is completed.)

### CMD_MODE

Sets the car's movement mode.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
stateDiagram-v2
    [*] --> Mode0: CMD_MODE#0

    Mode0: Mode 0: Free Control
    Mode1: Mode 1: Obstacle Avoidance
    Mode2: Mode 2: Line Following + Obstacle Clearing

    Mode0 --> Mode1: CMD_MODE#1
    Mode0 --> Mode2: CMD_MODE#2
    Mode1 --> Mode0: CMD_MODE#0
    Mode1 --> Mode2: CMD_MODE#2
    Mode2 --> Mode0: CMD_MODE#0
    Mode2 --> Mode1: CMD_MODE#1

    note right of Mode0
        Manual control via CMD_MOTOR
        No autonomous behavior
    end note

    note right of Mode1
        Automatic obstacle detection
        Ultrasonic data streaming
    end note

    note right of Mode2
        IR line following
        Obstacle detection & avoidance
        Ultrasonic data streaming
    end note

    accTitle: CMD_MODE State Transitions
    accDescr: State diagram showing three movement modes and transitions between them
```

| App Command      | Action                              |
|------------------|-------------------------------------|
| `CMD_MODE#0\n`   | Free control mode                   |
| `CMD_MODE#1\n`   | Obstacle avoidance mode             |
| `CMD_MODE#2\n`   | Line-following & obstacle-clearing mode |

### CMD_SONIC

Ultrasonic distance measurement.

| App Command        | Action                   |
|--------------------|--------------------------|
| `CMD_SONIC#data\n` | Enable ultrasonic ranging |

In `CMD_MODE#1` and `CMD_MODE#2` modes, the car sends ultrasonic distance data to the host (client).
