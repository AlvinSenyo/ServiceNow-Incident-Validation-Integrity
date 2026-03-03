# ServiceNow-Incident-Validation-Integrity (2026 Release)
This project implements advanced data validation and user guidance logic on the ServiceNow **Incident** table. It ensures that high-impact and hardware-related tickets follow strict data entry rules to improve reporting and routing accuracy.

## 📝 Features
- **Dynamic Field Enforcement:** Automated mandatory status for `cmdb_ci` (Hardware) and `assignment_group` (High Impact).
- **Clean UI Management:** Automated field message cleanup using UI Policy "Execute if False" blocks.
- **Hard Database Validation:** Prevents form submission if the `subcategory` is empty via `onSubmit` interruption.
- **RBAC Security:** Custom roles and ACLs protect the Configuration Item field from unauthorized edits.

---

## 🏗️ Architecture Detail

### 1. UI Policies (Front-End)
Uses declarative conditions to manage the form state. 
- **Hardware Logic:** Triggers a yellow `#FFFACD` flash and an info message if the CI field is empty.
- **Impact Logic:** Ensures Assignment Group is filled for Priority 1 issues.
- **State Cleanup:** Uses `g_form.hideFieldMsg()` in the "Execute if False" scripts to ensure messages don't persist after the condition is no longer met.

## 📂 File Inventory
* [Technical_Manual](./Technical_Manual_Incident_ValidationAndIntegrity.txt) - Detailed, click-by-click build guide for manual replication.
* [Message1_2.PNG](./Message1_2.PNG) - Visual validation message if conditions are not met.
* [Message3.PNG](./Message3.PNG) - Visual validation message if conditions are not met.


### 2. Client Scripts (Validation Layer)
The `onSubmit` script acts as the final gatekeeper. By returning `false`, the script cancels the browser's attempt to send data to the server, satisfying the requirement that data "cannot be saved on DB" without a subcategory.

### 3. Security (ACLs)
Strict field-level security is applied:
- `incident.cmdb_ci`: Restricted to `u_incident_validator` and `admin`.
- `incident.*`: Baseline `itil` access.



---

## 🚀 Deployment Instructions
1. Import the `Incident_Validation_v2.xml` Update Set.
2. Elevate to `security_admin` to commit the ACLs.
3. Assign the `u_incident_validator` role to appropriate support personnel.

---

## 🛠️ Code Highlight: Clean Messaging
This snippet shows how we prevent message stacking and ensure UI cleanup:

```javascript
// Execute if True
function onCondition() {
    g_form.hideFieldMsg('cmdb_ci'); // Flush previous messages
    if (!g_form.getValue('cmdb_ci')) {
        g_form.showFieldMsg('cmdb_ci', 'Hardware requires CI selection.', 'info');
    }
}

// Execute if False
function onCondition() {
    g_form.hideFieldMsg('cmdb_ci'); // Remove message when category changes
}
