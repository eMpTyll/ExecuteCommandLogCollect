# Linux-Command-Log-Collector

## Overview

This project helps in collecting logs of executed commands on a Linux system. The solution uses `LD_PRELOAD` to intercept command execution and `auditd` to track command execution events.

## Prerequisites

Before you start, ensure you have the following tools installed on your system:

- GCC Compiler
- Audit Daemon (`auditd`)

## Step-by-Step Setup

### 1. Compile the Logger

First, compile the shared library `exec_logger.so` from the source file `exec_logger.c`. This library will be used to intercept command executions.

```bash
gcc -shared -fPIC -o exec_logger.so exec_logger.c -ldl
```

### 2. Set Up LD_PRELOAD

Set up `LD_PRELOAD` to use the compiled library. This will ensure that the logger library is loaded before other libraries, allowing it to intercept command executions.

```bash
export LD_PRELOAD=/path/to/exec_logger.so
```

To make this change permanent, add it to your system profile:

```bash
echo 'export LD_PRELOAD=/path/to/exec_logger.so' | sudo tee -a /etc/profile
```

### 3. Install `auditd` and `audispd-plugins`

Update your package list and install `auditd` and its plugins:

```bash
sudo apt-get update
sudo apt-get install auditd audispd-plugins
```

### 4. Configure `auditd` Rules

To monitor command executions, you need to add audit rules. Open the audit rules file and add the following lines:

```bash
sudo nano /etc/audit/rules.d/audit.rules
```

Add these rules to the file:

```bash
-w /usr/bin/sudo -p x -k command_executions
-w /bin/ -p x -k command_executions
-w /usr/bin/ -p x -k command_executions
-w /usr/sbin/ -p x -k command_executions
```

These rules will track execution events for common command binaries and `sudo`.

### 5. Restart `auditd`

Finally, restart the `auditd` service to apply the new rules:

```bash
sudo service auditd restart
```

## Verification

After completing the setup, you can test it by executing some commands and checking the logs:

```bash
# Execute some commands
ls
pwd

# Check the logs
sudo ausearch -sc execve
```

You should see entries related to the executed commands in the audit logs.

## Troubleshooting

- If you encounter issues with `LD_PRELOAD`, ensure the path to `exec_logger.so` is correct and that the file has the appropriate permissions.
- Verify that `auditd` is running and that the audit rules are correctly applied by checking the status and rules with the following commands:

    ```bash
    sudo service auditd status
    sudo auditctl -l
    ```

Feel free to open an issue or submit a pull request if you encounter any problems or have suggestions for improvements.