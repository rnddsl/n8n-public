
This workflow creates a Retrieval-Augmented Generation (RAG) powered knowledge base using Google Drive for document source, Pinecone for vector storage, and Google Gemini for embeddings and chat functionalities. It enables end users to query a knowledge base with contextually relevant information sourced from your Google Drive documents.

### **Workflow Components**

1. **Google Drive Integration**:
    - Monitors a designated folder for new or updated files.
    - Downloads files for processing within n8n.
2. **Data Embedding and Storage**:
    - Utilizes Google Gemini to generate embeddings from document content.
    - Stores these embeddings in Pinecone to ensure consistent use for storage and retrieval operations.
3. **Chat Interface**:
    - Provides a chat trigger for users to pose questions.
    - Retrieves relevant document snippets from Pinecone based on queries.
    - Uses Google Gemini Chat to respond with detailed answers, including reference pages.
4. **Data Evaluation**:
    - Evaluation flow that evaluates the factual accuracy of responses, comparing them to expected outcomes using defined metrics.
### **Setup Instructions**

1. **Google Drive Connection**:
    - Open any Google Drive node and authenticate using your account credentials.
    - Specify the folder to monitor in nodes related to file search, creation, and updates.
2. **Pinecone Setup**:
    - Connect Pinecone by setting up your Pinecone API key in the Pinecone Vector Store nodes.
3. **Initial Run**:
    - Click the 'Execute Workflow' button to perform a one-time import of existing files from the folder.
    - This initializes the workflow by embedding all current documents. This only needs to be run once.
4. **Eval Workflow Setup**
	- Evaluate the quality of your RAG setup prior to publishing, after making changes to the workflow, and to assess model drift.
	1. **Dataset Preparation**:
	    - Organize your dataset with `input` and `expected` columns.
	    - `input` holds the queries, while `expected`contains the ground truth responses.
	2. **Evaluation Nodes**:
	    - ==Use nodes like `Evaluation`, `Evaluation1`, `Evaluation2`, and `Eval Check` for assessing response accuracy.==
	    - These nodes compare system-generated answers against the `expected` responses and score them from 1 to 5 based on predefined criteria.
	3. **Scoring Criteria**:
	    - Current criteria focus on factual accuracy and semantic similarity, allowing for variation in language but with correct content and context. You can modify as needed.
	4. **Integration**:
	    - Upload and link the dataset to your workflow.
	    - Configure nodes for evaluating and comparing each row of data.
	5. **Execution**:
	    - The workflow processes each dataset entry, scoring the system's responses against expected answers. Go to the Evaluations tab to kick off an evaluation run.
	    - Use these scores to refine and ensure the accuracy of your RAG system.

5. **Regular (Automatic) Execution**:
	- **Continuous Monitoring**:
	    - Start the workflow to automatically monitor the specified folder for new or updated documents without further manual intervention.
	- **User Interaction via Chat**:
	    - Once published, the workflow's chat functionality can be embedded into your application or website.
	    - To make the chat interface accessible:
	        - Navigate to the “Publish” section in your workflow settings.
	        - Enable public access options and configure permissions to allow user interactions.
	        - Embed the provided chat widget code into your desired platform to enable users to interact directly with the chat interface.
	- **Engagement with Knowledge Base**:
	    - Users can pose questions using the chat interface, and the workflow will process these queries by fetching and providing contextually relevant responses based on the documents stored in your Google Drive.

	This setup allows seamless user interaction with the knowledge base, offering real-time retrieval and response capabilities using the embedded chat interface.