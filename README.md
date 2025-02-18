# Ansible Leapp Automation

## Overview

This collection provides Ansible roles to automate the in-place upgrade process of Red Hat Enterprise Linux (RHEL) using the Leapp framework. It is designed to simplify and streamline the execution of upgrades across large-scale RHEL environments. By leveraging the roles in this collection, you can create a customized, enterprise-ready solution to handle RHEL upgrades in a consistent, repeatable way.

## Custom Modifications

This implementation builds upon the `infra.leapp` collection with the following modifications:
- Each role is executed directly from the directory rather than being pulled from the collection.
- Custom remediation scripts have been added to the `tasks` directory in the `remediate` role to address specific upgrade issues encountered during RHEL 8 upgrades.

## Roles

The collection includes the following roles:

### [`analysis`]
- **Purpose**: Executes the pre-upgrade analysis phase of the Leapp framework.
- **Description**: Runs the Leapp analysis to assess the system’s readiness for the upgrade. This role checks for potential inhibitors, software dependencies, and other issues that may block the upgrade process.

### [`common`]
- **Purpose**: Provides common tasks and utilities for the upgrade process.
- **Description**: This role is responsible for setting up logging, mutex locking, and defining common variables that are used throughout the upgrade process. It also ensures the consistency of shared resources across roles.

### [`parse_leapp_report`]
- **Purpose**: Reads and processes the Leapp analysis report.
- **Description**: After the analysis phase, this role parses the Leapp report to check for any inhibitors that would prevent the upgrade. It helps to identify issues that require remediation before proceeding.

### [`upgrade`]
- **Purpose**: Executes the in-place upgrade using Leapp.
- **Description**: This role handles the upgrade of the RHEL system to a newer version. It ensures that the system is upgraded correctly and performs the necessary cleanup tasks following the upgrade process.

### [`remediate`]
- **Purpose**: Assists in the remediation of system issues found during the upgrade process (RHEL 8 only).
- **Description**: This role contains custom remediation scripts that address specific issues encountered during the upgrade. These scripts are executed when inhibitors are detected in the pre-upgrade phase and must be resolved before the upgrade can proceed.

## Handling Errors

Errors encountered during the upgrade process are managed as follows:
- **Inhibitors Detected**: If inhibitors are found during the analysis phase, the remediation role is triggered to address them.
- **Persistent Inhibitors**: If the inhibitors persist after remediation, the playbook will revert configuration files to their previous state and halt execution to avoid any further issues.
- **Log Management**: Logs are consistently cleaned up and rotated throughout the process to maintain clarity, ensure ease of troubleshooting, and avoid file system clutter.

## Example Playbooks

### Step-by-Step Playbook Execution

1. **Pre-upgrade Analysis**  
   - Runs `check_ripu_log.yml` to check for existing logs.
   - Executes the `analysis` role to perform the Leapp pre-upgrade analysis.
   - Includes handlers for logging and notification.

2. **Generating and Analyzing Reports**  
   - Generates the Leapp upgrade report with `generate_report.yml`.
   - Runs `inhibitor_check.yml` to analyze the report for potential inhibitors.

3. **Remediation Phase (if inhibitors are found)**  
   - Runs the `remediate` role to address detected inhibitors.
   - Re-runs the `analysis` role to check if the inhibitors have been resolved.
   - If inhibitors persist, resets configuration files and halts further execution.

4. **Upgrade Execution**  
   - If no inhibitors are found, the `upgrade` role is executed to carry out the upgrade.
   - Cleans up logs and resets configuration files post-upgrade using `handlers/main.yml` and `reset_configs.yml`.

## Custom Remediation Scripts

These custom scripts are added to the `roles/remediate/tasks/` directory and are triggered when specific issues are detected during the pre-upgrade phase:

### 1. **leapp_firewalld_allowzonedrifting**
   - **Purpose**: Ensures that firewall settings allow zone drifting during the upgrade.
   - **Functionality**: Prevents conflicts when changes in firewall zones occur, ensuring smoother transitions during the upgrade process.

### 2. **firewalld_check_allow_zone_drifting**
   - **Purpose**: Verifies that the firewall allows zone drifting.
   - **Functionality**: Checks the firewall configuration for zone drift compatibility to avoid issues during the upgrade.

### 3. **leapp_vdo_check_needed**
   - **Purpose**: Verifies the need for a Virtual Data Optimizer (VDO) check on block devices.
   - **Functionality**: If necessary sections are missing in the `answerfile`, this script will create the file to ensure that the system is properly configured for VDO before proceeding with the upgrade.

### 4. **leapp_auditd_grub**
   - **Purpose**: Resolves issues related to the auditing subsystem and kernel compatibility.
   - **Functionality**: Handles errors related to audit login UID resets. If auditing is enabled on an old kernel, it advises upgrading the kernel or disabling auditing (`audit=0`) on the kernel command line to prevent conflicts with containerized environments.

### 5. **leapp_fapolicyd_disable**
   - **Purpose**: Disables the FAPolicyD service during the upgrade.
   - **Functionality**: Fixes failures related to trust management commands like `su - -c update-ca-trust`. By disabling FAPolicyD, it ensures that the system update and certificate trust updates complete successfully.

### 6. **leapp_gpgcheck_dnf**
   - **Purpose**: Addresses issues with GPG checking and proxy configurations in DNF/YUM.
   - **Functionality**: Resolves problems caused by misconfigured proxy settings in the `dnf.conf` file. If necessary, the script uses a custom `dnf.conf` from `/etc/leapp/files/dnf.conf` to ensure compatibility with the target system.

## Conclusion

The Ansible Leapp Collection provides a comprehensive solution for automating the RHEL in-place upgrade process, from pre-upgrade analysis to final cleanup. By customizing and extending the Leapp framework, this collection ensures a smooth, automated upgrade experience across your enterprise RHEL environment.
