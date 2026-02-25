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
Payload Used: [Insert payload here]

Result: [Describe if it succeeded or failed]

Screenshot:

Explanation of why it failed/worked: [Explain the stricter defenses implemented at this level and why your payload behaved the way it did]

Security Level Comparison
[Briefly summarize how the application's defensive behavior changed across the three levels for this specific vulnerability.]
