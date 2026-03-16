---
title: "Budget Hacking Labs"
layout: splash
permalink: /selfhosted-labs/
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.75"
  overlay_image: /assets/images/selfhosted-lab.svg
  actions:
    - label: "Get Early Access"
      url: "mailto:dcodezero@proton.me?subject=Budget Hacking Labs - Early Access"
---

<style>
.pricing-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 25px;
  margin: 40px 0;
}
.price-card {
  background: linear-gradient(145deg, #12121a 0%, #1a1a2e 100%);
  border: 2px solid #2a2a3e;
  border-radius: 16px;
  padding: 35px 30px;
  width: 280px;
  text-align: center;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}
.price-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, #ff0040, #ff4444);
  opacity: 0;
  transition: opacity 0.3s;
}
.price-card:hover {
  transform: translateY(-10px);
  border-color: #ff0040;
  box-shadow: 0 20px 40px rgba(255, 0, 64, 0.2);
}
.price-card:hover::before {
  opacity: 1;
}
.price-card.featured {
  border-color: #ff0040;
  transform: scale(1.05);
}
.price-card.featured::before {
  opacity: 1;
}
.price-card.featured:hover {
  transform: scale(1.05) translateY(-10px);
}
.price-card.coming-soon {
  opacity: 0.5;
  pointer-events: none;
}
.price-card.coming-soon:hover {
  transform: none;
  border-color: #2a2a3e;
  box-shadow: none;
}
.badge {
  position: absolute;
  top: 15px;
  right: -35px;
  padding: 5px 40px;
  font-size: 0.7em;
  font-weight: bold;
  transform: rotate(45deg);
  letter-spacing: 1px;
}
.badge-available {
  background: #00cc66;
  color: white;
}
.badge-soon {
  background: #666;
  color: #ccc;
}
.plan-name {
  font-size: 1.3em;
  font-weight: 600;
  color: #888;
  margin-bottom: 10px;
  text-transform: uppercase;
  letter-spacing: 2px;
}
.plan-subtitle {
  font-size: 0.85em;
  color: #666;
  margin-bottom: 15px;
}
.price-amount {
  font-size: 4em;
  font-weight: 800;
  color: #ff0040;
  line-height: 1;
  margin: 15px 0;
  text-shadow: 0 0 30px rgba(255, 0, 64, 0.3);
}
.price-amount .currency {
  font-size: 0.4em;
  vertical-align: super;
  color: #ff4444;
}
.price-amount .period {
  font-size: 0.25em;
  color: #666;
  font-weight: 400;
}
.plan-features {
  list-style: none;
  padding: 0;
  margin: 25px 0;
  text-align: left;
}
.plan-features li {
  padding: 10px 0;
  color: #aaa;
  border-bottom: 1px solid #2a2a3e;
  font-size: 0.9em;
}
.plan-features li:last-child {
  border-bottom: none;
}
.plan-features li::before {
  content: "→ ";
  color: #ff0040;
  font-weight: bold;
}
.roadmap-note {
  background: #12121a;
  border-left: 4px solid #ff8800;
  padding: 15px 20px;
  margin: 30px 0;
  border-radius: 0 8px 8px 0;
}
</style>

## Affordable AD Lab Access

Realistic Active Directory environments for hands-on practice. Built on [GOAD](https://github.com/Orange-Cyberdefense/GOAD) - the same lab framework used by red teamers worldwide.

No writeups. No hand-holding. Just lab access at a price that doesn't hurt.

---

## What You Get

- **Multi-VM Active Directory environment**
- **Pre-configured attack box** (Kali/Parrot)
- **WireGuard VPN access**
- **Self-service lab reset**

---

## Lab Tiers & Pricing

<div class="pricing-container">
  <div class="price-card featured">
    <div class="badge badge-available">AVAILABLE</div>
    <div class="plan-name">Mini</div>
    <div class="plan-subtitle">2 VMs • 1 Domain</div>
    <div class="price-amount">
      <span class="currency">$</span>5<span class="period">/mo</span>
    </div>
    <ul class="plan-features">
      <li>Domain Controller + Workstation</li>
      <li>Basic AD attacks</li>
      <li>Attack box included</li>
      <li>20 hours/month</li>
    </ul>
  </div>

  <div class="price-card coming-soon">
    <div class="badge badge-soon">COMING SOON</div>
    <div class="plan-name">Standard</div>
    <div class="plan-subtitle">3 VMs • 2 Domains</div>
    <div class="price-amount">
      <span class="currency">$</span>10<span class="period">/mo</span>
    </div>
    <ul class="plan-features">
      <li>Multi-domain environment</li>
      <li>Trust relationships</li>
      <li>Attack box included</li>
      <li>30 hours/month</li>
    </ul>
  </div>

  <div class="price-card coming-soon">
    <div class="badge badge-soon">COMING SOON</div>
    <div class="plan-name">Full</div>
    <div class="plan-subtitle">5 VMs • 2 Forests</div>
    <div class="price-amount">
      <span class="currency">$</span>20<span class="period">/mo</span>
    </div>
    <ul class="plan-features">
      <li>Multi-forest environment</li>
      <li>Complex attack paths</li>
      <li>Attack box included</li>
      <li>50 hours/month</li>
    </ul>
  </div>
</div>

<div class="roadmap-note">
  <strong>Scaling up:</strong> Starting with Mini labs. Standard and Full tiers will be added as infrastructure grows. Join now and get priority access to larger labs when they launch.
</div>

---

## Current Status

<span style="background: #ff8800; color: #000; padding: 8px 20px; border-radius: 20px; font-weight: bold; font-size: 0.9em;">BETA - Mini Labs Available</span>

Limited slots. Testing infrastructure and onboarding before expanding.

---

## Interested?

<a href="mailto:dcodezero@proton.me?subject=Budget Hacking Labs - Early Access" class="btn btn--danger btn--large">Request Access</a>

Questions? Same email.

---

## Related Posts

{% assign posts = site.posts | where_exp: "post", "post.categories contains 'SelfHosted-Labs'" %}
{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}
