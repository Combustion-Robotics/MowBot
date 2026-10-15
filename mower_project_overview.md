# Autonomous Robot Mower

A DIY, fully autonomous lawn mower built on a hacked hoverboard. It uses a camera + IMU to see where it is, GPS to keep that position honest, and ROS 2 to plan and drive full coverage paths across the lawn.

## At a glance
| | |
|---|---|
| **Compute** | Raspberry Pi 5 (4GB), actively cooled |
| **Software** | ROS 2 Jazzy |
| **Drive base** | Hoverboard running hoverboard-firmware-hack-FOC (UART control) |
| **Localization** | Mono camera + IMU (visual-inertial SLAM), GPS, wheel odometry |
| **Navigation** | Nav2 + `opennav_coverage` (Fields2Cover v2) |
| **Power** | 36V hoverboard battery, buck converters for Pi and blade |

## How it works
```
Camera + IMU + GPS + wheel odom
        → VIO/SLAM (VINS-Fusion)
        → robot_localization (dual EKF)
        → Nav2 + coverage planner
        → safety watchdog
        → hoverboard driver → wheels
```

- **Seeing:** A single Pi Camera Module 3 paired with an IMU. Pure mono SLAM was ruled out — grass is hard to track and mono has no scale.
- **Staying on track:** GPS corrects long-term SLAM drift; hoverboard hall sensors give free wheel odometry.
- **Mowing:** The coverage planner turns a drawn boundary into back-and-forth mowing paths that Nav2 follows.
- **Fallback:** If mono-inertial SLAM isn't reliable outdoors, swap in a 2D LiDAR (RPLidar A1) or stereo/depth camera — chassis space is reserved.

## Safety
- Hardware E-stop cuts blade + drive power directly (not via software).
- Tilt/lift switch kills the blade if the mower is picked up or flips.
- Software watchdog stops the wheels if `/cmd_vel` goes silent for >0.5s.
- Blade gets its own motor, ESC and fused line, and is added **last** — after weeks of drive-only testing.

## Key software
| Role | Package |
|---|---|
| Hoverboard driver | `hoverboard-robotics/hoverboard-driver` |
| VIO/SLAM | `JanekDev/VINS-Fusion-ROS2-humble-arm` (upgrade path: ORB-SLAM3) |
| IMU / GPS | `ros2_bno055`, `nmea_navsat_driver` |
| Sensor fusion | `robot_localization` |
| Navigation | `navigation2` + `opennav_coverage` |

## Build order
1. Hoverboard driver — wheels move, odometry publishes
2. IMU + GPS drivers
3. Local EKF (wheels + IMU)
4. VIO indoors, hand-held
5. Global EKF with GPS
6. Nav2 point-to-point outdoors
7. Coverage planning inside a boundary
8. Safety watchdog, then (much later) the blade

## Status
Architecture and parts are decided; hoverboard firmware not flashed yet.
**Next up:** EKF YAML config, URDF / TF tree, wiring diagram.

## Inspiration
OpenMower and its ROS 2 port OpenMowerNext — RTK-GPS mowers, useful references for mechanics and safety.
