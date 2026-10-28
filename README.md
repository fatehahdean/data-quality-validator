## Project 3: Data Quality Gate

A validator that checks every raw file before it is cleaned. `clean-power-data` calls `validate-power-data` first and only processes files that pass.

**Flow**

raw/ upload → EventBridge → clean-power-data → validate-power-data → PASS: clean to processed/ / FAIL: blocked, nothing written

**Checks performed**
- Header matches the expected 9-column schema
- Missing value ratio (`?`) is below 5%
- Voltage stays within 200 to 260 V

**Test results**

| Input | Validation | Outcome |
|---|---|---|
| Corrupted sample (10 voltage values set to 999) | FAIL (out_of_range_voltage: 10) | Blocked, no output written |
| Full dataset (2,075,259 rows) | PASS | Cleaned: 2,049,280 rows written, 25,979 skipped |



**Design choice**

The gate uses a synchronous Lambda call instead of reading the validator's report file. EventBridge invokes both functions at the same time, so cleaning would not be able to wait for a report written after it started. Calling the validator directly makes the decision happen before any output is written.

**Limitations**
- Validation runs on the whole file in one pass, so a 133 MB file takes about 40 seconds. Larger files would need chunked validation or Step Functions.
- The validator checks a fixed schema and voltage range, set for this household dataset. A real meter feed would need configurable rules.
- Failed files stay in `raw/`. There is no quarantine folder or alert yet.


