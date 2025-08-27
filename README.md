Retrieval-Augmented Generation Tutor Project

1. Overview 

This is a bit embarrassing, but I struggled with the Cryptography I course (https://www.coursera.org/learn/crypto) this year. I came up with an idea: using Large Launguage Model, I might build a personal crypto tutor just for myself.  I asked ChatGPT about its feasibility, and this report summarizes the planning, understanding, and trial-and-error process.

Using RAG (Retrieval-Augmented Generation) and Hugging Face, the system is designed to answer short questions.


2. System Structure

Two Spaces were prepared on Hugging Face: preprocess and tutor-demo. The preprocess Space generates and embeds chunks from the PDF, while the tutor-demo Space loads the dataset and answers questions.


3-a. Workflow　- Google Colab version

- Created text chunks from the book PDF using Google Colab.
- Uploaded the processed data to a Hugging Face Dataset.
- Started the tutor-demo Space and asked questions.
- Checked whether the answers were correct.
- Depending on the quality of the answers, adjusted parameters or prompts in tutor-demo.

In this experiment, I generated chunks and embeddings directly in Google Colab and uploaded them to a Hugging Face Dataset. 
The preprocess Space was not strictly necessary for this workflow, though it could be useful in the future as a more automated way to upload new chapters or additional materials.


3-b. Workflow - Hugging Face version
TBD


4. Failures and Fixes


5. Lessons Learned

- The quality of chunks, the adjustment of TOP_K, and the choice of the model are key factors.
- The quality of answers can also be influenced by the prompt.
- Although this system is not yet accurate enough to tutor me in basic cryptography, it was still a meaningful experiment.
- It was a valuable challenge for me, especially since I had no background in RAG beforehand and worked on it with the help of ChatGPT.


6. Next Steps

When I study AI/Machine Learning in the future, I would like to try stronger models.
