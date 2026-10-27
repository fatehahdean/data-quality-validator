
## Project 3: Data Quality Validator

An event-driven validator that runs whenever a new file lands in `raw/`. EventBridge triggers the pipeline, and the validator writes a PASS/FAIL report to `reports/`.

**Checks performed**
- Header matches the expected 9-column schema
- Missing value ratio (`?`) is below 5%
- Voltage stays within 200 to 260 V

**Test with a corrupted sample**
A 1,000-row sample was modified by setting 10 voltage values to `999`. The validator returned `status: FAIL` with `out_of_range_voltage: 10`.



