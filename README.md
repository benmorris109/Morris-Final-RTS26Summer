# Morris-Final-RTS26Summer
Below is the full README taken from Application 2

## Benjamin Morris - Application 2 README defense

### Task table (mandatory)

| Task  | Function                   | Period (ms) | WCET measured (µs) | WCET + 30% margin (µs)| Deadline | Priority | Core |
|-------|----------------------------|------------:|-------------------:|----------------------:|---------:|---------:|-----:|
| A     | Dispatch Interlock Check   | 10          | 584                | 759.2                 | 10 ms    | 15       | 1    |
| B     | Motor Speed Control        | 20          | 3913               | 5086.9                | 20 ms    | 10       | 1    |
| C     | Operator-Display Refresh   | 50          | 9085               | 11810.5               | 50 ms    | 5        | 1    |
| D     | Maintenance Logging        | 100         | 19262              | 25040.6               | 100 ms   | 2        | 1    |

https://i.gyazo.com/6dc8b339ac39f91213de3818c0afa06b.png <--- proof of serial monitor output of WCET values

https://i.gyazo.com/7032768cc0f2089440492e60c1747142.png <--- serial monitor output with mean calculations included
As we can see in this screenshot, the mean WCET values of the 4 tasks are very similar to the actual max WCET. This is another reason why the
30% margin is added when making the utilization calculations. When running the code with the mean calculations, it can be seen that the max
WCET values for task C and D slightly change but this change is extremely small and does not affect the utilization calculations. Since the mean
values are close to the max WCET, it shows that there is a low variability in execution time and the task behavior is predictable.  

https://i.gyazo.com/63e7ec3469a6b3cbe5a275723eeaafed.png <--- web page showing task table with heartbeats
This screenshot is to show that I was able to get the web server running with the four tasks but it provides completely different WCET values
compared to the serial monitor output. I'm not exactly sure why this is the case but I figure because there are extra delays, especially in 
task D, when the web server has to run due to it connecting the the Wi-Fi and printing the values there. Because of these unknown changes
I chose to use the serial monitor output values for my task table/WCET values. 

### Schedulability defense

- Total utilization U = ∑ Cᵢ/Tᵢ (convert WCET values from us to ms by dividing by 1000)
    Using base WCET values: U = (584/1000/10) + (3913/1000/20) + (9085/1000/50) + (19262/1000/100) = 0.6284
    Using +30% WCET values: U = (759.2/1000/10) + (5086.9/1000/20) + (11810.5/1000/50) + (25040.6/1000/100) = 0.8169
  As we can see, the U values show that under both conditions, the tasks are feasible to be scheduled under EDF since U < 1 for both.
  However, this does not guarantee that they are schedulable under RMS. To check its feasibility we must compare it to the Liu-Layland bound.

- Liu-Layland bound for n=4: U ≤ 4(2^(1/4) − 1) = 0.7568
    Using the base WCET values, we can see that U = 0.6284 and RMS(n) = 0.7568. Because 0.6284 < 0.7568, we can conclude that the tasks
    are feasible to be scheduled under RMS. However, if we look at the +30% WCET values, this changes because U = 0.8169 and RMS(n) is
    the same. Because 0.8169 > 0.7568, we can conclude that with an added 30% margin, the tasks are not feasible to be scheduled under RMS.

- If U > Liu-Layland: run response-time analysis on task D (lowest priority)
    Base values: D = T = 100 ms
    Rd(0) = Cd = 19262/1000 = 19.262 ms
    Rd(1) = 19.262 + [19.262/10]*0.584 + [19.262/20]*3.913 + [19.262/50]*9.085 = 19.262 + 2*0.584 + 1*3.913 + 1*9.085 = 33.428 ms
    Rd(2) = 19.262 + [33.428/10]*0.584 + [33.428/20]*3.913 + [33.428/50]*9.085 = 19.262 + 4*0.584 + 2*3.913 + 1*9.085 = 38.509 ms
    Rd(3) = 19.262 + [38.509/10]*0.584 + [38.509/20]*3.913 + [38.509/50]*9.085 = 19.262 + 4*0.584 + 2*3.913 + 1*9.085 = 38.509 ms
  Task D (Maintenance Logging) converges at 38.509 ms which is way less than the deadline of 100 ms. Therefore, task D is schedulable under
  RMS when the base WCET values are used. 
    +30% margin values: D = T = 100 ms
    Rd(0) = Cd = 25040.6/1000 = 25.0406 ms
    Rd(1) = 25.0406 + [25.0406/10]*0.7592 + [25.0406/20]*5.0869 + [25.0406/50]*11.8105 = 25.0406 + 3*0.7592 + 2*5.0869 + 1*11.8105 = 49.3025 ms
    Rd(2) = 25.0406 + [49.3025/10]*0.7592 + [49.3025/20]*5.0869 + [49.3025/50]*11.8105 = 25.0406 + 5*0.7592 + 3*5.0869 + 1*11.8105 = 56.0086 ms
    Rd(3) = 25.0406 + [56.0086/10]*0.7592 + [56.0086/20]*5.0869 + [56.0086/50]*11.8105 = 25.0406 + 6*0.7592 + 3*5.0869 + 2*11.8105 = 68.2783 ms
    Rd(4) = 25.0406 + [68.2783/10]*0.7592 + [68.2783/20]*5.0869 + [68.2783/50]*11.8105 = 25.0406 + 7*0.7592 + 4*5.0869 + 2*11.8105 = 74.1244 ms
    Rd(5) = 25.0406 + [74.1244/10]*0.7592 + [74.1244/20]*5.0869 + [74.1244/50]*11.8105 = 25.0406 + 8*0.7592 + 4*5.0869 + 2*11.8105 = 74.8836 ms
    Rd(6) = 25.0406 + [74.8836/10]*0.7592 + [74.8836/20]*5.0869 + [74.8836/50]*11.8105 = 25.0406 + 8*0.7592 + 4*5.0869 + 2*11.8105 = 74.8836 ms
  Task D (Maintenance Logging) converges at 74.8836 ms when the WCET values with the added margin are used. This is still less than the
  deadline of 100 ms, so task D is schedulable under RMS when the margin WCET values are used. 

