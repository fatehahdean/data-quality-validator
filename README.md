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

![Trigger: raw and processed]
<img width="1441" height="694" alt="9-2-project-trigger" src="https://github.com/user-attachments/assets/8479b755-1126-48f4-9a2f-d23632d53c1b" />
<img width="1447" height="698" alt="9-1-project-trigger" src="https://github.com/user-attachments/assets/5b89f657-3523-46a8-885c-3cc671372674" />

![Validator report: FAIL]
<img width="292" height="160" alt="10-project3-reportt" src="https://github.com/user-attachments/assets/9f5c7f8f-f98d-45f7-8db4-a85a14d09435" />

![Validator test: FAIL]
<img width="1158" height="691" alt="11-project3-fail-testtt" src="https://github.com/user-attachments/assets/1b71c5d0-1c77-4ee4-b8d8-e38b7fc81367" />

![Gate blocked corrupted file]
<img width="1133" height="494" alt="12-project3-gate-blocked" src="https://github.com/user-attachments/assets/281e6b4b-6890-41d7-9465-f9d07daacbd5" />

![Gate passed clean file]
<img width="1146" height="545" alt="13-project-gate-pass" src="https://github.com/user-attachments/assets/7a081ca9-d4ce-45c5-909d-ec72ecf5c972" />

**Design choice**

The gate uses a synchronous Lambda call instead of reading the validator's report file. EventBridge invokes both functions at the same time, so cleaning would not be able to wait for a report written after it started. Calling the validator directly makes the decision happen before any output is written.

**Limitations**
- Validation runs on the whole file in one pass, so a 133 MB file takes about 40 seconds. Larger files would need chunked validation or Step Functions.
- The validator checks a fixed schema and voltage range, set for this household dataset. A real meter feed would need configurable rules.
- Failed files stay in `raw/`. There is no quarantine folder or alert yet.


