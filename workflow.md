I am working on a workflow for

"Consolidating Large Document Collections into Comprehensive Markdown using Advanced LLMs"

First major step is "Document Pre-processing".

Please help me create a prompt and instructions for an llm when I give it documents, it should create new output documents doing the following:

# Advanced Document Pre-processing (Chunking):

"Even with LLMs boasting massive context windows, intelligent chunking remains a critical step. It helps manage information overload, improves recall, and ensures semantic coherence, directly impacting the quality of your consolidated output.

Understand the Importance of Chunking: Chunking breaks large texts into smaller, manageable segments. This is essential for:

- Managing token limits for documents that might still exceed even the largest context windows.
- Improving contextual understanding by providing focused information.
- Enhancing retrieval accuracy and mitigating hallucinations by preventing the model from being overwhelmed.
- Identifying semantically similar sections for effective summarization and synthesis.

Chunking Strategy:
Context-aware Chunking (Recommended): This method considers the semantic meaning or logical flow of the text.

Semantic Segmentation:
Divide text based on topic shifts or changes in meaning, ensuring each chunk is a coherent unit. This improves relevance and reduces noise.

Structural Segmentation:
Utilize the document's inherent structure (headings, paragraphs, bullet points) to define boundaries, maintaining logical flow.

Recursive Text Splitter:
Ideal for complex documents like PDFs, allowing hierarchical splitting (e.g., by chapters, then paragraphs) and managing overlaps to preserve context.

Optimize Chunk Size and Overlap:

- Dynamic Adjustment:
  Adjust chunk size based on text complexity and expected queries. More complex content may benefit from smaller, more focused chunks.

- Minimal Overlap:
  While overlaps can preserve context across boundaries, excessive overlap leads to redundancy and increased computational costs. Aim for minimal overlap. A token-based chunking approach with a specified character size and minimal overlap is practical.

Metadata Integration: Use metadata (timestamps, authorship, document type) to guide chunking and group related content logically."

AND:

# Iterative Summarization and Consolidation Workflow:

"For large document sets, a multi-pass summarization workflow is essential to progressively condense and abstract information, ensuring a comprehensive yet concise final output.

Multi-Pass Summarization (Map-Reduce Approach):
This approach is particularly robust for larger sets of documents, especially when the total text length exceeds the LLM's context window.

Chunk-Level Summaries ("Map" Step):

- Divide original documents into manageable chunks (e.g., 1-2 pages or 1000-2000 tokens per chunk).
- Use an LLM to generate a concise summary for each individual chunk. Instruct the LLM to focus on core ideas without excessive detail."

==

The Prompt:

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
