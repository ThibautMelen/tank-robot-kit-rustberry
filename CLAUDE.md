# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Freenove Tank Robot Kit for Raspberry Pi - a robotics platform with client-server architecture for controlling a tank-style robot.

## Architecture

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart LR
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef process fill:#6366f1,stroke:#4f46e5,stroke-width:2px,color:#ffffff
    classDef data fill:#06b6d4,stroke:#0891b2,stroke-width:2px,color:#ffffff
    classDef external fill:#64748b,stroke:#475569,stroke-width:2px,color:#ffffff

    subgraph Client["Desktop Client"]
        UI[Main.py<br/>Entry Point]:::success
        ClientUI[Client_Ui.py<br/>PyQt5 UI]:::process
    end

    subgraph Network["TCP Communication"]
        CMD[Port 5003<br/>Commands]:::data
        VIDEO[Port 8003<br/>Video Stream]:::data
    end

    subgraph Server["Raspberry Pi Server"]
        MAIN[main.py<br/>Entry Point]:::success
        CAR[car.py<br/>Core Logic]:::process
        MSG[message.py<br/>Protocol Parser]:::process

        subgraph Hardware["Hardware Modules"]
            MOTOR[motor.py<br/>gpiozero]:::external
            SERVO[servo.py<br/>pigpio/gpiozero]:::external
            LED[led.py<br/>WS281X/SPI]:::external
            CAM[camera.py<br/>picamera2]:::external
            SONIC[ultrasonic.py]:::external
            IR[infrared.py]:::external
        end
    end

    UI --> ClientUI
    ClientUI -->|Send Commands| CMD
    CMD --> MAIN
    MAIN --> CAR
    CAR --> MSG
    CAR --> MOTOR
    CAR --> SERVO
    CAR --> LED
    CAR --> SONIC
    CAR --> IR
    CAM -->|Stream Video| VIDEO
    VIDEO -->|Receive Stream| ClientUI
```

**Server** (`Code/Server/`): Runs on Raspberry Pi, controls hardware, streams video
- Entry point: `main.py`
- Core logic: `car.py` coordinates all hardware modules
- Protocol parser: `message.py` with `#` delimiter format

**Client** (`Code/Client/`): Desktop app for remote control
- Entry point: `Main.py`
- UI: `Client_Ui.py` (PyQt5, generated from `Client_Ui.ui`)

## Communication Protocol

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
sequenceDiagram
    participant C as Desktop Client
    participant N as TCP Port 5003
    participant S as RPi Server
    participant H as Hardware

    Note over C,S: Text-based messages with # separator

    C->>N: CMD_MOTOR#-2000#2000
    N->>S: Parse command
    S->>H: Control motors

    C->>N: CMD_SERVO#0#90
    N->>S: Parse command
    S->>H: Set servo angle

    C->>N: CMD_LED#1#255#0#0#100
    N->>S: Parse command
    S->>H: Update LEDs

    C->>N: CMD_SONIC
    N->>S: Parse command
    S->>H: Measure distance
    H-->>S: Distance value
    S-->>N: Response
    N-->>C: Distance data
```

**Protocol Format**: Text-based TCP messages using `#` separator
- `CMD_MOTOR#duty1#duty2` - Motor control (-4095 to 4095)
- `CMD_SERVO#channel#angle` - Servo control (0-180)
- `CMD_LED#mode#r#g#b#brightness` - LED control
- `CMD_SONIC` - Request ultrasonic measurement
- `CMD_ACTION#action_id` - Predefined actions

See `Communication Protocol.pdf` for full specification.

## Hardware Compatibility

| Component | PCB V1.0 | PCB V2.0 |
|-----------|----------|----------|
| Servo GPIOs | 7, 8 | 12, 13 |
| Infrared Pins | 16, 20, 21 | 16, 26, 21 |
| LED Driver | GPIO18 WS281X | GPIO10 SPI |
| Best RPi | RPi 4 | RPi 5 |

**Critical**: RPi 5 cannot use pigpio or WS281X driver with PCB V1.0

## Setup

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart TD
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef process fill:#6366f1,stroke:#4f46e5,stroke-width:2px,color:#ffffff
    classDef decision fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#ffffff
    classDef info fill:#8b5cf6,stroke:#7c3aed,stroke-width:2px,color:#ffffff

    START([Start Setup]):::success
    INSTALL[cd Code<br/>sudo python3 setup.py]:::process
    PROMPT{Select Camera Model<br/>OV5647 or IMX219?}:::decision
    CONFIG[Generate params.json<br/>RPi version, PCB version, Camera]:::process
    DEPS[Install Dependencies<br/>PyQt5, gpiozero, picamera2]:::process

    SERVER[Run Server on RPi<br/>cd Code/Server<br/>python3 main.py]:::process
    CLIENT[Run Client on Desktop<br/>cd Code/Client<br/>python3 Main.py]:::process

    READY([System Ready]):::success

    START --> INSTALL
    INSTALL --> PROMPT
    PROMPT -->|User Selection| CONFIG
    CONFIG --> DEPS
    DEPS --> SERVER
    SERVER --> CLIENT
    CLIENT --> READY

    Note1[Requires sudo access]:::info
    Note2[Auto-detects RPi & PCB version]:::info

    INSTALL -.-> Note1
    CONFIG -.-> Note2
```

**Setup Commands**:

```bash
# Raspberry Pi setup (requires sudo, prompts for camera model)
cd Code && sudo python3 setup.py

# Run server on Pi
cd Code/Server && python3 main.py

# Run client on desktop
cd Code/Client && python3 Main.py
```

## Configuration

`params.json` (auto-generated in Server/) stores:
- Raspberry Pi version (3/4/5)
- PCB version (1.0/2.0)
- Camera model (OV5647/IMX219)
- Camera port for RPi 5

## Key Dependencies

- **PyQt5**: UI framework (both client and server)
- **gpiozero**: GPIO abstraction for motors/servos
- **pigpio**: Advanced GPIO (RPi 4 only, not compatible with RPi 5)
- **picamera2**: Camera streaming
- **numpy/opencv2**: Computer vision

## License

Creative Commons Attribution-NonCommercial-ShareAlike 3.0 - **Commercial use NOT permitted**
