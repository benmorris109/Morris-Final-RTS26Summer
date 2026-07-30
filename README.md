# Application 2: Industrial Ride Control System Analysis — Real-Time Systems Final Capstone

## Overview
This project implements a real-time industrial ride control system on Wokwi using FreeRTOS. The application demonstrates fixed-priority scheduling, worst-case execution time (WCET) measurement, schedulability analysis, and real-time system monitoring. Four periodic tasks execute on Core 1 while Core 0 provides either a serial monitoring interface or a Wi-Fi web dashboard for observing system performance.

## Demo
- Video: <YouTube / Wokwi link>
- Wokwi: https://wokwi.com/projects/466554293336470529

## Architecture

                  -------------------------------------------------------
                 | Core 1 - 4 tasks on same core                         |
                 | Preemptive scheduling on core 1                       |
                 | Task A: Dispatch Interlock (10 ms) Priority 15        |
                 | Task B: Motor Control (20 ms) Priority 10             |
                 | Task C: Operator-Display Refresh (50 ms) Priority 5   |
                 | Task D: Maintenance Logging (100 ms) Priority 2       |
                  -------------------------------------------------------
                    |             |                    |         
                    |writes       |writes              |writes
                    v             v                    v         
              hb_a/b/c/d   wcet_max_a/b/c/d   wcet_total_a/b/c/d
                    ^             ^                    ^ 
                    |reads        |reads               |reads
                    |             |                    |
                  -------------------------------------------------------
                 | Core 0 - Serial Monitor OR Wi-Fi + HTTP Server        |
                 | Prints task table with heartbeats/WCET stats          |
                 | USE_WEBSERVER = 0 for monitor                         |
                 | USE_WEBSERVER = 1 for HTTP server + WiFi              |
                 | task_monitor() for serial monitor                     |
                 | handle_root() for webserver                           |
                  -------------------------------------------------------
                  
The application is divided across the two processor cores. Core 1 executes the four periodic real-time tasks using fixed-priority preemptive scheduling, while Core 0 provides system monitoring through either the serial monitor or the embedded HTTP server. Runtime statistics, including heartbeat counters and WCET measurements, are shared between the two cores so that the monitoring interface can display the current state of the system without interfering with the real-time control tasks.

## Tasks & timing (WCET evidence)
| Task  | Function                   | Period (ms) | WCET measured (µs) | WCET + 30% margin (µs)| Deadline | Priority | Core |
|-------|----------------------------|------------:|-------------------:|----------------------:|---------:|---------:|-----:|
| A     | Dispatch Interlock Check   | 10          | 584                | 759.2                 | 10 ms    | 15       | 1    |
| B     | Motor Speed Control        | 20          | 3913               | 5086.9                | 20 ms    | 10       | 1    |
| C     | Operator-Display Refresh   | 50          | 9085               | 11810.5               | 50 ms    | 5        | 1    |
| D     | Maintenance Logging        | 100         | 19262              | 25040.6               | 100 ms   | 2        | 1    |

Using base WCET values: U = (584/1000/10) + (3913/1000/20) + (9085/1000/50) + (19262/1000/100) = 0.6284
Using +30% WCET values: U = (759.2/1000/10) + (5086.9/1000/20) + (11810.5/1000/50) + (25040.6/1000/100) = 0.8169

Schedulable under EDF (U<1) and under RMS based on measured WCET values. Response-time analysis confirms that all tasks meet their deadlines, including the lowest priority (Task D).

    Base values: D = T = 100 ms
    
    Rd(0) = Cd = 19262/1000 = 19.262 ms
    
    Rd(1) = 19.262 + [19.262/10]*0.584 + [19.262/20]*3.913 + [19.262/50]*9.085 = 19.262 + 2*0.584 + 1*3.913 + 1*9.085 = 33.428 ms
    
    Rd(2) = 19.262 + [33.428/10]*0.584 + [33.428/20]*3.913 + [33.428/50]*9.085 = 19.262 + 4*0.584 + 2*3.913 + 1*9.085 = 38.509 ms
    
    Rd(3) = 19.262 + [38.509/10]*0.584 + [38.509/20]*3.913 + [38.509/50]*9.085 = 19.262 + 4*0.584 + 2*3.913 + 1*9.085 = 38.509 ms

## Hazard analysis & standard mapping
| Hazard	                        | Mitigation                                                        |
----------------------------------|-------------------------------------------------------------------|
| Missed dispatch safety checks   |	Highest-priority task executes every 10 ms.                       | 
| Motor control delays	          | Dedicated 20 ms periodic control task.                            | 
| Display update latency	        | Display task assigned lower priority than control tasks.          |
| High processor utilization	    | Verified using measured WCET values and schedulability analysis.  |
| Logging interference	          | Maintenance logging executes as the lowest-priority task.         |

## Graceful degradation
To demonstrate graceful degradation, the system was evaluated under increased execution times by applying a 30% margin to each task's measured WCET. Although the resulting processor utilization exceeded the Liu-Layland RMS utilization bound, Response-Time Analysis confirmed that every periodic task continued to complete before its deadline. Rather than failing immediately when processor demand increased, the system maintained correct operation and predictable timing behavior, demonstrating flexibility under heavier loads. 

## Build & run
Target: ESP32

Framework: ESP-IDF

Operating System: FreeRTOS

Simulator: Wokwi

Open the project in Wokwi, build using ESP-IDF, and run the simulation. Runtime statistics can be viewed through either the serial monitor or the web dashboard depending on the selected configuration.

## Tailored for
This project is tailored towards the themed entertainment industry. More specifically, it is tailored towards ride control systems and controls/electrical engineering. The choices found in this project fit this role 
because the tasks are tailored towards actual tasks that ride control systems would have to undergo. The hazard analysis and schedulability analysis are all done in the context of this industry and the system diagram
also shows how the tasks all connect together under a single ride control system. 
