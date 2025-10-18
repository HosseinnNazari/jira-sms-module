# 📱 Jira No-Response SMS Automation

A **Jira ScriptRunner custom endpoint** that allows users to send an SMS directly from a Jira issue when the customer does not respond to calls.  
The sent message is also added automatically as a **comment** to the Jira issue, including the **issue key**, **phone number**, **message text**, and **contact date/time**.

---

## ✨ Features
- Custom Jira button to trigger SMS sending dialog  
- In-browser form with **phone number validation**  
- SMS text auto-generated using issue details (issue key & summary)  
- Sends SMS via **PayamSMS API**  
- Inserts a detailed comment in the Jira issue automatically  
- Works for Jira **administrators** and **users**  
- Mobile-friendly, responsive design

---

## 📸 Screenshots

### 1. Jira Issue with Custom Button
![Jira Custom Button](screenshots/jira-custom-button.png)

### 2. SMS Sending Dialog
![SMS Dialog](screenshots/sms-dialog.png)

### 3. Successful Comment Added
![Issue Comment](screenshots/issue-comment.png)

---

## 🛠️ Installation

### Prerequisites
- Jira Server/Data Center
- [ScriptRunner for Jira](https://www.adaptavist.com/doco/jira/latest)
- Access to **PayamSMS API**
- Jira admin rights

### Steps
1. Open **ScriptRunner > Custom Endpoints** in Jira admin panel
2. Create **2 endpoints**:
   - `sendSMSDialog` (GET) → Displays SMS form dialog
   - `sendSMS1` (POST) → Sends SMS and logs comment
3. Paste the following code into each endpoint:

---

## 📄 Code

### Endpoint 1 — SMS Dialog (GET)
```groovy
// (paste the 'sendSMSDialog' Groovy code here)
