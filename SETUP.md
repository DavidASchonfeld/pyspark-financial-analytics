# Environment Setup Guide

This file documents the one-time setup steps needed to run this notebook locally.

---

## Step 1: Install Java (Required by PySpark)

PySpark is built on Apache Spark, which runs on the Java platform. Even though you write Python, Spark itself is Java software — so Java must be installed for PySpark to start at all.

**Which version:** Java 21 (the newest Long-Term Support release validated by PySpark 4.x)

**Which distribution:** Eclipse Temurin — a free, fully open-source build of Java with no licensing restrictions or subscriptions. It is maintained by the Eclipse Foundation, a well-known vendor-neutral organisation. You can verify this at eclipse.org.

**Why not Oracle JDK:** Oracle's JDK has historically changed its licensing terms, creating ambiguity about what is free vs. paid. Temurin has no such ambiguity — it is always free.

**Download:** https://adoptium.net/temurin/releases/?version=21
Select: macOS -> x64 -> .pkg, then run the installer.

---

## Step 2: Configure VS Code / Pylance for PySpark

Pylance is VS Code's Python language server — it provides real-time error highlighting and auto-complete. Out of the box it does not know about the project's virtual environment, so it will flag every PySpark import as an error. The steps below fix that permanently.

### Key concept: VS Code workspace root matters

Pylance reads its settings from a `.vscode/settings.json` file placed **directly inside whichever folder VS Code has open as its workspace root** — the top-level folder shown in the Explorer panel on the left.

If VS Code is opened at a parent folder (e.g. `PySpark/`) rather than the project subfolder (`pyspark-financial-analytics/`), any `settings.json` inside the project subfolder is completely ignored by Pylance.

**Always verify the workspace root** by checking the folder name at the very top of the VS Code Explorer panel before troubleshooting Pylance errors.

### Step 2a: Create the virtual environment and install dependencies

From a terminal, run the following from inside the project folder:

```bash
python3.13 -m venv .venv
.venv/bin/pip install pyspark ipykernel requests python-dotenv
```

`ipykernel` is required so VS Code can register the virtual environment as a Jupyter kernel. `python-dotenv` lets the notebook load API keys and other secrets from a `.env` file on disk, keeping sensitive values out of the notebook code itself.

### Step 2b: Create `.vscode/settings.json` at the workspace root

Create a file at `.vscode/settings.json` inside the workspace root folder. The `${workspaceFolder}` placeholder below automatically expands to wherever VS Code has the project open — no manual path editing needed:

```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
    "python.analysis.extraPaths": [
        "${workspaceFolder}/.venv/lib/python3.13/site-packages"
    ],
    "python.analysis.diagnosticSeverityOverrides": {
        "reportUndefinedVariable": "none"
    }
}
```

`reportUndefinedVariable` is set to `none` because Jupyter notebooks split code across cells — a variable defined in one cell appears "undefined" to Pylance when referenced in another cell, which is a false positive.

### Step 2c: Register the project kernel and tell VS Code which Python to use

Run this from inside the project folder to register the project's `.venv` as a named Jupyter kernel:

```bash
.venv/bin/python -m ipykernel install --user --name pyspark-venv --display-name "PySpark (venv)"
```

This creates a kernel entry that points explicitly to this project's `.venv`. Without this step, VS Code uses the generic `python3` kernel name, which it may resolve to a different `.venv` found elsewhere on disk — causing `ModuleNotFoundError` for packages that are installed locally but not globally.

Next, select the interpreter manually so VS Code's Python extension also uses the right environment:

1. Press `Cmd+Shift+P` and type **"Python: Select Interpreter"**, then press Enter.
2. Choose the entry that shows this project's `.venv` path (e.g. `./.venv/bin/python`).

**When opening the notebook**, always confirm the kernel shown in the top-right corner of VS Code reads **"PySpark (venv)"**, not just `.venv` or `python3`. If it shows anything else, click it and switch to **"PySpark (venv)"** — an ambiguously named kernel can silently use the wrong Python and break imports.

### Step 2d: Reload VS Code

Press `Cmd+Shift+P` → **"Developer: Reload Window"**.

After the reload, Pylance will use the virtual environment's Python (which has PySpark installed) and the red underlines under import statements will be gone.

**If errors still show up:** Run `Cmd+Shift+P` → **"Python: Restart Language Server"**. On rare occasions VS Code rejects the symlinked `.venv/bin/python` entry — if that happens, point `python.defaultInterpreterPath` directly to `/usr/local/opt/python@3.13/bin/python3.13` instead.

---

## Why the Notebook Sets JAVA_HOME in Code

After installing Java, the notebook still needs to tell PySpark exactly where to find it. Jupyter kernels launch in a clean environment that does not inherit the path settings from your terminal, so the path must be set explicitly inside the notebook before Spark starts.

This is handled at the top of Step 1 in the notebook:

```python
import os
os.environ.setdefault("JAVA_HOME", "/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home")  # Eclipse Temurin 21.0.10
```

`setdefault` is used so this line has no effect on Databricks or any environment that already has `JAVA_HOME` configured — it only fills in the value when it is missing.

---

## Troubleshooting: Cells Appear Frozen or Take Unexpectedly Long

### Why a fast cell can appear stuck

Jupyter runs cells **sequentially** — one at a time, in the order they are queued. If a slow or hung cell is already running, any cell queued after it (even a trivial `print` statement) will show a spinning `[*]` indicator and wait until the earlier cell finishes or is cancelled.

This commonly happens after accidentally clicking **"Execute Cells Above"** or **"Execute Cells Below"** in VS Code — those buttons queue every cell in the notebook at once, including the SparkSession startup cell, which takes 60–90 seconds on first run.

### How to clear a stuck queue

**Option 1 — Interrupt only (preferred):** Click the **square stop button (⬛)** in the VS Code toolbar, or press **`Cmd+.`** (Command + period). This cancels the currently running cell and clears all queued cells without losing any variables already defined in the session.

**Option 2 — Restart VS Code:** Closing and reopening VS Code kills the kernel process entirely and clears the queue. Use this as a last resort — it resets all variables and requires re-running cells from the top.

### What the JVM startup messages mean

When Step 1 runs successfully you will see output that mixes lines from two different sources:

**Lines from your cell code (Python stdout):**
- `Starting Spark (JVM startup takes 30-60 seconds on first run — please wait)...` — the progress message added to Step 1 so you know the cell is running normally
- `Setup complete! ...`, `Spark version: ...`, `AQE Status: ...` — the confirmation prints at the bottom of Step 1

**Lines printed by the JVM and Spark internals (stderr):**
- `WARNING: Using incubator modules: jdk.incubator.vector` — Java 21 advertising an experimental feature it activated; harmless
- `Using Spark's default log4j profile: ...` and `Setting default log level to "WARN".` — Spark's internal logging system starting up; expected every run
- `WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable` — Spark tried to load optimised native Hadoop libraries, could not find them on macOS, and fell back to pure-Java equivalents; this has no effect on correctness or performance for this project

VS Code's notebook captures both Python stdout and JVM stderr from a running cell and displays them together in a single output block below the cell. This is why the JVM lines appear mixed in with the Python print lines — they all belong to the same cell, just from different output channels.

None of these messages indicate a problem. If Step 1 ends with `Setup complete!` and a Spark version number, the session started correctly.

---

### Diagnostic commands

Run these in an isolated cell to verify the environment is configured correctly:

```python
import sys, os
print("Python:", sys.executable)
print("JAVA_HOME:", os.environ.get("JAVA_HOME"))
print("sys.path:", sys.path)
```