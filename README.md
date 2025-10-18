 📱 Jira No-Response SMS Automation

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
![Jira Custom Button]<img width="1349" height="482" alt="4" src="https://github.com/user-attachments/assets/f76cd47d-c51e-4424-a8cc-9eb10145bb4a" />


### 2. SMS Sending Dialog
![SMS Dialog]<img width="1365" height="641" alt="2" src="https://github.com/user-attachments/assets/976c53e5-1f29-4882-9443-1181cc086860" />

### 3. Successful Comment Added
![Issue Comment]<img width="1366" height="641" alt="3" src="https://github.com/user-attachments/assets/3bc61af4-041a-4bd8-bae4-4fb47c181529" />



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
import com.atlassian.jira.component.ComponentAccessor
import com.onresolve.scriptrunner.runner.rest.common.CustomEndpointDelegate
import groovy.json.JsonBuilder
import groovy.transform.BaseScript

import javax.ws.rs.core.MultivaluedMap
import javax.ws.rs.core.Response

@BaseScript CustomEndpointDelegate delegate

// اندپوینت اول: نمایش فرم ارسال پیامک
sendSMSDialog(httpMethod: "GET", groups: ["jira-administrators", "jira-users"]) { MultivaluedMap queryParams, String body ->
    
    def issueId = queryParams.getFirst("issueId") as String
    
    if (!issueId) {
        return Response.status(Response.Status.BAD_REQUEST)
            .entity("Issue ID is required")
            .type("text/plain")
            .build()
    }
    
    def issueManager = ComponentAccessor.getIssueManager()
    def issue = issueManager.getIssueObject(issueId as Long)
    
    if (!issue) {
        return Response.status(Response.Status.NOT_FOUND)
            .entity("Issue not found")
            .type("text/plain")
            .build()
    }
    
    def smsText = """بدلیل عدم پاسخگویی مشتری، امکان پیگیری تسک ${issue.getKey()} وجود نداشت.
واحد پشتیبانی ورانگر"""
    
    def escapedSmsText = smsText.replace("&", "&amp;")
                                .replace("<", "&lt;")
                                .replace(">", "&gt;")
                                .replace('"', "&quot;")
    
    def htmlContent = """<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ارسال پیامک</title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            padding: 20px;
            direction: rtl;
            min-height: 100vh;
        }
        
        .form-container {
            max-width: 500px;
            margin: 0 auto;
            background: white;
            border-radius: 15px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.2);
            overflow: hidden;
        }
        
        .form-header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 25px;
            text-align: center;
        }
        
        .form-header h2 {
            font-size: 24px;
            margin: 0;
        }
        
        .form-body {
            padding: 30px;
        }
        
        .info-box {
            background: #f8f9fa;
            border-right: 4px solid #667eea;
            padding: 15px;
            margin-bottom: 25px;
            border-radius: 5px;
            font-size: 14px;
            line-height: 1.8;
        }
        
        .info-box strong {
            color: #667eea;
        }
        
        .field-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            color: #333;
            font-weight: 600;
            font-size: 14px;
        }
        
        input[type="text"] {
            width: 100%;
            padding: 12px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            font-size: 16px;
            transition: border-color 0.3s;
            direction: ltr;
            text-align: right;
        }
        
        input[type="text"]:focus {
            outline: none;
            border-color: #667eea;
        }
        
        input[type="text"].error {
            border-color: #dc3545;
        }
        
        .sms-preview {
            background: #f8f9fa;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            padding: 15px;
            white-space: pre-wrap;
            font-size: 14px;
            line-height: 1.8;
            color: #333;
            min-height: 100px;
        }
        
        .button-group {
            display: flex;
            gap: 10px;
            margin-top: 25px;
        }
        
        .btn {
            flex: 1;
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s;
        }
        
        .btn-primary {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        
        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
        }
        
        .btn-primary:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none;
        }
        
        .btn-secondary {
            background: #6c757d;
            color: white;
        }
        
        .btn-secondary:hover {
            background: #5a6268;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 13px;
            margin-top: 5px;
            display: none;
        }
        
        .error-message.show {
            display: block;
        }
        
        .success-message {
            background: #d4edda;
            border: 1px solid #c3e6cb;
            color: #155724;
            padding: 12px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: none;
        }
        
        .success-message.show {
            display: block;
        }
        
        .info-message {
            background: #cce5ff;
            border: 1px solid #b8daff;
            color: #004085;
            padding: 12px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: none;
        }
        
        .info-message.show {
            display: block;
        }
        
        .spinner {
            display: inline-block;
            width: 14px;
            height: 14px;
            border: 2px solid rgba(255,255,255,0.3);
            border-top-color: white;
            border-radius: 50%;
            animation: spin 0.6s linear infinite;
            margin-right: 8px;
        }
        
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body>
    <div class="form-container">
        <div class="form-header">
            <h2>📱 ارسال پیامک عدم پاسخگویی</h2>
        </div>
        
        <div class="form-body">
            <div class="info-box">
                <strong>شماره تسک:</strong> ${issue.getKey()}<br>
                <strong>موضوع:</strong> ${issue.getSummary()}
            </div>
            
            <div id="success-message" class="success-message">
                ✅ پیامک با موفقیت ارسال شد
            </div>
            
            <div id="info-message" class="info-message">
                ⏳ در حال ارسال پیامک...
            </div>
            
            <div id="error-message" class="error-message">
                ❌ خطا در ارسال پیامک
            </div>
            
            <form id="sms-form">
                <div class="field-group">
                    <label for="mobile">شماره موبایل مشتری:</label>
                    <input type="text" 
                           id="mobile" 
                           name="mobile" 
                           placeholder="09123456789" 
                           maxlength="11" 
                           pattern="09[0-9]{9}"
                           required>
                    <div id="mobile-error" class="error-message">
                        لطفاً شماره موبایل را به فرمت صحیح وارد کنید (09xxxxxxxxx)
                    </div>
                </div>
                
                <div class="field-group">
                    <label>پیش‌نمایش پیامک:</label>
                    <div class="sms-preview" id="sms-preview">${escapedSmsText}</div>
                </div>
                
                <input type="hidden" id="issue-id" value="${issueId}">
                <input type="hidden" id="sms-text" value="${escapedSmsText}">
                
                <div class="button-group">
                    <button type="button" class="btn btn-secondary" onclick="window.close()">انصراف</button>
                    <button type="submit" id="send-btn" class="btn btn-primary">ارسال پیامک</button>
                </div>
            </form>
        </div>
    </div>
    
    <script>
        \$(document).ready(function() {
            \$('#sms-form').on('submit', function(e) {
                e.preventDefault();
                
                // دریافت مقادیر از فرم
                var phoneNumber = \$('#mobile').val();
                var issueId = \$('#issue-id').val();
                var smsText = \$('#sms-text').val();
                
                // اعتبارسنجی شماره موبایل
                if (!phoneNumber.match(/^09[0-9]{9}\$/)) {
                    \$('#mobile-error').addClass('show');
                    \$('#mobile').addClass('error');
                    return false;
                }
                
                // پاک کردن خطاها
                \$('#mobile-error').removeClass('show');
                \$('#mobile').removeClass('error');
                \$('.success-message').removeClass('show');
                \$('#error-message').removeClass('show');
                
                // نمایش پیام در حال ارسال
                \$('#info-message').addClass('show');
                \$('#send-btn').prop('disabled', true).html('<span class="spinner"></span>در حال ارسال...');
                
                // ارسال درخواست AJAX
                \$.ajax({
                    url: '/rest/scriptrunner/latest/custom/sendSMS1',
                    type: 'POST',
                    contentType: 'application/json',
                    dataType: 'json',
                    headers: {
                        'X-Atlassian-Token': 'no-check'
                    },
                    data: JSON.stringify({
                        issueId: issueId,
                        phoneNumber: phoneNumber,
                        smsText: smsText
                    }),
                    success: function(response) {
                        console.log('Success:', response);
                        \$('#info-message').removeClass('show');
                        \$('#success-message').addClass('show');
                        \$('#send-btn').prop('disabled', false).html('ارسال پیامک');
                        
                        // بستن پنجره بعد از 2 ثانیه
                        setTimeout(function() {
                            window.close();
                        }, 2000);
                    },
                    error: function(xhr, status, error) {
                        console.error('Error:', xhr.responseText);
                        \$('#info-message').removeClass('show');
                        \$('#send-btn').prop('disabled', false).html('ارسال پیامک');
                        
                        var errorMsg = 'خطا در ارسال پیامک';
                        
                        try {
                            var errorData = JSON.parse(xhr.responseText);
                            errorMsg = errorData.body || errorData.error || errorMsg;
                        } catch(e) {
                            if (xhr.status === 0) {
                                errorMsg = 'عدم ارتباط با سرور';
                            } else if (xhr.status === 403) {
                                errorMsg = 'خطای دسترسی - لطفاً دوباره لاگین کنید';
                            } else if (xhr.status === 500) {
                                errorMsg = 'خطای سرور - لطفاً با پشتیبانی تماس بگیرید';
                            }
                        }
                        
                        \$('#error-message').html('❌ ' + errorMsg).addClass('show');
                    }
                });
                
                return false;
            });
            
            // اعتبارسنجی زنده شماره موبایل
            \$('#mobile').on('input', function() {
                var value = \$(this).val();
                if (value.length === 11) {
                    if (!value.match(/^09[0-9]{9}\$/)) {
                        \$('#mobile-error').addClass('show');
                        \$(this).addClass('error');
                    } else {
                        \$('#mobile-error').removeClass('show');
                        \$(this).removeClass('error');
                    }
                }
            });
        });
    </script>
</body>
</html>"""
    
    return Response.ok()
        .entity(htmlContent.toString())
        .type("text/html; charset=UTF-8")
        .build()
}

