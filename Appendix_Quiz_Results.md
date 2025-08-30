### Appendix: Quiz Results

## Summary ##
As a result of adjusting parameters and settings, I was able to get 3 correct answers out of 5 mock questions. Here are the screenshots and my observations.

For easy quizzes that can be answered with Yes/No or an acronym, correct answers were produced. However, when the difficulty increased to “explain this” type of questions, additional irrelevant information was often mixed in and the answers became incorrect.

As written in the README.md, the model also started to generate its own new questions and answers. This behavior seems to come from the model’s training style (Q&A format) and insufficient output control in the prompt. In practice, stricter prompt engineering or output post-processing would be required to avoid this.



## 1. What does the acronym "RSA" stand for?

![Quiz 1 Screenshot](images/1.png)
Ideal answer: Rivest, Shamir, and Adleman
-> Correct!

## 2. What does "AES" stand for?

![Quiz 2 Screenshot](images/2.png)
Ideal answer: Advanced Encryption Standard
-> Correct!

## 3. In symmetric cryptography, do the sender and receiver share the same key?
![Quiz 3 Screenshot](images/3.png)
Ideal answer: Yes
-> Correct!

## 4. What is the main security assumption of RSA?
![Quiz 4 Screenshot](images/4.png)
Ideal answer: The difficulty of factoring large integers

## 5. What mathematical operation is Diffie-Hellman key exchange based on?
![Quiz 5 Screenshot](images/5.png)
Ideal answer: Modular exponentiation over a finite group
