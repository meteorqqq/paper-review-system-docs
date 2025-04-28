# Scientific Paper Review System - User Guide

## Introduction

The Scientific Paper Review System is an advanced AI-powered tool designed to automate and enhance the academic paper review process. This system leverages multiple large language models to analyze scientific papers, generate comprehensive reviews, calculate scores, and provide intelligent search and question-answering capabilities through Retrieval-Augmented Generation (RAG).

## System Overview

This application provides the following key features:

1. **PDF Paper Processing**: Upload and process scientific papers in PDF format
2. **AI-Generated Reviews**: Generate detailed reviews using various AI models (GPT-4, Llama3.1, Gemini)
3. **Paper Scoring**: Calculate paper scores using the Qwen2.5 model
4. **RAG Search & Q&A**: Search within the paper content and ask questions about the paper
5. **Innovation Assessment**: Evaluate the paper's innovation with human-AI collaboration

## Getting Started

### Prerequisites

Before using the system, ensure you have the following:

1. **Required Services**:
   - Ollama: For running open-source large language models
   - ScienceBeam: For parsing PDF papers into XML format
   - API Keys: OpenAI API key (for GPT models) and Google API key (for Gemini models)

2. **Start Services Script**:
   - Use the provided `start_services.zsh` script to initialize all necessary services

### Starting the System

1. Open a terminal in the project directory
2. Run the start script:
   ```bash
   ./start_services.zsh
   ```
3. The script will:
   - Prompt for API keys configuration (OpenAI and Google)
   - Start the Ollama service (port 11434)
   - Start the ScienceBeam service (port 8080)
   - Start the Streamlit application (port 8501)

4. Access the application in your web browser at:
   ```
   http://localhost:8501
   ```

## Using the Application

### Interface Overview

The interface consists of:

1. **Sidebar**: For model selection and configuration
2. **Main Panel**: For file upload and processing
3. **Result Tabs**: For viewing different aspects of the paper analysis

### Model Selection

In the sidebar, select your preferred AI model:

- **GPT-4**: OpenAI's most advanced model (requires API key)
- **Ollama**: Local open-source model (default: llama3.1)
- **Gemini**: Google's advanced model (requires API key)

### Processing a Paper

1. Use the "Upload a scientific paper (PDF)" button to select your PDF file
2. Click "Process Paper" to start the analysis

The system will:
- Convert the PDF to XML using ScienceBeam
- Extract key paper components (abstract, sections, figures, etc.)
- Generate an AI review
- Create a searchable index for RAG
- Calculate a paper score

### Viewing Results

After processing completes, navigate through the five tabs:

#### 1. Paper Summary
- Displays the paper's abstract and key information

#### 2. AI-Generated Review
The AI review includes:
- Significance and novelty assessment
- Potential reasons for acceptance
- Potential reasons for rejection 
- Suggestions for improvement
- Important formulas (with LaTeX rendering)

#### 3. Score Result
- Displays the paper score (1-10 scale) calculated by Qwen2.5
- Shows detailed reasoning for the score
- Based on the acceptance and rejection factors from the review

#### 4. RAG Search & Q&A
This tab enables two key functionalities:

**Search Function**:
- Enter keywords or phrases in the search box
- Adjust the number of results using the slider
- Click "Search" to find relevant passages in the paper

**Q&A Function**:
- Enter questions about the paper in the text area
- Click "Get Answer" to receive responses based on the paper content
- View answers with context from relevant paper sections

#### 5. Innovation Assessment
This tab facilitates human-AI collaboration in evaluating innovation:

- Click "Generate AI Innovation Assessment" to start the process
- View AI-generated scores across multiple dimensions:
  - Technical Novelty
  - Conceptual Originality
  - Potential Impact
  - Methodological Innovation
  - Application Innovation
  - Solution Innovation

- For each dimension:
  - Review the AI's explanation
  - Adjust scores if you disagree
  - Provide reasons for your adjustments
  - Save your feedback to improve the model

- Additional features:
  - View historical feedback data
  - Export learning samples for model fine-tuning

## Advanced Features

### RAG System

The RAG (Retrieval-Augmented Generation) system:
- Creates vector embeddings of paper content
- Divides content into searchable chunks
- Enables semantic search rather than just keyword matching
- Provides context-aware answers to questions

### Human-AI Collaboration

The innovation assessment feature:
- Combines AI analysis with human expert judgment
- Records score adjustments and reasoning
- Builds a dataset of expert feedback
- Generates learning samples for future model improvement

## Troubleshooting

### API Key Issues
- Ensure OpenAI and Google API keys are correctly configured
- Check the `.api_keys` file in your project directory
- Delete this file and restart services to reconfigure keys

### Service Issues
- Check if ScienceBeam service is running (port 8080)
- Verify Ollama is installed and running (port 11434)
- Ensure required models are downloaded in Ollama

### Model Availability
If Ollama models are not found, download them:
```bash
ollama pull llama3.1:latest
ollama pull qwen2.5:latest
```

### PDF Processing Errors
- Ensure the PDF is not password-protected
- Check that the PDF contains standard academic paper formatting
- Verify ScienceBeam service is running properly

## Shutting Down

To stop all services:
- Press `Ctrl+C` in the terminal running the start script
- The script will automatically clean up resources and stop all services

## Technical Details

### Model Information

The system uses multiple models:

1. **Review Generation**:
   - GPT-4 (OpenAI)
   - Llama3.1 (via Ollama)
   - Gemini (Google)

2. **Paper Scoring**:
   - Qwen2.5 (via Ollama)

3. **Vector Embeddings**:
   - OpenAI Embeddings (for RAG)

### Directory Structure

The system maintains several directories:
- `cache/`: Temporary storage for processed files
- `logs/`: Log files from various services
- `data/`: Training data and configuration files

For additional technical information or assistance, please refer to the environment setup guide or contact the system administrator.