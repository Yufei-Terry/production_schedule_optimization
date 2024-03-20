# production_schedule_optimization
A Project for Optimizing Production Line Schedule

##### Overview
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
result:
![image](https://github.com/Yufei-Terry/production_schedule_optimization/assets/146860931/ff5b9c70-593a-4e52-abb3-f8aed666a877)

