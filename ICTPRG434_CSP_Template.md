---
title: "ICTPRG434 Collaborative Software Project"
author: 
 - Jarna Salonen (13229846)
 - Shehreen Haider (13247624)
 - Nick Woolcock
---

# Executive Summary

This report details the development of a host-based domain checker aimed at enhancing the management and monitoring of domain assets across various hosting platforms. The project leverages automation and advanced querying techniques to meet the needs of administrators and stakeholders in domain infrastructure oversight.

# Project Overview

## Introduction

This project introduces a comprehensive host-based domain checker, designed to streamline the management and monitoring of domain assets across diverse hosting platforms. By harnessing automation and leveraging advanced querying techniques, this tool aims to address the evolving needs of administrators and stakeholders tasked with overseeing domain infrastructures.

## Team Structure and Roles

Our team was organized to maximize efficiency and ensure clear role distribution. The roles and responsibilities were as follows:

- **Jarna Salonen (13229846)**: Project Manager. Responsible for project oversight, code development, and testing.
- **Shehreen Haider (13247624)**: Lead Developer. Responsible for project oversight, code development, and testing.
- **Nick Woolcock**(): Documentation Specialist and Tester. Responsible for creating project documentation and conducting rigorous testing.

## Problem Analysis and Requirements

We analyzed the need for a domain checker that automates the process of managing domain assets. The requirements for the solution included:

The ability to accept user-defined options for blocking specific content types (advertisements, malware, tracking).
Reading URLs of domain block-lists from a configuration file.
Downloading, parsing, and merging hosts files into a unique list of domains.
Backing up the original hosts file with the option to restore it.
Appending blocked domains to the existing hosts file without replacing it.
Ensuring cross-platform compatibility for Windows and Linux.

# Design Process

## Conceptual Design

The initial concept focused on a user-friendly tool that automates domain management tasks. Key considerations included ease of use, reliability, and extensibility.

## Algorithm Design

Our algorithm parses and updates the hosts file based on user preferences. Below is a flowchart representing the algorithm.

flowchart LR
 A(Start) --> B[Input Preferences]
 B --> C[Download Block Lists]
 C --> D[Parse Lists]
 D --> E[Backup Hosts File]
 E --> F[Append to Hosts File]
 F --> G(End)

```mermaid
flowchart LR
    A(Start) --> B[Input Preferences]
    B --> C[Download Block Lists]
    C --> D[Parse Lists]
    D --> E[Backup Hosts File]
    E --> F[Append to Hosts File]
    F --> G(End)
```

```
FUNCTION update_hosts_file(preferences):
    IF preferences NOT SELECTED:
        RETURN "Error: No preferences selected"
    URLs = GET_URLs_FROM_CONFIG(preferences)
    domains = DOWNLOAD_AND_PARSE_LISTS(URLs)
    hosts_path = GET_HOSTS_FILE_PATH()
    BACKUP_HOSTS_FILE(hosts_path)
    APPEND_TO_HOSTS_FILE(hosts_path, domains)
    RETURN "Success: Hosts file updated"
```

## Software Design

The software is designed in Python due to its simplicity and extensive library support. The architecture includes modules for configuration management, URL fetching, file manipulation, and user interaction through a GUI.

# Implementation

## Code Development

The code developed for this project includes functions for downloading and parsing domain lists, backing up and updating the hosts file, and a GUI for user interaction. The main script integrates these components.

