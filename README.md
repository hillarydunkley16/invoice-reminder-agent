# AI-Powered Invoice Reminder Workflow with n8n + Gemini

This project automates overdue invoice reminders using n8n, Google Sheets, Gmail, and Gemini AI. The workflow generates personalized reminder emails, routes them through a human approval step, and sends them sequentially to customers.

## Features
- Reads overdue invoices from Google Sheets
- Uses Gemini AI to generate personalized email reminders
- Structured JSON output parsing
- Human approval workflow using Gmail Send & Wait
- Sequential email processing with Loop Over Items
- Dynamic email generation per customer

## Tech Stack
- n8n
- Google Gemini API
- Gmail API
- Google Sheets API
- Structured Output Parser

## Workflow Architecture
1. Pull overdue invoices from Google Sheets
2. Aggregate invoice records
3. Generate AI email drafts
4. Parse structured JSON output
5. Split customer records into individual items
6. Human approval step
7. Send approved emails

## Problem Solving 
