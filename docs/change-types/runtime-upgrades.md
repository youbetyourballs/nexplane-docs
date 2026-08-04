# Runtime Upgrades

Runtime upgrade change types manage in-place version upgrades for language runtimes on managed hosts. Each type detects the current runtime, snapshots the host before making changes, installs the new version, updates system PATH and alternatives, and verifies the result.

---

## Java Runtime Upgrade

**Change type:** `java_runtime_upgrade`

Upgrades the JVM on a managed host. Supports OpenJDK and Amazon Corretto distributions. The executor detects running JVM processes before making any changes and records them in the execution result so operators can verify application behaviour after the upgrade.

**Phases:**

1. Preflight — detect the current Java version via `java -version`, list all running JVM processes, confirm the target distribution package is available in the configured package repository
2. Snapshot — take an EBS or filesystem snapshot of the host
3. Install new runtime — install the target Java version using the host package manager (`apt`, `yum`, or `dnf`)
4. Update alternatives — configure `update-alternatives` (Linux) or update `JAVA_HOME` and `PATH` in the system profile to point to the new runtime
5. Verify — run `java -version` and confirm the output matches the target version
6. Report — emit the before/after version, list of JVM processes identified in preflight, and snapshot ID

**Rollback:** Restore the host from the snapshot taken in phase 2.

**Connector:** Nexplane Agent

---

## Python Runtime Upgrade

**Change type:** `python_runtime_upgrade`

Upgrades Python on a managed host. The executor enumerates installed pip packages before making changes and records them in the execution result to assist post-upgrade compatibility verification.

**Phases:**

1. Preflight — detect the current Python version via `python3 --version`, list running Python processes, enumerate installed pip packages using `pip3 list --format=json`, confirm the target version is available in the configured package repository
2. Snapshot — take an EBS or filesystem snapshot of the host
3. Install new runtime — install the target Python version using the host package manager
4. Update alternatives — configure `update-alternatives` or update `PATH` and any version-pinned symlinks to point to the new runtime
5. Verify — run `python3 --version` and confirm the output matches the target version
6. Report — emit the before/after version, list of running Python processes identified in preflight, pip package inventory, and snapshot ID

**Rollback:** Restore the host from the snapshot taken in phase 2.

**Connector:** Nexplane Agent