```python
import os
import platform
import shutil
import requests
import re
import json
import tkinter as tk
from tkinter import messagebox
from datetime import datetime

CONFIG_FILE = 'config.json'

def get_hosts_file_path():
    return r'D:\test\hosts' if platform.system() == 'Windows' else '/etc/hosts'

def backup_hosts_file(hosts_path):
    timestamp = datetime.now().strftime('%Y%m%d%H%M%S')
    backup_path = f"{hosts_path}_backup_{timestamp}"
    shutil.copy2(hosts_path, backup_path)
    shutil.copy2(hosts_path, 'hosts_last_used')
    return backup_path

def restore_hosts_file():
    hosts_path = get_hosts_file_path()
    if os.path.exists('hosts_last_used'):
        shutil.copy2('hosts_last_used', hosts_path)
        messagebox.showinfo("Restore", "Hosts file restored to the last used state.")
    else:
        messagebox.showerror("Restore", "No backup found to restore.")

def download_and_parse_lists(urls):
    domains = set()
    for url in urls:
        try:
            response = requests.get(url)
            response.raise_for_status()
            # Debug print to check the content of the response
            print(f"Content from {url}:\n{response.text[:500]}...\n")  # Print first 500 characters
            domains.update(re.findall(r'^\s*(?:0\.0\.0\.0|127\.0\.0\.1)\s+(\S+)', response.text, re.MULTILINE))
        except requests.RequestException as e:
            print(f"Failed to download {url}: {e}")
    return domains


def append_to_hosts_file(hosts_path, domains):
    with open(hosts_path, 'a') as hosts_file:
        hosts_file.write('\n'.join(f"0.0.0.0 {domain}" for domain in domains) + '\n')

def update_hosts_file(block_ads, block_malware, block_tracking, block_malicious):
    with open(CONFIG_FILE, 'r') as file:
        config = json.load(file)

    urls = []
    if block_ads:
        urls.extend(config.get('ad_block_lists', []))
    if block_malware:
        urls.extend(config.get('malware_block_lists', []))
    if block_tracking:
        urls.extend(config.get('tracking_block_lists', []))
    if block_malicious:
        urls.extend(config.get('malicious_block_lists', []))

    if not urls:
        messagebox.showerror("Error", "No block lists selected.")
        return

    domains = download_and_parse_lists(urls)
    hosts_path = get_hosts_file_path()
    backup_hosts_file(hosts_path)
    append_to_hosts_file(hosts_path, domains)
    messagebox.showinfo("Success", "Hosts file updated successfully.")

def create_gui():
    root = tk.Tk()
    root.title("Hosts-Based Domain Blocker")

    tk.Label(root, text="Select the types of content to block:").pack(pady=10)

    block_ads = tk.BooleanVar()
    block_malware = tk.BooleanVar()
    block_tracking = tk.BooleanVar()
    block_malicious = tk.BooleanVar()

    tk.Checkbutton(root, text="Advertisements", variable=block_ads).pack(anchor='w')
    tk.Checkbutton(root, text="Malware", variable=block_malware).pack(anchor='w')
    tk.Checkbutton(root, text="Tracking", variable=block_tracking).pack(anchor='w')
    tk.Checkbutton(root, text="Malicious", variable=block_malicious).pack(anchor='w')

    tk.Button(root, text="Update Hosts File", command=lambda: update_hosts_file(block_ads.get(), block_malware.get(), block_tracking.get(), block_malicious)).pack(pady=10)
    tk.Button(root, text="Restore Hosts File", command=restore_hosts_file).pack(pady=10)

    root.mainloop()

if __name__ == '__main__':
    create_gui()
## Testing and Debugging
```

## Testing and Debugging -Jarna

Testing involved verifying the program's ability to download block lists, parse domain entries, back up the hosts file, and update it without issues. We used a combination of unit tests and manual testing to ensure robustness.

# Collaboration and Project Management

## Meeting Notes

Regular team meetings were held to discuss progress, address challenges, and plan upcoming tasks. Decisions and action items were documented for reference.

## Project Management Tools and Techniques

We used tools such as Trello for task management, GitHub for version control, and Slack for communication. Agile methodologies were employed to maintain flexibility and ensure timely delivery of project milestones.

# Documentation

## Developer Documentation -Jarna

The developer documentation includes detailed explanations of the code structure, functions, and modules. This ensures future developers can easily understand and maintain the project.

## User Documentation -shehreen

A separate user manual has been created to guide end-users on how to install, configure, and use the domain blocker tool. This document is available in the project repository.

# Reflection and Conclusion

The project presented several challenges, including ensuring cross-platform compatibility and handling various edge cases during domain list parsing. These challenges were overcome through thorough testing and iterative development. The project successfully met its objectives, providing a reliable tool for managing domain block lists. Future improvements could include adding a GUI for configuring block lists and enhancing performance for large domain lists.

# Appendices

Links to the code repositories containing the final source code, user manuals, presentations, and other documentation:

[GitHub Repository]
