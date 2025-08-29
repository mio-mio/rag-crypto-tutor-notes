<B>Retrieval-Augmented Generation Tutor Demo</b>

1. Overview 

This is a bit embarrassing, but I struggled with the Cryptography I course (https://www.coursera.org/learn/crypto) this year. I came up with an idea: using Large Launguage Model, I might build a personal crypto tutor just for myself.  I asked ChatGPT about its feasibility, and this report summarizes the planning, understanding, and trial-and-error process.

Using RAG (Retrieval-Augmented Generation) and Hugging Face, the system is designed to answer short questions.


2-a. Environment

- **Embedding model**: `sentence-transformers/all-MiniLM-L6-v2`  
  A lightweight sentence transformer (~22M parameters, 384-dimensional embeddings).  
  Used to convert both the user query and text chunks into vectors for similarity search.

- **Generation model**: `TinyLlama/TinyLlama-1.1B-Chat-v1.0`  
  A compact instruction-tuned language model (~1.1B parameters).  
  Used to generate natural language answers based on the retrieved chunks and the user query.

- **Execution environment**:  
  - Preprocessing: Google Colab (for text extraction, chunking, and embeddings)  
  - Hosting: Hugging Face Spaces (CPU basic, 2 vCPU / 16 GB RAM)  
    - `preprocess` Space for generating and uploading artifacts  
    - `tutor-demo` Space for interactive Q&A with RAG



2-b. System Structure

Two Spaces were prepared on Hugging Face: preprocess and tutor-demo. The preprocess Space generates and embeds chunks from the PDF, while the tutor-demo Space loads the dataset and answers questions.

Note: Although a preprocess Space was prepared, in practice I found it more stable to run the preprocessing in Google Colab and then upload the results to the Hugging Face Dataset. This made the demo Space lighter and more reliable for testing.


3-a. Workflow　- Google Colab version

- Created text chunks from the book PDF using Google Colab.
- Uploaded the processed data to a Hugging Face Dataset.
- Started the tutor-demo Space and asked questions.
- Checked whether the answers were correct.
- Depending on the quality of the answers, adjusted parameters or prompts in tutor-demo.


3-b. Workflow - Hugging Face version

- Opened the preprocess Space on Hugging Face.
- Uploaded the book PDF through the UI.
- Waited for the Space to generate chunks and embeddings automatically.
- Confirmed that the artifacts were saved to the Hugging Face Dataset.
- Started the tutor-demo Space and asked questions using the updated dataset.
- Checked whether the answers were correct.
- Depending on the quality of the answers, adjusted parameters or prompts in tutor-demo.


3-c. System Internals

- The preprocess Space extracts text from PDF pages, splits them into chunks, and generates vector embeddings using a sentence-transformer model.
- These artifacts are stored in a Hugging Face Dataset (`chunks.jsonl` and `embeddings.npz`).  **When using Google Colab, upoload these two files directly to Hugging Face Dataset.*
- The tutor-demo Space downloads the dataset, runs similarity search over the embeddings, and retrieves the top-k relevant chunks.
- The retrieved chunks, together with the user query, are passed to the language model to generate an answer.


4. Failures and Fixes

401 Unauthorized error occurred in Hugging Face Space.
- Fixed by adjusting the token name settings.

Loading took more than ten minutes.
- In practice, responses that take longer than five minutes are considered too slow and typically warrant a restart (this is a practical rule of thumb). I mitigated this by making the prompt more concise.

Answers only repeated the question again and again.
- Adjusted the value of TOP_K from 2 to 4.

Wrong information appeared in the answers.
- Changed the prompt and the value of TOP_K, but it was not effective. It seems that modifying the retrieved chunks and using a stronger model are required.


5. Lessons Learned
From these failures I learned,
- The quality of chunks, the adjustment of TOP_K, and the choice of the model are key factors. In addition, controlling   max_new_tokens and setting a practical response timeout are also important for balancing accuracy and usability.
- The quality of answers can also be influenced by the prompt.
- In several answers the model started to generate **its own new questions and answers**. This behavior is caused by the model’s training style (Q&A format) and insufficient output control in the prompt.  In practice, stricter prompt engineering or output post-processing would be required to avoid this.
- Although this system is not yet accurate enough to tutor me in basic cryptography, it was still a meaningful experiment.
- It was a valuable challenge for me, especially since I had <b>no background in RAG beforehand</b> and worked on it with the help of ChatGPT.


6. Next Steps

When I study AI/Machine Learning in the future, I would like to try stronger models.
