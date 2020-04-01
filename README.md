# 2020 ROADEF - RTE : Solution checker user guide

## General notice
---
The solution cheker is developped in Python3.

It was not designed to be efficient, but rather to be easily understandable.

To ease its usage, the whole source code is embedded in a single file, which shall be lauched as a script.

Challenge_Rules.pdf is the document describing the rules of the challenge and Challenge_Subject.pdf the document defining the problem to be solved. For more detailed and registration please refer to https://www.roadef.org/challenge/2020/en.


## Usage
---
The `RTE_ChallengeROADEF2020_checker` script accepts 2 arguments:
- *instance_file* : path to the instance file, in `.json` format
- *solution_file* : path to corresponding solution file, in `.txt` format

**Remark:** some of the instance files are very large (several gigas), reading them might take a few minutes.

### Example
```
python RTE_ChallengeROADEF2020_checker.py exemple1.json output1.txt
```

## Produced output
---
The script prompts all detected inconsistencies on standard output, and computes a few indicators to evaluate the solution overall quality.

The solution is considered valid if no error has been detected.


### Successfull checking output example
```
Reading json file example1.json...
Done
Loading solution from output1.txt...
Done
Checking constraints...
        Checking schedule constraints...
        Done
        Checking resources constraints...
        Done
        Checking exclusions constraints...
        Done
Done
Computing objectives values...
        Computing risk...
        Done
        Computing mean risk...
        Done
        Computing Q95...
        Done
Done
Instance infos:
        Interventions number:  3
        Scenario numbers:  3
Solution evaluation:
        mean_risk over time:  [4.66666667 4.         3.        ]
        Objective 1 (mean risk):  3.8888888888888893
        Q95 over time:  [5. 8. 4.]
        Objective 2 (expected excess):  1.7777777777777777
```

### Failed checking output example
```
Reading json file example1.json...
Done
Loading solution from output1.txt...
ERROR: Unexpected Intervention I4 in solution file../resources/output/output1.txt.
Done
Checking constraints...
        Checking schedule constraints...
ERROR: Schedule constraint 4.1.2: Intervention I1 has not been scheduled.
        Done
        Checking resources constraints...
ERROR: Resources constraint 4.2 lower bound: Workload on Resource c1 at time 0 does not match lower bound. Value 0.0 is lower than bound 10 minus tolerance 1e-05.
ERROR: Resources constraint 4.2 lower bound: Workload on Resource c1 at time 2 does not match lower bound. Value 0.0 is lower than bound 6 minus tolerance 1e-05.
        Done
        Checking exclusions constraints...
ERROR: Exclusions constraint 4.3: Interventions I2 and I3 are both ongoing at time 2.
        Done
Done
Computing objectives values...
        Computing risk...
        Done
        Computing mean risk...
        Done
        Computing Q95...
        Done
Done
Instance infos:
        Interventions number:  3
        Scenario numbers:  3
Solution evaluation:
        mean_risk over time:  [0. 4. 0.]
        Objective 1 (mean risk):  1.3333333333333333
        Q95 over time:  [0. 8. 0.]
        Objective 2 (expected excess):  1.3333333333333333
```

The first detected error does not interrupt the checking process. However, since the evaluated solution is not valid, the computed indicators are no more relevant.


## Carried checks
---
Following checks are being carried by checker.  
If none of the following erros has been uncovered, the solution is considered valid.

Constraints mentioned by a paragraph number in error messages refer to the ones described in the challenge's presentation document.


### Undefined intervention
Solution file contains a reference to an intervention that does not exist in the instance.
```
ERROR: Unexpected Intervention I100 in solution file output1.txt.
```
Intervention is then ignored.


### Starting date format unmet
Intervention's starting date is not an integer.
```
ERROR: Unexpected starting time 1.0 for Intervention I1. Expect integer value.
```
Intervention is considered unscheduled, and is not taken into account in further checks and indicators computation.

### Duplicate entry
Solution file defines several starting date for a same intervention (i.e. intervention appears on several lines).
```
ERROR: Duplicate entry for Intervention I1. Only first read value is being considered.
```
Only the first date is taken into account.

### Unscheduled intervention
An intervention has not been scheduled: it is not present in the solution file, or its starting date is ineligible and was previously ignored.
```
ERROR: Schedule constraint 4.1.2: Intervention I1 has not been scheduled.
```

### Starting date out of planification range
An intervention's starting date is out of the range of the planification.
```
ERROR: Schedule constraint 4.1 time validity: Intervention I1 starting time 4 is not a valid starting date. Expected value between 1 and 3.
```
Intervention is considered unscheduled, and is not taken into account in further checks and indicators computation.

