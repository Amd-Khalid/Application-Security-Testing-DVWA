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

Payload Used: External HTML link. Referer: http://127.0.0.1.hacker.com/evil.html

Result: Password for the website was changed successfully after the new url was used.

Screenshot:
<img width="1471" height="377" alt="CSRF Medium level" src="https://github.com/user-attachments/assets/0415dc62-c4ed-48fb-bcb7-c80958a6386b" />






Explanation of why it worked: The security check is broken, the developer isn't checking if the referer being used for the URL exactly matches, they are simply checking if 127.0.0.1 is anywhere in the line. Hence as long as the URL for the evil.html file contains 127.0.0.1 anywhere, it still goes through and the same exploit for changing password works.
