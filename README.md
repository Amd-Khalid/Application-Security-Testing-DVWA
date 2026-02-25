# Application-Security-Testing-DVWA

# DVWA Security Lab Report

Vulnerability Testing
1. Brute Force
Security Level: Low
Payload Used: Burp Suite Intruder Attack

Result: Multiple attempts were made to log in based on the password list I had provided, a few lengths were differnent indicating they were correct password matches.

Screenshot:
<img width="1818" height="857" alt="Attack Result" src="https://github.com/user-attachments/assets/6a43b826-facc-42ce-bd2c-cb0cb6f72cb8" />


Explanation of why it worked: There is no limit to the number of log in attempts I can make, hence I can simply brute force every common password possible. Especially since the passwords are not complicated.

Security Level: Medium
Payload Used: [Insert payload here]

Result: [Describe the result]

Screenshot:

Explanation of why it worked: [Explain how you bypassed the medium-level defenses]

Security Level: High
Payload Used: [Insert payload here]

Result: [Describe if it succeeded or failed]

Screenshot:

Explanation of why it failed/worked: [Explain the stricter defenses implemented at this level and why your payload behaved the way it did]

Security Level Comparison
[Briefly summarize how the application's defensive behavior changed across the three levels for this specific vulnerability.]
