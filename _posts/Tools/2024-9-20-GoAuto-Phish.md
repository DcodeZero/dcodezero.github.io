---
title: GoAutoPhish
category: tools
excerpt: This project automates the creation and management of phishing campaigns using the GoPhish API. The script handles template management, sending profiles, and campaign creation with advanced features such as error handling, custom headers, and rate limiting.
header:
  overlay_color: "#000"
  overlay_filter: "0.75"
  overlay_image: /assets/images/sample/gophish/go-phish.webp
  teaser: /assets/images/sample/gophish/go-phish.webp
tags: python, GoPhish
toc: true
comments: true
---


# Automating Phishing Campaigns with GoPhish: A Complete Guide

## Introduction

In the realm of cybersecurity assessment, phishing simulations play a crucial role in evaluating and improving an organization's security awareness. Today, we'll explore a powerful automation tool that streamlines the process of creating and managing phishing campaigns using GoPhish's API. The [GoPhish-Automation](https://github.com/DcodeZero/Gophish-Automation) project offers a robust solution for security professionals looking to enhance their phishing simulation capabilities.

## Understanding GoPhish Automation

GoPhish Automation is a Python-based project that simplifies the creation and management of phishing campaigns. By leveraging the GoPhish API, it automates template management, sending profiles, and campaign creation while incorporating advanced features like error handling, custom headers, and rate limiting.

## Project Structure

The project follows a clean, organized structure:

```
/GoPhish-Automation/
│
├── /templates/
│   ├── template1.txt
│   ├── template2.txt
│   └── template3.txt
│
├── /logs/
│   └── phishing_campaign.log
│
├── config.json
│
└── configauto.py
```

### Key Components

1. **Templates Directory**: Stores email templates in text format
2. **Logs Directory**: Contains campaign execution logs
3. **Configuration File**: Manages campaign settings and credentials
4. **Main Script**: Handles the automation logic

## Setting Up the Environment

### Prerequisites

Before diving in, ensure you have:

- Python 3.x installed
- A running GoPhish server
- GoPhish API key
- Basic understanding of phishing campaigns

### Installation Steps

1. Clone the repository:
```bash
git clone https://github.com/DcodeZero/Gophish-Automation.git
cd Gophish-Automation
```

2. Install required packages:
```bash
pip install gophish
```

## Configuration Deep Dive

The `config.json` file is the heart of the configuration. Here's a detailed breakdown:

```json
{
  "api_key": "your_api_key_here",
  "host": "https://your_gophish_server_url",
  "template": {
    "is_base64": false,
    "send_all_types": false
  },
  "smtp_profiles": [
    {
      "name": "Profile1",
      "host": "smtp.example.com",
      "from_address": "phish@example.com",
      "username": "user1",
      "password": "password1",
      "ignore_cert_errors": true,
      "headers": [
        {"key": "X-Header", "value": "Foo Bar"}
      ]
    }
  ],
  "group": {
    "name": "Phishing Group",
    "targets": [
      {
        "email": "target1@example.com",
        "first_name": "John",
        "last_name": "Doe"
      }
    ]
  },
  "landing_page": {
    "name": "Phishing Page",
    "html": "base64_encoded_html_here",
    "is_base64": true,
    "capture_credentials": true,
    "capture_passwords": true
  }
}
```

### Configuration Components

1. **API Configuration**
   - API key authentication
   - GoPhish server host URL

2. **Template Settings**
   - Base64 encoding options
   - Multiple template type support

3. **SMTP Profiles**
   - Email server configurations
   - Custom header support
   - Certificate handling

4. **Target Groups**
   - Target recipient management
   - Personalization options

5. **Landing Pages**
   - HTML content configuration
   - Credential capture settings

## Creating Email Templates

Templates are stored in `.txt` files with a specific format:

```text
Subject: Your Compelling Subject Line
HTML: <html>
<body>
<p>Dear {FIRST_NAME},</p>
<p>Your phishing content here...</p>
</body>
</html>
```

### Template Best Practices

1. Use variables for personalization
2. Structure HTML properly
3. Include both plain text and HTML versions
4. Test templates before deployment

## Advanced Features

### 1. Error Handling and Logging
```python
try:
    # Campaign operation
    log.info("Campaign created successfully")
except Exception as e:
    log.error(f"Campaign creation failed: {str(e)}")
```

### 2. Rate Limiting
```python
def send_campaign(profile, delay=60):
    time.sleep(delay)
    # Campaign sending logic
```

### 3. Custom Headers
```python
headers = [
    {"key": "X-Priority", "value": "1"},
    {"key": "X-Custom", "value": "Value"}
]
```

## Campaign Execution

Running the automation is straightforward:

```bash
python AutoGoPhish.py
```

The script performs:

1. Template validation and processing
2. SMTP profile configuration
3. Group creation/update
4. Campaign launch with rate limiting
5. Comprehensive logging

## Monitoring and Logging

The script maintains detailed logs in `/logs/phishing_campaign.log`:

```text
2024-03-11 10:15:23 INFO: Starting campaign setup
2024-03-11 10:15:24 INFO: Template validated successfully
2024-03-11 10:15:25 INFO: Campaign 'Security Training' launched
2024-03-11 10:15:26 WARNING: Rate limit applied - waiting 60 seconds
```

## Best Practices

1. **Template Management**
   - Regularly update templates
   - Test with small groups first
   - Monitor spam score

2. **Rate Limiting**
   - Adjust based on SMTP provider
   - Monitor delivery rates
   - Implement progressive delays

3. **Error Handling**
   - Monitor logs regularly
   - Set up alerts for failures
   - Maintain backup configurations

## Security Considerations

1. **API Key Protection**
   - Store securely
   - Rotate regularly
   - Use environment variables

2. **SMTP Security**
   - Use encryption
   - Implement SPF/DKIM
   - Monitor for abuse

3. **Data Protection**
   - Encrypt sensitive data
   - Clean up after campaigns
   - Follow data retention policies

## Conclusion

The GoPhish-Automation project significantly streamlines the process of managing phishing campaigns. By automating routine tasks and implementing best practices, security teams can focus on analyzing results and improving security awareness programs.

## Resources

- [GitHub Repository](https://github.com/DcodeZero/Gophish-Automation)
- [GoPhish Documentation](https://docs.getgophish.com)
- [Python GoPhish Library](https://github.com/gophish/api-client-python)

---

*Note: This tool should only be used for legitimate security testing with proper authorization.*