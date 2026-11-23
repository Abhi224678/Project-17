
# Project 17 – Log Forwarding to CloudWatch

## 1. Project Overview

**Objective:** Implement centralized log management using Amazon CloudWatch by installing and configuring the Amazon CloudWatch Agent on a Linux virtual machine.

The CloudWatch Agent collects system and authentication logs from the Ubuntu server and forwards them to Amazon CloudWatch Logs for centralized monitoring and management.

## 2. Technologies Used

| Technology | Purpose |
|---|---|
| Ubuntu 24.04 LTS | Linux operating system |
| AWS CloudWatch | Centralized log management |
| CloudWatch Agent | Log collection and forwarding |
| AWS CLI | AWS resource management |
| JSON | Agent log collection configuration |
| TOML | Agent credential configuration |

## 3. Environment Details

- Operating System: Ubuntu 24.04.3 LTS
- AWS Region: ap-south-1 (Mumbai)
- CloudWatch Agent Version: 1.300075.0b2061
- Virtual Machine: Azure-hosted Linux VM

## 4. Architecture

The implementation follows this workflow:

```text
Ubuntu Linux VM
      |
      |
      v
CloudWatch Agent
      |
      |
      v
Log Collection
      |
      +----------------------+
      |                      |
      v                      v
  System Logs          Authentication Logs
  /var/log/syslog      /var/log/auth.log
      |                      |
      v                      v
      +----------+-----------+
                 |
                 v
          AWS CloudWatch
                 |
          +------+------+
          |             |
          v             v
    System Log      Authentication
      Group           Log Group
```

## 5. Implementation Steps

### Step 1: Create Project Directory

```bash
mkdir -p ~/capstone-projects/17-log-forwarding-cloudwatch
cd ~/capstone-projects/17-log-forwarding-cloudwatch
```

### Step 2: Download CloudWatch Agent

Downloaded the official Amazon CloudWatch Agent package for Ubuntu.

```bash
wget https://amazoncloudwatch-agent.s3.amazonaws.com/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
```

### Step 3: Install CloudWatch Agent

Installed the downloaded package using the Debian package manager.

```bash
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
```

### Step 4: Configure Log Collection

Created the CloudWatch Agent configuration file:

`/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`

Configuration:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/syslog",
            "log_group_name": "project17-system-logs",
            "log_stream_name": "{instance_id}-syslog",
            "timezone": "Local"
          },
          {
            "file_path": "/var/log/auth.log",
            "log_group_name": "project17-auth-logs",
            "log_stream_name": "{instance_id}-auth",
            "timezone": "Local"
          }
        ]
      }
    }
  }
}
```

**Configuration explanation:**

- `/var/log/syslog`: Collects general system logs.
- `/var/log/auth.log`: Collects authentication-related logs.
- `log_group_name`: Defines the destination CloudWatch log group.
- `log_stream_name`: Defines the log stream name.
- `timezone`: Specifies the local timezone.

### Step 5: Configure AWS Credentials

Since the virtual machine is hosted on Azure, AWS credentials were configured explicitly for the CloudWatch Agent.

The AWS credentials were made available to the root user with restricted permissions.

Configured file:

`/opt/aws/amazon-cloudwatch-agent/etc/common-config.toml`

```toml
[credentials]
shared_credential_profile = "default"
shared_credential_file = "/root/.aws/credentials"
```

The AWS region was configured as:

`ap-south-1`

### Step 6: Start the CloudWatch Agent

Restarted the CloudWatch Agent to load the configuration.

```bash
sudo systemctl restart amazon-cloudwatch-agent
```

### Step 7: Verify Agent Status

Executed the following command:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
```

**Observed result:**

- Status: running
- Configuration status: configured
- Version: 1.300075.0b2061

## 6. CloudWatch Log Groups

The following log groups were configured in AWS CloudWatch:

| Log Group | Log Source |
|---|---|
| project17-system-logs | /var/log/syslog |
| project17-auth-logs | /var/log/auth.log |

## 7. Verification

### Verify System Log Stream

```bash
aws logs describe-log-streams \
  --log-group-name project17-system-logs \
  --region ap-south-1
```

**Observed log stream:**

`i-UNKNOWN-syslog`

### Verify Authentication Log Stream

```bash
aws logs describe-log-streams \
  --log-group-name project17-auth-logs \
  --region ap-south-1
```

**Observed log stream:**

`i-UNKNOWN-auth`

Both log groups successfully received log events.

**Note:** The `i-UNKNOWN` identifier appears because the virtual machine is hosted on Azure rather than Amazon EC2. This does not prevent log forwarding.

## 8. Deliverables

The following screenshots demonstrate the completed implementation:

1. CloudWatch Agent installation on Ubuntu.
2. CloudWatch Agent running and configured status.
3. CloudWatch Log Groups in the AWS Console.
4. System log stream displaying incoming events.
5. Authentication log stream displaying incoming events.

## 9. Final Result

Successfully installed and configured the Amazon CloudWatch Agent on Ubuntu 24.04.3 LTS.

System logs and authentication logs were successfully forwarded to AWS CloudWatch in the Mumbai region (`ap-south-1`).

The project demonstrates centralized log collection and monitoring using Amazon CloudWatch.

---
