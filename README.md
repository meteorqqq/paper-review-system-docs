# Scientific Paper Review System User Guide

## Introduction

This Scientific Paper Review System is an AI-powered tool designed to automate and enhance the scientific paper review process. The system integrates multiple AI models to analyze academic papers, generate comprehensive reviews, calculate paper scores, and provide intelligent search and question-answering capabilities through Retrieval-Augmented Generation (RAG).

## System Requirements

Before using the system, ensure the following prerequisites are met:

### Required Services
1. **Ollama**: Local service for running open-source large language models
2. **ScienceBeam**: Service for parsing PDF papers and converting them to XML format
3. **Streamlit**: Web application framework for the user interface

### Required Environments
1. **Conda environments**:
   - `ScienceBeam`: Environment for the ScienceBeam service
   - `llm`: Environment for the Streamlit application

### API Keys (if applicable)
- OpenAI API key (for GPT models)
- Google API key (for Gemini models)

## Getting Started

### Starting the System

The system uses a script called `start_services.zsh` to initialize all necessary services:

1. Open a terminal in the project directory
2. Run the start script:
   ```bash
   ./start_services.zsh
   ```

This script will:
- Check for all dependencies
- **Prompt for API keys configuration** (OpenAI and Google API keys)
- Start the Ollama service (port 11434)
- Start the ScienceBeam service (port 8080)
- Start the Streamlit application (port 8501)

### API Key Configuration

When you run the script for the first time, it will check for existing API keys and prompt you to enter them if none are found:

1. When prompted `"No API configuration found. Would you like to configure API keys now? (y/n)"`, enter `y` to configure API keys
2. Enter your OpenAI API key when prompted (or leave empty to skip)
3. Enter your Google API key when prompted (or leave empty to skip)

The keys will be stored securely in a `.api_keys` file in your project directory and automatically loaded on subsequent script runs.

To update your API keys later, you can either:
- Edit the `.api_keys` file directly
- Delete the `.api_keys` file and restart the services to be prompted again

### Accessing the Interface

Once all services are running, access the system through your web browser:
```
http://localhost:8501
```

## Using the System

### Interface Overview

The interface is organized into the following sections:

1. **Sidebar**: Model selection and configuration
2. **Main Panel**: File upload and processing
3. **Result Tabs**: Different views of the processed paper

### Step-by-Step Guide

#### 1. Select AI Model

In the sidebar, choose from the available models:
- **GPT-4**: OpenAI's GPT-4 model (requires API key)
- **Ollama**: Local Ollama model (default: llama3.1:latest)
- **Gemini**: Google's Gemini model (requires API key)

#### 2. Upload a Paper

1. Use the "Upload a scientific paper (PDF)" button to select your PDF file
2. Click "Process Paper" to start analysis

Processing includes:
- PDF to XML conversion (via ScienceBeam)
- Content extraction and structure analysis
- AI review generation
- RAG index creation
- Score calculation (using Qwen2.5 model)

#### 3. Review Results

After processing completes, review the results across five tabs:

##### Tab 1: Paper Summary
- Basic paper information
- Abstract and key points

##### Tab 2: AI-Generated Review
- Significance and novelty
- Potential reasons for acceptance
- Potential reasons for rejection
- Suggestions for improvement
- Key formulas and equations

##### Tab 3: Score Result
- Paper score (1-10 scale)
- Based on Qwen2.5 model evaluation
- Calculated from acceptance and rejection factors

##### Tab 4: RAG Search & Q&A
- **Search Function**: Search within the paper's content
  - Enter keywords or phrases
  - Set number of results to return
  - View matching passages with context

- **Q&A Function**: Ask questions about the paper
  - Enter your question
  - Receive answers based on paper content
  - View source passages for verification

##### Tab 5: Innovation Assessment
- Innovation scores across multiple dimensions
- Technical novelty evaluation
- Potential impact assessment
- Human expert feedback options
- Rating adjustment capabilities

## Advanced Features

### Model Information

The system utilizes multiple AI models:

1. **Review Generation Models**:
   - GPT-4 (OpenAI)
   - Llama3.1 (Ollama local deployment)
   - Gemini (Google)

2. **Scoring Model**:
   - Qwen2.5 (Alibaba, deployed via Ollama)

3. **Supporting Models**:
   - OpenAI Embeddings (for RAG vectorization)

### Human-AI Collaboration

The innovation assessment tab supports human-AI collaboration:
- AI generates initial assessments
- Human experts can adjust scores
- Feedback is recorded for model improvement
- Learning samples can be exported for model fine-tuning

### Customizing RAG Settings

The RAG system can be customized for specific needs:
- Text chunking parameters (chunk size, overlap)
- Vector retrieval parameters (number of results)
- Q&A generation parameters (temperature, context amount)

## Troubleshooting

### API Key Issues

If you encounter issues with API keys:
1. Check if the `.api_keys` file exists in your project directory
2. Verify the API keys are correctly formatted in the file
3. Ensure the API keys are valid and have not expired
4. Delete the `.api_keys` file and restart the services to reconfigure keys

### Service Startup Issues

If services fail to start:
1. Check log files in the `logs` directory
2. Ensure Ollama is properly installed and running
3. Verify ScienceBeam environment is correctly configured
4. Check if ports are already in use (8080, 8501, 11434)

### Model Not Found

If Ollama models are not found:
```bash
ollama pull llama3.1:latest
ollama pull qwen2.5:latest
```

### PDF Processing Errors

If PDF processing fails:
1. Ensure ScienceBeam service is running
2. Check if the PDF meets requirements (academic paper format)
3. Verify the PDF isn't password protected

## Shutting Down

To stop all services, press `Ctrl+C` in the terminal where `start_services.zsh` is running. The script will automatically clean up and stop all services.

## Technical Information

- System last updated: April 28, 2025
- For support, please contact the system administrator