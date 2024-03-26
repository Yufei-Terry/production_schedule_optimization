# Job Shop Scheduling Optimization
A Project for Optimizing Production Line Schedule

## Overview
<img width="1071" alt="job-shop-scheduling-problem" src="https://github.com/Yufei-Terry/production_schedule_optimization/assets/146860931/44012583-a0ba-446d-a125-16a2c3cc17aa">

### 1. What is job-shop schedule problem


### 2. Input
##### 2.1 Production Sequence for each item
![image](https://github.com/Yufei-Terry/production_schedule_optimization/assets/146860931/6bcb8e3b-1f19-4f7e-9593-c6d9c330cc67)

##### 2.2 Processing time on each machine for each item
![image](https://github.com/Yufei-Terry/production_schedule_optimization/assets/146860931/c3394fe8-705d-4e5f-ae59-4faa160a33ac)

### 3. Data Processing
```python
import pandas as pd

file_path = 'jobshop.xlsx'

# loading all sheets
xls = pd.ExcelFile(file_path)

# read each sheet
sheet_names = xls.sheet_names
sheets_data = {sheet: xls.parse(sheet) for sheet in sheet_names}

sheets_preview = {sheet: data.head() for sheet, data in sheets_data.items()}
sheets_preview, sheet_names
```
result:
![image](https://github.com/Yufei-Terry/production_schedule_optimization/assets/146860931/8f7e0a43-c52e-46c6-893b-4e44a3704d7f)
<br><br>
```python
machines_sequence_df = sheets_data['Machines Sequence'].drop('Work Sequence', axis=1)
processing_time_df = sheets_data['Processing Time'].drop('Time', axis=1)

jobs_data = []

for index, row in machines_sequence_df.iterrows():
    job = []
    for col in machines_sequence_df.columns:
        machine_id = row[col]  # 从1开始的索引
        processing_time = processing_time_df.loc[index, col]
        if processing_time > 0:  # 忽略处理时间为0的工位
            job.append((machine_id, processing_time))
    jobs_data.append(job)

jobs_data
```

### 4. Solution
##### 4.1 


### 5. Model Programming
##### 5.1 Import required libraries
```python
pip install ortools
import plotly.figure_factory as ff
import collections
from ortools.sat.python import cp_model
import pandas as pd
import datetime
from collections import defaultdict
import math
```

##### 5.2 Initialization and Calculation of Problem Parameters
```python
machines_count = 1 + max(task[0] for job in jobs_data for task in job)
all_machines = range(machines_count)
jobs_count = len(jobs_data)
# Computes horizon dynamically as the sum of all durations.
horizon = sum(task[1] for job in jobs_data for task in job)
```

##### 5.3 Declare the model
```python
# Create the model.
model = cp_model.CpModel()
```

##### 5.4 Define the Variables
```python
# Named tuple to store information about created variables.
task_type = collections.namedtuple("task_type", "start end interval")
# Create the variables.
all_tasks = {}
machine_to_intervals = collections.defaultdict(list)
job_ends = []

for job_id, job in enumerate(jobs_data):
    for task_id, (machine, duration) in enumerate(job):
        suffix = f"_{job_id}_{task_id}"
        start_var = model.NewIntVar(0, horizon, "start" + suffix)
        end_var = model.NewIntVar(0, horizon, "end" + suffix)
        interval_var = model.NewIntervalVar(start_var, duration, end_var, "interval" + suffix)
        all_tasks[job_id, task_id] = task_type(start=start_var, end=end_var, interval=interval_var)
        machine_to_intervals[machine].append(interval_var)
        if task_id == len(job) - 1:
            job_ends.append(end_var)
```

##### 5.5 Define the constraints
```python
# Add the constraints.
# Processing sequence and no overlapping constraints.
for machine, intervals in machine_to_intervals.items():
    model.AddNoOverlap(intervals)

for job_id, job in enumerate(jobs_data):
    for task_id in range(1, len(job)):
        model.Add(all_tasks[job_id, task_id].start >= all_tasks[job_id, task_id - 1].end)
```

##### 5.6 Define the objective
```python
# Completion time definition.   objective function: minimize makespan time
cmax_var = model.NewIntVar(0, horizon, "cmax")
model.AddMaxEquality(cmax_var, job_ends)
model.Minimize(cmax_var)
```

##### 5.7 Invoke the solver: google_or_tools
```python
# Solve the model.
solver = cp_model.CpSolver()
status = solver.Solve(model)
```

##### 5.8 Display the detailed result
```python
if status in (cp_model.OPTIMAL, cp_model.FEASIBLE):
    print(f"Total completion time: {solver.Value(cmax_var)}")
    for job_id, job in enumerate(jobs_data):
        print(f"Job {job_id}:")
        for task_id, (machine, _) in enumerate(job):
            start = solver.Value(all_tasks[job_id, task_id].start)
            end = solver.Value(all_tasks[job_id, task_id].end)
            print(f"  Task {task_id} (Machine {machine}) - Start: {start} End: {end}")
else:
    print("No solution found.")

tasks = []
machine_jobs = defaultdict(list)

if status in (cp_model.OPTIMAL, cp_model.FEASIBLE):
    for job_id, job in enumerate(jobs_data):
        for task_id, (machine, _) in enumerate(job):
            start = solver.Value(all_tasks[job_id, task_id].start)
            end = solver.Value(all_tasks[job_id, task_id].end)
            machine_jobs[machine].append((start, end, job_id))
else:
    print("No solution found.")
```

##### 5.9 Display the result visually - by Gantt chart
```python
# Convert to DataFrame format for Plotly
df = []
for machine, jobs in machine_jobs.items():
    for start, end, job_id in jobs:
        df.append(dict(Task=f'Machine {machine}', Start=start, Finish=end, Resource=f'Job {job_id}'))

# Adjust time format for Gantt chart
for task in df:
    task['Start'] = datetime.datetime(2024, 1, 1) + datetime.timedelta(seconds=task['Start'])
    task['Finish'] = datetime.datetime(2024, 1, 1) + datetime.timedelta(seconds=task['Finish'])

# Create Gantt chart
fig = ff.create_gantt(df, index_col='Resource', show_colorbar=True, group_tasks=True, showgrid_x=True, title='Job Shop Schedule')
fig.show()
```
![image](https://github.com/Yufei-Terry/production_schedule_optimization/assets/146860931/a0188ae0-65c7-45f5-8201-e2693dc31af6)
