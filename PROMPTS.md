
1st:

```

I am providing you with multiple documents. Your task is to perform the workflow described in the instructions below:

# Instructions for the LLM:

Your task is to act as an advanced document pre-processor. You will be given multiple documents and a set of instructions to follow. Your primary goal is to intelligently segment the document into semantically coherent chunks, each chunk in its own .md file, then create a toc.md with all the chunks listed and then create a chunk_summaries.md with concise, high-level summary of each chunk.

Process all input documents in ./input/
Save all chunks in ./out/chunks/
toc.md in ./out/
chunk_summaries.md in ./out/

Step 1: Context-aware Chunking

Analyze the Document's Structure:
- You must first analyze the input document to identify its inherent structure. Look for logical breaks such as headings, subheadings, new paragraphs, and bullet points.
- Prioritize these structural elements as chunk boundaries.
- If a structural boundary does not exist for a long section of text, perform semantic segmentation. Identify points where the main topic or idea shifts and use these as chunk boundaries.

Optimize Chunk Size and Overlap:
- Each chunk should be a manageable size, ideally between 1000 and 2000 tokens (approximately 1-2 pages of text).
- Avoid arbitrary splitting mid-sentence.
- Ensure there is a minimal overlap between chunks to maintain context without creating excessive redundancy. The overlap should be just enough to bridge ideas between segments.

Integrate Metadata:
- If any metadata is provided (e.g., authorship, date), you should use this information to inform your chunking decisions. For example, if a document contains meeting notes, you could create a chunk for each speaker's turn.

NB: Don't forget about the toc.md while creating the chunks.

Step 2: Chunk-Level Summarization

Process Each Chunk:
- Once you have created a logical list of chunks, you must process each chunk individually.
- Generate a concise summary for each chunk. The goal is to distill the core ideas, main arguments, or key takeaways.
- The summary should be objective and should not include excessive or minor details. It should be a high-level representation of the chunk's content.

```

2nd:

```

Using everything in ./out/ Do the following:

Logical Organization and Sorting:

"The goal is to not just combine text, but to organize it logically for enhanced readability and accessibility."

Semantic Grouping:
For large documents or multiple summaries, guide the LLM to group semantically related content. This can be achieved by instructing the LLM to identify related topics or themes, which can then be used to structure the Markdown output with appropriate headings.   

Hierarchical Structuring: Instruct the LLM to use Markdown's heading system (#, ##, ###) to delineate sections, sub-sections, and sub-sub-sections based on the content's inherent structure or semantic groupings.   

Maintain Coherence and Relevance: Throughout the process, emphasize maintaining relevance and coherence in prompts. Ensure each segment of the consolidated document contains logically connected information and that the LLM focuses on relevant sections without losing context.

This should be done in chunks and create the consolidated .md document in ./final/

Think Deep and recursively go through the files in ./out/

```

