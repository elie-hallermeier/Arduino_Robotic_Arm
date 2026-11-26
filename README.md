# Arduino_Robotic_Arm

Firmware for a 4-servo robotic arm that picks up an object, carries it to a second spot and puts it down.

## The problem

Hobby servos jump to whatever angle you send them. Command a new pose directly and the arm snaps there, which shakes the base and can throw what it is holding. I wanted smooth, repeatable pick and place from an Arduino with no feedback sensors.

## What I built

An Arduino sketch driving four servos (base, shoulder, elbow, claw) through a PCA9685 PWM board over I2C. It runs one fixed sequence: reach down, close the claw, lift, rotate the base, lower, open the claw, return home. The angles in the current version are tuned for a 3D printed Benchy.

## Architecture

```mermaid
flowchart LR
    seq[Pick and place sequence<br/>in loop] --> interp[Joint interpolation<br/>getTo1, getTo3, getToDouble]
    interp --> pwm[Angle to PWM<br/>PCA9685_ServoEval]
    pwm --> pca[PCA9685, 50 Hz] --> servos[Servos on channels 0, 4, 8, 12]
```

| File | What it is |
|---|---|
| `robotic_arm.ino` | Current version |
| `Code_0.0.1` | First version, slower and with a shorter sequence |

## Design decisions

- Every move is interpolated in joint space. A move is split into `speed` equal steps with a delay between them, so the arm sweeps to the target at a set pace.
- Shoulder and elbow move together in `getToDouble`. Stepping both in the same loop moves the claw along one smooth path instead of two separate arcs.
- The claw closes and opens one degree at a time, slower when closing, so it grips the object without knocking it over.
- A PCA9685 instead of the Arduino's own PWM pins. Timing for all four servos comes from one chip over I2C.
- The current angle of each joint is kept in a global and updated as the arm moves. With no encoders, that is the only record of where the arm is.

## Setup

Hardware: an Arduino, a PCA9685 board on I2C, four servos on channels 0, 4, 8 and 12, and a separate 5V supply for the servos.

Install the [PCA9685-Arduino library by NachtRaveVL](https://github.com/NachtRaveVL/PCA9685-Arduino) from the Arduino Library Manager, open `robotic_arm.ino` and upload.

The angles in the sketch are specific to my arm and to where the object sits. Start from the home pose in the globals at the top (`ang1` to `ang4`) and adjust the steps in `loop()`.

## Limits

Open loop only. Positions are hardcoded angles, with no inverse kinematics and no sensing of whether the grab worked.