//========================================================================================================
import groovyx.net.http.ContentType
import groovy.json.JsonOutput
import groovy.json.JsonSlurper
import groovy.transform.BaseScript
import com.onresolve.scriptrunner.runner.rest.common.CustomEndpointDelegate
import javax.ws.rs.core.Response
import javax.ws.rs.core.MultivaluedMap
import java.net.HttpURLConnection
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.IssueManager
import com.atlassian.jira.issue.comments.CommentManager
import com.atlassian.jira.user.ApplicationUser

//@BaseScript CustomEndpointDelegate delegate

//--------------------------------------------------------------
// تابع دریافت توکن از سرویس پیامک
//--------------------------------------------------------------
def getToken(String systemName, String username, String password) {
    String loginUrl = "https://www.payamsms.com/auth/oauth/token"
    String requestBody = "systemName=${systemName}&username=${username}&password=${password}&scope=webservice&grant_type=password"

    HttpURLConnection connection = (HttpURLConnection) new URL(loginUrl).openConnection()
    connection.setRequestMethod("POST")
    connection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded")

    String authBasic = "YourCompanypanel:YourCompanypanelWebservice".bytes.encodeBase64().toString()
    connection.setRequestProperty("Authorization", "Basic ${authBasic}")

    connection.setDoOutput(true)
    connection.outputStream.withWriter("UTF-8") { writer -> writer << requestBody }

    try {
        int responseCode = connection.responseCode
        String responseText = (responseCode == 200) ? connection.inputStream.text : (connection.errorStream?.text ?: "")
        def parsed = new JsonSlurper().parseText(responseText ?: "{}")
        Map parsedMap = (parsed instanceof Map) ? (Map) parsed : [:]

        if (responseCode == 200 && parsedMap.containsKey("access_token")) {
            return parsedMap.get("access_token") as String
        } else {
            log.error("getToken failed: code=${responseCode}, resp=${responseText}")
            return null
        }
    } catch (Exception e) {
        log.error("getToken exception: ${e.message}", e)
        return null
    } finally {
        try { connection.disconnect() } catch (ignore) {}
    }
}

