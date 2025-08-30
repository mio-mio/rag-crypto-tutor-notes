
### Learning Notes on Code ###

Below are selected code excerpts from the tutor-demo project.  
The code itself was generated with the help of ChatGPT, but I added my own comments to highlight what I understood during the process.

## 1. load_artifacts() ##
```
def load_artifacts():
    root = snapshot_download(repo_id=DATASET_REPO, repo_type="dataset",
                             local_dir=ARTIFACT_LOCAL, token=TOKEN)
    # 配置ゆらぎを吸収
    cand_chunks = [os.path.join(root, "artifacts", "chunks.jsonl"),
                   os.path.join(root, "chunks.jsonl"),
                   os.path.join(ARTIFACT_LOCAL, "artifacts", "chunks.jsonl")]
    cand_embs   = [os.path.join(root, "artifacts", "embeddings.npz"),
                   os.path.join(root, "embeddings.npz"),
                   os.path.join(ARTIFACT_LOCAL, "artifacts", "embeddings.npz")]

    chunks_path = next((p for p in cand_chunks if os.path.exists(p)), None)
    embs_path   = next((p for p in cand_embs   if os.path.exists(p)), None)
    if not (chunks_path and embs_path):
        raise FileNotFoundError("chunks.jsonl / embeddings.npz are not found.Please check the location in the Datasets.")

    chunks = [json.loads(l) for l in open(chunks_path, encoding="utf-8")]
    embs   = np.load(embs_path)["embeddings"].astype("float32")
    # 正規化
    norms = np.linalg.norm(embs, axis=1, keepdims=True) + 1e-9
    embs  = embs / norms
    return embs, chunks
```

Steps :
1. Download artifacts (embeddings + chunks) from the Hugging Face dataset.*1
2. Load embeddings into NumPy and chunks into JSON.*2
3. Return them for use in retrieval.

*1 `snapshot_download`is the function only in Hugging Face.
*2 `chunks.jsonl` and `embedings.npz` must have the same amout of lines.

Each text chunk in `chunks.jsonl` must correspond to exactly one embedding vector in `embeddings.npz`.  
- `chunks.jsonl` contains the split pieces of text (one JSON line per chunk).  
- `embeddings.npz` contains the numerical vector representation for each chunk.  

During retrieval, the system looks up the top-k embedding indices and then fetches the corresponding text chunks by line number.  
Therefore, the number of entries in both files must always match (1-to-1 mapping).  
If they are misaligned, retrieval will either fail (index out of range) or return incorrect results.


## 2. search() ##

```
def search(query: str, k=TOP_K):
    q = embed_query(query)
    sims = cosine_similarity(q, EMBS)[0]
    idx = np.argsort(-sims)[:k]
    results = []
    for i in idx:
        c = CHUNKS[i]
        results.append({
            "score": float(sims[i]),
            "chapter": c.get("chapter",""),
            "page_start": c.get("page_start",""),
            "page_end": c.get("page_end",""),
            "text": (c.get("text","") or "")[:1000]
        })
    return results
```
This function is for searching purpose. 

Steps :
- Embed the query with the same model as the corpus (e.g., all-MiniLM-L6-v2).
- Compute cosine similarity between the query vector and all stored embeddings.
- Select the top-k hits and return them along with chapter, page range, and similarity score.


## 3. generate_answer() ##

 ```
 def generate_answer(query: str, contexts):
    context_strs = []
    for c in contexts:
        cite = f"[{c['chapter']} p.{c['page_start']}-{c['page_end']}]"
        context_strs.append(f"{cite}\n{c['text']}")
    context_block = "\n\n---\n\n".join(context_strs)
    if len(context_block) > MAX_INPUT_CHARS:
        context_block = context_block[:MAX_INPUT_CHARS]

    prompt = (
         "You are a helpful cryptography tutor.\n"
        "You must answer ONLY using the CONTEXT below.\n"
        "If the CONTEXT lacks required facts, say so explicitly and ask for a more specific section/page.\n"
        "Never invent steps or use outside knowledge.\n"
        "Answer the user question only, do not generate new questions."
        "Cite the page ranges from the CONTEXT in your answer.\n\n"
        f"CONTEXT:\n{context_block}\n\n"
        f"QUESTION:\n{query}\n\n"
        "Answer:"
    )

    inputs = tok(prompt, return_tensors="pt").to(device)
    streamer = TextIteratorStreamer(tok, skip_prompt=True, skip_special_tokens=True)
    gen_kwargs = dict(
        **inputs,
        max_new_tokens=64,
        temperature=0.2,
        top_p=0.95
    )
    thread = threading.Thread(target=model.generate, kwargs={**gen_kwargs, "streamer": streamer})
    thread.start()
    out_text = ""
    for new_text in streamer:
        out_text += new_text
    return out_text
```
Builds the prompt and generates the model's answer.

Steps :
1. Collect top-k context chunks, with chapter and page info for citation.
2. Join them into a context block, truncated if too long (MAX_INPUT_CHARS).
3. Build a strict prompt.
4. Tokenize the prompt and send it to the model (TinyLlama).
5. Stream the generated answer token-by-token.
6. Return the full generated text.

kwargs stands for "keyword arguments". 


## 4. ask() ##
```
def ask(query):
    try:
        hits = search(query, k=TOP_K)
        answer = generate_answer(query, hits)
        cites = "\n".join([f"- {h['chapter']} p.{h['page_start']}-{h['page_end']} (score={h['score']:.3f})" for h in hits])
        return answer, json.dumps(hits, ensure_ascii=False, indent=2), cites
    except Exception:
        tb = traceback.format_exc()
        return f"[ERROR]\n{tb}", "[]", ""
```
Steps :
1. Receive the question from the user.
2. Retrieve top-k relevant text chunks (search).
3. Generate an answer using the chunks as context.
4. Return the answer together with references (citations).