- Conclusion: feasible / infeasible / borderline. State which.
    In conclusion, it is safe to say that it is feasible for all of these tasks to be scheduled based on their WCET values and periods. Using 
    multiple tests with both the base WCET values and WCET values with an added 30% margin prove this as they were both feasible under the 
    tests. Even though the total utilization for the added margin values exceeds the RMS value of 0.7568, we can still conclude that the 
    tasks are feasible with the added margin as the RTA test on task D proves that the task converges below the deadline. Since D is the lowest
    priority task, we can conclude that all of the higher priority tasks also converge before the deadline when analyzing with RTA. Adding in the 
    margin to the WCET values helps prove that the task system can still be scheduled even in a case that exceeds the worst-case execution time.
    Thus, we can assume that the task will be schedulable as the base values are feasible to be scheduled with every test. Only the margin values
    fail the Liu-Layland test, but this does not guarantee that tasks are not schedulable, it can only guarantee that tasks are schedulable
    which is why we used RTA to check as well. The best way to describe this system would be that it operates with reduced timing margin but it 
    is not infeasible. For the most part, it is feasible to schedule all of the tasks even with the margin but becuase one test is failed, it 
    could cause some questions about the overrall feasibility. These WCET values are accurate becuase the instertion sort used in task D forces a
    worst case response. The array is reverse-sorted every period so it produces the true WCET as the measured max. If this reset is not done, it
    creates a best case scenario and the WCET will be misleading. The WCET values were then collected over many task activations, making them honest.

### Preemption evidence

Add this to one of your task bodies:
```c
int64_t t = esp_timer_get_time();
ESP_LOGI(TAG, "task_a tick t=%lld", t);
```
Along with these lines of code in Task A and Task B, I also added an extra line after the WCET measurement that 
indicated the tick that the task ended at using these commands:
int64_t t_end = esp_timer_get_time();
ESP_LOGI(TAG, "task_a end t=%lld", t_end);

I did this because it showed the beginning tick of the task as well as the ending time of tasks A and B in the serial
monitor so that evidence of preemption could be seen. After running the code with these lines in both tasks A and B, I
received confirmation of preemption working in the serial monitor through the following output log lines:
I (4310) app2: task_b start t=4340401
I (4320) app2: task_a start t=4346838
I (4320) app2: task_a end t=4349661
I (4330) app2: task_a start t=4353247
I (4330) app2: task_a end t=4358167
I (4340) app2: task_b end t=4363350
https://i.gyazo.com/40b3822786590d71ce514f44d7f6f70d.png <-- link to screenshot showing serial monitor output

From these log lines, we can see that preemption is working because while task B is active, task A is starting up and running
multiple times before task B ends. This shows preemption because it shows that a higher priority task will run even if it wakes
during a lower priority task's iteration. After running this code with these extra lines, I commented out the lines as they interrupted
the functionality of the rest of the tasks, so they are simply being used for preemption evidence and are not used for the functionality
of the tasks all working together. 

### Engineering analysis

1. **Priority defense** — explain each priority. RMS says shortest period &rarr; highest priority. Did you follow it?
  The task priorities in this system were assigned based on the RMS policy, where tasks with shorter periods receive a higher priority because of
  the more frequent deadline. The dispatch interlock task has the highest priority at 15 with a period of 10 ms becuase it is the most time-critical
  function that is necessary for upholding the safety of the ride. The motor control task has a period of 15 ms and has a priority of 10. This task
  is not necessarily vital to the safety of the ride but it is critical to its functionality and helping it run, thus it has the 2nd highest priority.
  The operator display refresh task has a period of 50 ms and a priority of 5. Display updates are less time sensitive then the other functions but
  they are still critical in ensuring the ride runs properly as the operators must be kept updated on the ride's status. The maintenance logging task
  has the lowest priority at 2 with a period of 100 ms. This is because logging is the least time-critical function and does not need to be completed
  as often as the other tasks, which are important to the safety and functionality of the ride. Logging/housekeeping should always be kept as the
  lowest priority task, a rule that was followed in this system. The priority assignments of these 4 tasks follows the general RMS rule as well as
  the popular rules when it comes to ordering functions based on their importance. 

2. **3× WCET stress** — if your highest-priority task's WCET tripled, what's the new U? Is the set still feasible?
  If the highest priority task (Task A - Dispatch Interlock Check) had its WCET tripled, then it would be 1752 us. Adding a 30% margin to this value
  would then give a WCET of 2277.6 us. If we calculate U with the base WCET and this tripled task A value, then U = 0.7452. If we run this same 
  calculation using the +30% margin WCET values, then U = 0.9687. Both of these values are less than 1, meaning the tasks are still schedulable under
  EDF and with the base values 0.7452 < 0.7568 so it would still be schedulable under RMS. However, when we use the added margin values, 0.9687 > 0.7568,
  so we cannot guarantee that the set is still feasible. However, this does not prove that it is infeasible, and RTA would have to be performed to see 
  if the tasks converge before their respective deadlines. 

3. **Preemption proof** — quote the two timestamps showing preemption.
  Preemption evidence can be found in the section above this (##Preemption evidence)

### Concurrency Diagram

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

Since Core 0 runs either the web server or the serial monitor based on the chosen output, I chose to include them in the same block.