### Starting date is beyond interevntion's limit
An intervention's starting date is posterior to the intervention's limit date, beyond which it cannot be scheduled in order to be finished in time (before the latest considered planification's period).
```
ERROR: Schedule constraint 4.1.3: Intervention I1 realization exceeds time limit. It starts at 2 while time limit is 1.
```
Intervention is considered unscheduled, and is not taken into account in further checks and indicators computation.

### Workload exceeding resource upper bound
Total workload over a period (arising from the interventions schedule) is greater than a resource availability.
```
ERROR: Resources constraint 4.2 upper bound: Workload on Resource c1 at time 0 exceeds upper bound. Value 50.0 is greater than bound 49 plus tolerance 1e-05.
```
This check is carried out with a 1e-5 tolerance.

### Workload not meeing resource lower bound
Total workload over a period (arising from the interventions schedule) does not met a resource necessary minimum usage.
```
ERROR: Resources constraint 4.2 lower bound: Workload on Resource c1 at time 0 does not match lower bound. Value 0.0 is lower than bound 10 minus tolerance 1e-05.
```
This check is carried out with a 1e-5 tolerance.

### Unmet exclusion
Two interventions that can not be done concurrently are yet performed on overlapping periods.
```
ERROR: Exclusions constraint 4.3: Interventions I2 and I3 are both ongoing at time 1.
```



## Remarks
---
**Remark 1:** no test ensures that the given solution does match the provided instance (no shared name does exist to ensure so).  
It is up to the user to ensure a certain level of consistency between the supplied files.

**Remark 2:** paths may be given as relative paths.  
The reference for relative path is the current repository.

**Remark 3:** the checker does not stop on the first found error. All possible erros are looked up for to try and be as comprehensible as possible in a single run.  
Nontheless, if inconsistencies are detected on first constraint, the subsequent checks might be skewed if some information were to be ommited.
For example, an intervention could have been considered unscheduled, and thus not been taken into account to compute workload.

## Instances format

The instance's data format is described in the challenge's rule document (§3.1.1).

Two small functional examples are provided: `exemple1.json` and `exemple2.json`.

### Commented example

The items of an insance are elucidated below.  
One can also look at the swagger model description to benefit from an alternative presentation of the underlying data model.

#### Resources
Set of available resources.  
A particular resource is identified by its name.  
The upper ("max") and lower ("min") bounds on resource availability are lists of float of length T (total duration of the instance).  
```
"Resources": {
  "c1" : {
    "max": [49.0, 23.0, 15.0],
    "min": [10.0, 0.0, 6.0]
}
```

#### Seasons
Studied periods are split into seasons.  
Seasons are used to define exclusions.  
Possible seasons are:
- "full": has to contain all periods
- "winter"
- "summer"
- "is": inter-season

Each period is present in one and only one season.
```
"Seasons": {
  "winter": [1, 2],
  "summer": [3],
  "is": []
}
```

#### Interventions
Set of all interventions to schedule.  
A particular intervention is identified by its name.

"tmax" is the date/period before which the intervention has to be scheduled (if it is scheduled after that date, it will still be ongoing after the planification limit date, which is forbidden).

"Delta" is the duration of the intervention depending on its starting date.  
It is a list of size T (total duration of instance), though values of index beyond tmax are not used.

Workload gives the the consumption of resources by the intervention (see below for details).

Risk gives the risk values associated with the intervention realization (see below for details).


```
"Interventions": {
  "I1": {
    "tmax": 1,
      "Delta": [3, 3, 2],
      "workload": {...},
      "risk": {...}
  }
```

##### Workload
Worlad describes the usage of each needed resource.  
Used resources are identified by their names ("c1" here).  
Several resources might be needed.  
Resources hereby mentionned have to be defined in the Resources set.

> "t" : { "st_1": w_1, "st_2": w_2, "st_3": w_3}

For a given resource, the workload is given for each timestep "t".  
For each timestep, the workload "w" is given depending on the starting time "st" of the intervention.
```
workload": {
  "c1": {
    "1": { "1": 14, "2":  0, "3":  0},
    "2": { "1":  0, "2": 14, "3":  0},
    "3": { "1":  0, "2":  0, "3": 14}
  }
}
```

**Remark:** Contrary to what is shown in the above example, null values are not written in generated instances. Some element might hence be "missing" in the table.


##### Risk
> "t": {"st_1":[...], "st_2":[...]}

Risk values for each scnerio are given for each timestep "t", and depending on the intervention starting time "st".  
For a given "t" and "st", risk is a list whose size is equal to the number of scenario (see below) for the current timestep. It gives a risk value for each of the scenario.
```
"risk": {
  "1": {"1": [4, 8, 2], "2": [0, 0, 0]},
  "2": {"1": [0, 0], "2": [3, 8]},
  "3": {"1": [0, 0, 0], "2": [0, 0, 0]}
sss}
```

#### Exclusions
Set of interventions mutual exclusion over a season.  
The interventions names and the season name are given in a list.  
The interventions shall not be concurrently ongoing on any of the season's period.
```
"Exclusions": {
  "E1": ["I2", "I3", "full"]
}
```

#### Instance horizon
The index of the last period belonging to the studied horizon.  
As each study starts at time 1, it is also the length of the studied horizon.

```
"T": 3
```

#### Scenario number
Number of available and considered scenario for each timestep.  
It is a list of size T.
```
"Scenarios_number": [3, 2, 3]
```

## Solutions format

The solution's data format is also described in the rule document (§3.1.2).

Solutions matching the prodvided examples are also available: `output1.txt` and `output2.txt` respectively.

**Remark:** beside all the hereby described cheks, the solution format is not being tested.  
Format issues may arise to script failure.
