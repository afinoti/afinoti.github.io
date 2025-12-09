---
title: AWS Configuration Quick Reference
author: afinoti
date: 2024-12-19 14:30:00 +0800
categories: [Blogging, References]
tags: [aws, cloud, configuration, devops]
pin: true
---

## What is it?

Quick reference guide for AWS configuration and common setup tasks. Copy-paste ready commands and snippets.

## AWS CLI Setup

### Installation & Basic Configuration
```bash
# Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure default profile
aws configure
# AWS Access Key ID: YOUR_ACCESS_KEY
# AWS Secret Access Key: YOUR_SECRET_KEY
# Default region: us-east-1
# Default output format: json
```


## Multiple Profiles
Set one profile for terminal and type chaws to choose AWS profile.
```shell
chaws 
aws configure
``` 

### Code (~/.bashrc)

```shell
AWS_PROFILES=( $(aws configure list-profiles) )
PROFILE=$(echo "${AWS_PROFILES[0]}")
echo "Selected profile: |$PROFILE|"
export AWS_PROFILE=$PROFILE
export AWS_DEFAULT_PROFILE=$PROFILE

# Change AWS Profile
chaws() {
    AWS_PROFILES=($(aws configure list-profiles))

    if [[ ${#AWS_PROFILES[@]} -eq 0 ]]; then
        echo "No AWS profiles found"
        return 1
    fi

    printf "\n!!Configured AWS Profiles:\n"
    for i in "${!AWS_PROFILES[@]}"; do
        echo "$((i + 1)): ${AWS_PROFILES[i]}"
    done

    printf "\nEnter profile number: "
    read -r choice

    # Validate input is a number and within range
    if ! [[ "$choice" =~ ^[0-9]+$ ]] || [[ $choice -lt 1 ]] || [[ $choice -gt ${#AWS_PROFILES[@]} ]]; then
        echo "Invalid selection"
        return 1
    fi

    PROFILE=$(echo "${AWS_PROFILES[choice-1]}" | xargs)
    echo "Selected profile: |$PROFILE|"
    export AWS_PROFILE=$PROFILE
    export AWS_DEFAULT_PROFILE=$PROFILE
    printf "\nAWS profile set to \"%s\"\n" "$AWS_PROFILE"
}
```

## Troubleshooting

### Common Issues
```bash
# Check current identity
aws sts get-caller-identity

# Verify region
aws configure get region

# Test connectivity
aws s3 ls

# Debug with verbose output
aws s3 ls --debug
```
