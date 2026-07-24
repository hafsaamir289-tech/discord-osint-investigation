# Discord Infrastructure — Passive OSINT Investigation

A passive, non-intrusive reconnaissance case study on Discord's public-facing internet infrastructure — DNS configuration, email authentication, TLS/certificate posture, network/cloud footprint, and technology stack.

📄 **[Read the full report](./discord-osint-investigation-report.md)**

## Why this project

Reconnaissance is the first phase of any professional security assessment — before a pentester or red teamer ever touches a target, they map what's already publicly visible. This project was built to practice that exact skill set against a real, large-scale production service, using only open-source and publicly available data.

## What's covered

- DNS records (A, MX, TXT) and what they reveal about infrastructure design
- SPF & DMARC email authentication posture
- TLS/SSL configuration and Certificate Transparency log analysis
- ASN, cloud provider, and CDN footprint
- Public technology stack fingerprinting

## Methodology

100% passive OSINT. No scanning, no exploitation, no interaction with any non-public system or endpoint. All data was sourced from public DNS databases, Certificate Transparency logs, BGP/ASN registries, and Discord's own published engineering content.

**Tools/sources:** Connected.app, NSLookup.io, DNSChecker, DNSLookup, DMARC Manager, Hurricane Electric BGP Toolkit, Cloudflare Radar, AppSec Santa, Discord Engineering Blog.

## Disclaimer

This is an educational research project. No sensitive, non-public, or unauthorized information was accessed at any point. Every finding in this report is derived from data that is already publicly visible to anyone who looks.

## About

Written while learning cybersecurity fundamentals, as a first hands-on OSINT/reconnaissance exercise. Feedback welcome.
