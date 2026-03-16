---
title: Cobalt Strike Malleable C2 Profiles
category: Red-Team
excerpt: Hide from blue team like a pro by using malleable c2 profiles.
header:
  overlay_color: "#000"
  overlay_filter: "0.75"
  overlay_image: /assets/images/sample/homelab/malleablec2.jpg
  teaser: /assets/images/sample/homelab/malleablec2.jpg
tags: HomeLab IT RED-Team
toc: true
classes: wide
comments: true
---


# Overview


Beacon's HTTP indicators are controlled by a Malleable Command and Control (Malleable C2) profile. A Malleable C2 profile is a simple program that specifies how to transform data and store it in a transaction. The same profile that transforms and stores data, interpreted backwards, also extracts and recovers data from a transaction.

To use a custom profile, you must start a Cobalt Strike team server and specify your profile at that time.

```
./teamserver [external IP] [password] [/path/to/my.profile]
```

You may only load one profile per Cobalt Strike instance.

Malleable C2 is a domain specific language to redefine indicators in Beacon's communication. This repository is a collection of Malleable C2 profiles that you may use. These profiles work with Cobalt Strike 3.x.

### **Slack and Discord Simulation Profiles**

#### **Purpose**
These profiles are designed to simulate HTTP communication patterns resembling Slack and Discord traffic. They aim to improve operational security (OPSEC) by mimicking legitimate application behavior to evade detection in network monitoring environments.

---

   - The profiles utilize dynamic URIs for both HTTP GET and POST requests.
   - A metadata block is used to prepend and append channel-specific identifiers (e.g., `channel-<dynamic>-dynamic`) to create the appearance of real-time channel activity.

   - HTTP GET traffic uses a Slack-specific user-agent: `Slack/4.23.0 (Mac OS X 10.15.7)`.
   - HTTP POST traffic uses a Discord-specific user-agent: `Discord/1.0 (Linux; Android 10; Scale/3.0)`.

   - `spawnto_x86` and `spawnto_x64` are set to `taskhostw.exe`, a legitimate Windows process, instead of the highly scrutinized `svchost.exe`.
   - The `host_stage` parameter is disabled (`false`) to avoid exposing payloads to unauthorized parties.

   - The certificates mimic Slack and Discord services:
     - For Slack: `CN="slack.com"`, `O="Slack Technologies, LLC"`.
     - Valid for 365 days to appear realistic.

   - Authorization headers include tokens (`Bearer <bot-token>` and `Bearer <user-token>`) to simulate API usage.
   - Headers like `Content-Type`, `Accept`, and `User-Agent` are customized to match Slack and Discord API patterns.

   - GET and POST responses are encoded in Base64 and prefixed with keywords (e.g., `response_`, `msg_send_`) to mimic legitimate payloads.
   - The traffic mimics typical JSON API responses.


### You can find the C2-Profiles in my github repo: [C2-Profiles](https://github.com/DcodeZero/c2-profiles/)
{: .notice--info}



---
#### **Traffic Example**
**HTTP-GET Request:**
```http
GET /api/v9/channels/channel-123456-dynamic HTTP/1.1
Host: slack.com
Authorization: Bearer <bot-token>
User-Agent: Slack/4.23.0 (Mac OS X 10.15.7)
Accept: application/json
```

**HTTP-POST Request:**
```http
POST /api/v9/channels/channel-123456-dynamic HTTP/1.1
Host: discord.com
Authorization: Bearer <user-token>
User-Agent: Discord/1.0 (Linux; Android 10; Scale/3.0)
Content-Type: application/json
```
---

#### Real Time Example:

Once the profile is loaded with cobalt-strike. Create a listner with port 80. The reason to use port 80 is just for demonstration purposes. 

#### listner config

![2.png](/assets/images/sample/homelab/C2-profile/2.png)

#### Beacon call back from victim

![1.png](/assets/images/sample/homelab/C2-profile/1.png)


Open wireshark on the victim machine and check for traffic. 

![3.png](/assets/images/sample/homelab/C2-profile/3.png)

Select the destination address and port number.

![4.png](/assets/images/sample/homelab/C2-profile/4.png)

right click and check for follow TCP-stream

![5.png](/assets/images/sample/homelab/C2-profile/5.png)