//--------------------------------------------------------------
// Endpoint برای ارسال پیامک عدم پاسخگویی
//--------------------------------------------------------------
sendSMS1(httpMethod: "POST", groups: ["jira-software-users", "jira-users"]) { MultivaluedMap queryParams, String body ->

    // --- خواندن پارامترها هم از JSON body و هم از Query params ---
    Map json = [:]
try {
    if (body) {
        json = (Map) new JsonSlurper().parseText(body ?: "{}")
    }
} catch(e) {
    return Response.status(400).entity("فرمت JSON نامعتبر است").build()
    }

    def issueIdRaw = json["issueId"] ?: queryParams?.getFirst("issueId")
    def phoneRaw   = json["phoneNumber"] ?: queryParams?.getFirst("phoneNumber")
    def smsTextRaw = json["smsText"] ?: queryParams?.getFirst("smsText")

    if (!issueIdRaw) {
        return Response.status(400).entity(JsonOutput.toJson([type: "error", body: "issueId الزامی است."])).build()
    }
    if (!phoneRaw) {
        return Response.status(400).entity(JsonOutput.toJson([type: "error", body: "شماره موبایل الزامی است."])).build()
    }

    Long issueId = issueIdRaw.toString().toLong()
    String phoneNumber = phoneRaw.toString().trim()

    // --- گرفتن آبجکت تسک ---
    IssueManager issueManager = ComponentAccessor.getIssueManager()
    def issue = issueManager.getIssueObject(issueId)
    if (!issue) {
        return Response.status(404).entity(JsonOutput.toJson([type: "error", body: "Issue یافت نشد."])).build()
    }

    // اگر متن پیام از سمت UI نیومده، متن پیش‌فرض بساز
    String title = issue.getSummary()
    if (title.length() > 20) {
        title = title.substring(0, 20) + " ..."
    }
    String smsText = smsTextRaw ?: """مشتری گرامی
در خصوص تسک ${issue.getKey()} با موضوع ${title} با شما تماس گرفته شد و پاسخگو نبودید.
لطفاً در اولین فرصت تسک خود را پیگیری نمایید.

خدمات مشتریان ورانگر
لغو 11"""

    // --- اطلاعات API پیامک ---
    String systemName = "YourCompanySystemName"   //put your company info here
    String username = "YourCompanyUserName"       //put your company info here
    String password = "YourCompanyPassword"       //put your company info here   
    String sender = "YourCountryPhoneNumber"      //put your company info here
    String recipient = phoneNumber

    def smsItems = [[
        sender: sender,
        recipient: recipient,
        body: smsText,
        customerId: "1"
    ]]

    // --- دریافت توکن ---
    String token = getToken(systemName, username, password)
    if (!token) {
        return Response.status(500).entity(JsonOutput.toJson([type: "error", body: "دریافت توکن پیامک ناموفق بود."])).build()
    }

    // --- درخواست ارسال پیام ---
    String smsUrl = "https://www.payamsms.com/panel/webservice/send"
    HttpURLConnection smsConnection = (HttpURLConnection) new URL(smsUrl).openConnection()
    smsConnection.setRequestMethod("POST")
    smsConnection.setRequestProperty("Content-Type", "application/json; charset=UTF-8")
    smsConnection.setRequestProperty("Authorization", "Bearer ${token}")
    smsConnection.setDoOutput(true)

    smsConnection.outputStream.withWriter("UTF-8") { writer ->
        writer << JsonOutput.toJson(smsItems)
    }

    int smsResponseCode = smsConnection.responseCode
    String smsResponse = (smsResponseCode == 200) ? smsConnection.inputStream.text : (smsConnection.errorStream?.text ?: "")
    smsConnection.disconnect()

    if (smsResponseCode != 200) {
        log.error("Failed to send SMS: ${smsResponseCode} - ${smsConnection.responseMessage}")
        log.error("Error response: ${smsResponse}")
        return Response.status(500).entity(JsonOutput.toJson([type: "error", body: "ارسال پیامک ناموفق بود."])).build()
    }

    // --- درج کامنت در تسک ---
  // --- درج کامنت در تسک ---
try {
    CommentManager commentMgr = ComponentAccessor.getCommentManager()
    ApplicationUser user = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser()
    
    // متن کامنت که در تسک ثبت می‌شود
    String msg = """📱 بدلیل عدم پاسخگویی مشتری، امکان پیگیری تسک ${issue.getKey()} وجود نداشت و به شماره ${phoneNumber} پیامک ارسال شد.

📝 متن پیام ارسالی:
${smsText}

🕐 زمان ارسال: ${new Date().format('yyyy-MM-dd HH:mm:ss')}
واحد پشتیبانی ورانگر"""
    
    commentMgr.create(issue, user, msg, true)
    log.warn("Comment added to issue")
} catch (Exception e) {
    log.error("Failed to add comment: ${e.message}")
}

    // --- خروجی موفقیت ---
    def result = [
        type : 'success',
        title: "ارسال پیامک",
        close: 'auto',
        body : "پیامک به شماره ${phoneNumber} ارسال شد."
    ]

    return Response.ok(JsonOutput.toJson(result)).type("application/json").build()
}
