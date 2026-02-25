# Application-Security-Testing-DVWA

# DVWA Security Lab Report

Vulnerability Testing
1. Brute Force
Security Level: Low
Payload Used: Burp Suite Intruder Attack

Result: Multiple attempts were made to log in based on the password list I had provided, a few lengths were different indicating they were correct password matches.

Screenshot:
<img width="1818" height="857" alt="Attack Result" src="https://github.com/user-attachments/assets/6a43b826-facc-42ce-bd2c-cb0cb6f72cb8" />


Explanation of why it worked: There is no limit to the number of log in attempts I can make, hence I can simply brute force every common password possible. Especially since the passwords are not complicated.

Security Level: Medium
Payload Used: Burp Suite Intruder Attack

Result: Multiple attempts were made to log in again based on the same password list, once more the correct password was found.

Screenshot:<img width="1810" height="811" alt="Brute Med" src="https://github.com/user-attachments/assets/81aec2a7-f268-42be-9822-6ad1a4f70fb9" />


Explanation of why it worked: Because the medium security only has a 2 second gap between attempts, brute force is still possible as there is only a minor delay between password attempts.

Security Level: High
Payload Used: Burp Suite Intruder Attack

Result: Multiple attempts were made to log in, this time with a token being generated and Recursive GERP being utilised. Once more, the correct password and token combination was found.

Screenshot:<img width="1803" height="752" alt="Brute High" src="https://github.com/user-attachments/assets/80df1b5d-3f4e-4dfa-a29f-6393c317cc4a" />


Explanation of why it failed/worked: There is a CSRF token being used on this level, where on every page refresh a new token is generated. In order to do this, I had to not only brute force the passwords but extract the token for each refresh as well. Successfully doing this allowed me to figure out the password.

Security Level Comparison
At Low security, there were no protection measures so it was a trivial case of trying multiple passwords. At medium difficulty there was only a short delay but it was still easy. At high difficulty it was more complicated where I had to extract the token as well to find out the password.
