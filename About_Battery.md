# About Battery

> Essential information for powering your Freenove Tank Robot Kit safely and correctly.

## Critical Safety Warning

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart TD
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef error fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#ffffff
    classDef critical fill:#ec4899,stroke:#db2777,stroke-width:2px,color:#ffffff
    classDef decision fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#ffffff

    START[Battery Installation]:::decision
    CHECK{Batteries<br/>Fully Charged?}:::decision
    INSTALL[Install Batteries]:::success
    DAMAGE[Servo Damage<br/>Installation Errors]:::error
    CHARGE[Charge Batteries First]:::critical

    START --> CHECK
    CHECK -->|Yes| INSTALL
    CHECK -->|No| CHARGE
    CHARGE --> INSTALL
    CHECK -.->|If No, but<br/>install anyway| DAMAGE

    style DAMAGE stroke-width:4px,stroke-dasharray:5 5
```

> **CRITICAL: Assemble ONLY with fully charged batteries!**
> Installing batteries that are not fully charged will cause servo damage and installation errors.

> **WARNING: USB cable does NOT charge batteries**
> The control board requires a separate 18650 battery charger. Almost any charger suitable for 18650 batteries can be used.

## Do I Have the Right Battery?

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart TD
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef error fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#ffffff
    classDef decision fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#ffffff
    classDef info fill:#8b5cf6,stroke:#7c3aed,stroke-width:2px,color:#ffffff

    START[Check Your Battery]:::info
    SIZE{Battery Type:<br/>18650?}:::decision
    PROTECTION{Has Protection<br/>Circuit?}:::decision
    TOP_TYPE{Top Type?}:::decision

    CORRECT_FLAT[18650 Flat-Top<br/>Unprotected ✓]:::success
    CORRECT_BUTTON[18650 Button-Top<br/>Unprotected ✓]:::success
    WRONG_PROTECTED[Wrong: Protected<br/>18650 Battery]:::error
    WRONG_SIZE[Wrong: Not<br/>18650 Size]:::error

    START --> SIZE
    SIZE -->|Yes| PROTECTION
    SIZE -->|No| WRONG_SIZE

    PROTECTION -->|Unprotected| TOP_TYPE
    PROTECTION -->|Protected| WRONG_PROTECTED

    TOP_TYPE -->|Flat| CORRECT_FLAT
    TOP_TYPE -->|Button| CORRECT_BUTTON
```

As shown in the decision tree above, you need:

- **Size**: 18650 lithium-ion battery
- **Protection**: Unprotected (no circuit board at top)
- **Top Type**: Either Flat-Top or Button-Top

## Battery Types

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart LR
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef info fill:#8b5cf6,stroke:#7c3aed,stroke-width:2px,color:#ffffff

    subgraph Compatible ["Compatible Battery Types"]
        FLAT["Flat-Top<br/>────────<br/>Flat positive terminal<br/>No button protrusion"]:::success
        BUTTON["Button-Top<br/>────────<br/>Raised positive terminal<br/>Small button protrusion"]:::success
    end

    subgraph Features ["Both Must Be"]
        UNPROTECTED["Unprotected<br/>No protection circuit"]:::info
        SIZE["18650 Size<br/>18mm × 65mm"]:::info
    end

    Compatible --> Features
```

### Visual Difference

- **Flat-Top**: The positive terminal is flat and flush with the battery top
- **Button-Top**: The positive terminal has a small raised button (nipple)

Both types are compatible with this robot kit, as long as they are **unprotected** 18650 batteries.

## Power Flow

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'lineColor': '#64748b'}}}%%
flowchart LR
    classDef success fill:#10b981,stroke:#059669,stroke-width:2px,color:#ffffff
    classDef process fill:#6366f1,stroke:#4f46e5,stroke-width:2px,color:#ffffff
    classDef data fill:#06b6d4,stroke:#0891b2,stroke-width:2px,color:#ffffff

    BATT1["18650 Battery 1<br/>3.7V"]:::success
    BATT2["18650 Battery 2<br/>3.7V"]:::success
    HOLDER["Battery Holder<br/>Series Connection"]:::data
    BOARD["Control Board<br/>7.4V Input"]:::process

    subgraph Components ["Robot Components"]
        MOTORS["DC Motors"]:::process
        SERVOS["Servo Motors"]:::process
        LED["LED Lights"]:::process
        PI["Raspberry Pi"]:::process
    end

    BATT1 --> HOLDER
    BATT2 --> HOLDER
    HOLDER -->|7.4V| BOARD
    BOARD --> MOTORS
    BOARD --> SERVOS
    BOARD --> LED
    BOARD --> PI
```

The robot requires **two 18650 batteries** connected in series to provide 7.4V to the control board, which then powers all robot components.

## Purchase Links

Please find the battery description and purchase links on the following pages:

- [18650 Flat-Top Unprotected](https://github.com/Freenove/Freenove_Battery_List/blob/main/18650_Flat-Top_Unprotected.md)
- [18650 Button-Top Unprotected](https://github.com/Freenove/Freenove_Battery_List/blob/main/18650_Button-Top_Unprotected.md)

> **Note**: Some links in the list may be invalid. Please select other available links or purchase independently.

> **Need Help?** If you need assistance or want to report a link failure, please contact support@freenove.com

## Pre-Assembly Checklist

Before you start assembling your robot, verify:

- [ ] You have **two 18650 batteries** (Flat-Top or Button-Top, unprotected)
- [ ] You have a **battery charger** for 18650 batteries
- [ ] Both batteries are **fully charged**
- [ ] You understand that **USB cable does NOT charge the batteries**

Following this checklist prevents servo damage and ensures smooth assembly.
