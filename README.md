# Ansible Leapp Collection

## Overview

This collection provides Ansible roles you can use to perform RHEL in-place upgrades using the Leapp framework. Successfully executing upgrades at scale across a large RHEL estate demands a customized end-to-end automation approach tailored to meet the requirements of your enterprise environment. Use these roles as the foundation of your RHEL in-place upgrade automation solution.

## Custom Modifications

This implementation uses the `infra.leapp` collection with modifications:
- Each role is executed directly from the directory rather than being pulled from the collection.
- Custom remediation scripts have been added to the `tasks` directory in the `remediate` role.

## Roles

These are the roles included in the collection:

- [`analysis`] - Executes the Leapp pre-upgrade phase
- [`common`] - Used for local logging, mutex locking, and common variables
- [`parse_leapp_report`] - Reads pre-upgrade results and checks for inhibitors
- [`upgrade`] - Executes the Leapp OS upgrade
- [`remediate`] - Assists in the remediation of a system (RHEL 8 only)

## Handling Errors

Errors encountered during the upgrade process are handled as follows:
- Inhibitors found during the analysis phase will trigger a remediation process.
- If inhibitors persist after remediation, the playbook will revert configuration files and stop execution.
- Logs are cleaned up and rotated throughout the process to maintain consistency and debugging capabilities.

## Example Playbooks

### Step-by-Step Playbook Execution

#### Pre-upgrade Analysis
- Checks for existing logs (`check_ripu_log.yml`)
- Runs the `analysis` role to perform a Leapp pre-upgrade analysis
- Includes necessary handlers for logging and notifications

#### Generating and Analyzing Reports
- Generates a Leapp upgrade report (`generate_report.yml`)
- Runs an analysis to check for upgrade inhibitors (`inhibitor_check.yml`)

#### Remediation Phase (if inhibitors are found)
- Runs the `remediate` role to address inhibitors
- Re-runs the `analysis` role after remediation
- If inhibitors persist, resets configuration files and stops the playbook

#### Upgrade Execution
- If no inhibitors are present, proceeds with the `upgrade` role
- Performs a final cleanup of logs (`handlers/main.yml`)
- Resets configuration files post-upgrade (`reset_configs.yml`)

## Example Playbook


