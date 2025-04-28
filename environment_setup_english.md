# Environment Setup Guide for Scientific Paper Review System

This guide provides detailed instructions for setting up the environment required to run the Scientific Paper Review System. The system uses multiple AI models for paper review, RAG-based search, and innovation assessment.

## System Requirements

- **Operating System**: Linux (Ubuntu/Debian recommended)
- **Python**: Version 3.9+ recommended
- **Conda**: For managing virtual environments
- **GPU**: Recommended for faster processing (but not required)

## Setup Script

We've provided a simple script that automates the entire setup process. Follow these steps:

1. Save the script below as `setup_environment.sh` in your project directory
2. Make it executable: `chmod +x setup_environment.sh`
3. Run the script: `./setup_environment.sh`

```bash
#!/bin/bash

# setup_environment.sh - Environment setup script for Scientific Paper Review System
# Created: April 28, 2025

set -e  # Exit on error

# ANSI color codes for output formatting
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m' # No color

# Print colored message
print_message() {
    local color=$1
    local message=$2
    echo -e "${color}${message}${NC}"
}

# Check if command exists
check_command() {
    if ! command -v $1 &> /dev/null; then
        print_message $RED "Error: Command $1 not found. Please install it first."
        return 1
    fi
    return 0
}

# Create project directory structure
create_directory_structure() {
    print_message $YELLOW "Creating project directory structure..."
    
    # Create main directories
    mkdir -p cache logs data finetune plots
    
    print_message $GREEN "Directory structure created successfully!"
}

# Install Conda if not installed
install_conda() {
    if ! check_command conda; then
        print_message $YELLOW "Conda not found. Installing Miniconda..."
        
        # Download Miniconda installer
        wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh
        
        # Install Miniconda
        bash miniconda.sh -b -p $HOME/miniconda
        
        # Add conda to path
        export PATH="$HOME/miniconda/bin:$PATH"
        
        # Initialize conda
        eval "$($HOME/miniconda/bin/conda shell.bash hook)"
        
        # Add to bashrc
        echo 'export PATH="$HOME/miniconda/bin:$PATH"' >> ~/.bashrc
        echo 'eval "$(conda shell.bash hook)"' >> ~/.bashrc
        
        print_message $GREEN "Miniconda installed successfully!"
    else
        print_message $GREEN "Conda is already installed."
    fi
}

# Install Ollama if not installed
install_ollama() {
    if ! check_command ollama; then
        print_message $YELLOW "Ollama not found. Installing Ollama..."
        
        # Download and run Ollama installer
        curl -fsSL https://ollama.com/install.sh | sh
        
        print_message $GREEN "Ollama installed successfully!"
    else
        print_message $GREEN "Ollama is already installed."
    fi
}

# Create ScienceBeam environment
create_sciencebeam_env() {
    print_message $YELLOW "Creating ScienceBeam conda environment..."
    
    # Check if environment already exists
    if conda info --envs | grep -q "ScienceBeam"; then
        print_message $YELLOW "ScienceBeam environment already exists. Updating..."
        conda activate ScienceBeam
    else
        # Create new environment
        conda create -n ScienceBeam python=3.9 -y
        conda activate ScienceBeam
    fi
    
    # Install packages
    pip install sciencebeam-parser tornado lxml
    
    # Verify installation
    python -c "import sciencebeam_parser; print(f'ScienceBeam installed: {sciencebeam_parser.__version__}')" || \
        print_message $RED "ScienceBeam installation verification failed."
    
    conda deactivate
    print_message $GREEN "ScienceBeam environment created successfully!"
}

# Create LLM environment
create_llm_env() {
    print_message $YELLOW "Creating LLM conda environment..."
    
    # Check if environment already exists
    if conda info --envs | grep -q "llm"; then
        print_message $YELLOW "LLM environment already exists. Updating..."
        conda activate llm
    else
        # Create new environment
        conda create -n llm python=3.10 -y
        conda activate llm
    fi
    
    # Install packages
    pip install streamlit langchain langchain-community langchain-openai tiktoken pikepdf faiss-cpu pandas numpy
    pip install openai google-generativeai requests matplotlib jupyter ipykernel
    
    # Install additional packages for RAG
    pip install sentence-transformers chromadb pydantic
    
    conda deactivate
    print_message $GREEN "LLM environment created successfully!"
}

# Download Ollama models
download_ollama_models() {
    print_message $YELLOW "Downloading Ollama models..."
    
    # Start Ollama service if not running
    if ! pgrep -x "ollama" > /dev/null; then
        print_message $YELLOW "Starting Ollama service..."
        ollama serve &
        OLLAMA_PID=$!
        sleep 5  # Wait for service to start
    fi
    
    # Download models
    print_message $YELLOW "Pulling Llama3.1 model... (this may take a while)"
    ollama pull llama3.1:latest
    
    print_message $YELLOW "Pulling Qwen2.5 model... (this may take a while)"
    ollama pull qwen2.5:latest
    
    # Kill Ollama if we started it
    if [[ -n $OLLAMA_PID ]]; then
        kill $OLLAMA_PID
    fi
    
    print_message $GREEN "Ollama models downloaded successfully!"
}

# Configure API keys
configure_api_keys() {
    print_message $YELLOW "Configuring API keys..."
    
    # Prompt for OpenAI API key
    read -p "Enter your OpenAI API key (leave empty to skip): " OPENAI_API_KEY
    
    # Prompt for Google API key
    read -p "Enter your Google API key (leave empty to skip): " GOOGLE_API_KEY
    
    # Save API keys to file
    if [[ -n $OPENAI_API_KEY || -n $GOOGLE_API_KEY ]]; then
        echo "# API Keys for Scientific Paper Review System" > .api_keys
        
        if [[ -n $OPENAI_API_KEY ]]; then
            echo "OPENAI_API_KEY=\"$OPENAI_API_KEY\"" >> .api_keys
        fi
        
        if [[ -n $GOOGLE_API_KEY ]]; then
            echo "GOOGLE_API_KEY=\"$GOOGLE_API_KEY\"" >> .api_keys
        fi
        
        chmod 600 .api_keys
        print_message $GREEN "API keys saved to .api_keys file."
    else
        print_message $YELLOW "No API keys provided. Some features may not work."
    fi
}

# Create start_services script if it doesn't exist
create_start_script() {
    if [[ ! -f "start_services.zsh" ]]; then
        print_message $YELLOW "Creating start_services.zsh script..."
        
        cat > start_services.zsh << 'EOF'
#!/usr/bin/env zsh

# Script to start ScienceBeam, Streamlit app, and Ollama service
# Created: April 28, 2025

# Set color output
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m' # No color

# Environment variables
SCIENCEBEAM_CONDA_ENV="ScienceBeam"
STREAMLIT_CONDA_ENV="llm"
STREAMLIT_APP_PATH="$(pwd)/streamlit_rag.py"
OLLAMA_PORT="11434"

# API Keys configuration
OPENAI_API_KEY=""
GOOGLE_API_KEY=""
API_CONFIG_FILE=".api_keys"

# Set log files
LOG_DIR="./logs"
mkdir -p $LOG_DIR
SCIENCEBEAM_LOG="$LOG_DIR/sciencebeam_$(date +%Y%m%d_%H%M%S).log"
STREAMLIT_LOG="$LOG_DIR/streamlit_$(date +%Y%m%d_%H%M%S).log"
OLLAMA_LOG="$LOG_DIR/ollama_$(date +%Y%m%d_%H%M%S).log"

# Print colored message
print_message() {
    local color=$1
    local message=$2
    echo "${color}${message}${NC}"
}

# Check if command exists
check_command() {
    if ! command -v $1 &> /dev/null; then
        print_message $RED "Error: Command $1 not found. Please make sure it is installed."
        return 1
    fi
    return 0
}

# Check if port is in use
check_port() {
    local port=$1
    if lsof -Pi :$port -sTCP:LISTEN -t &> /dev/null ; then
        print_message $YELLOW "Warning: Port $port is already in use."
        return 1
    fi
    return 0
}

# Check dependencies
check_dependencies() {
    print_message $YELLOW "Checking dependencies..."
    
    # Check if conda is installed
    if ! check_command "conda"; then
        print_message $RED "Error: conda is not installed. Please make sure conda is installed."
        exit 1
    fi
    
    # Check if conda environments exist
    if ! conda env list | grep -q "$SCIENCEBEAM_CONDA_ENV"; then
        print_message $RED "Error: conda environment '$SCIENCEBEAM_CONDA_ENV' does not exist."
        print_message $YELLOW "Please make sure the sciencebeam environment is created."
        exit 1
    fi
    
    # Check if Streamlit conda environment exists
    if ! conda env list | grep -q "$STREAMLIT_CONDA_ENV"; then
        print_message $RED "Error: conda environment '$STREAMLIT_CONDA_ENV' does not exist."
        print_message $YELLOW "Please make sure the llm environment is created."
        exit 1
    fi
    
    # Check if Ollama is installed
    if ! check_command "ollama"; then
        print_message $RED "Error: ollama is not installed. Please install ollama first."
        print_message $YELLOW "Installation guide: https://ollama.com/download"
        exit 1
    fi
    
    # Check ports
    if ! check_port 8080; then
        print_message $YELLOW "Warning: ScienceBeam port (8080) is already in use. Make sure no other ScienceBeam instances are running."
    fi
    
    if ! check_port 8501; then
        print_message $YELLOW "Warning: Streamlit default port (8501) is already in use. Streamlit will try to use another port."
    fi
    
    if ! check_port $OLLAMA_PORT; then
        print_message $YELLOW "Warning: Ollama port ($OLLAMA_PORT) is already in use. This might indicate Ollama is already running."
    fi
    
    print_message $GREEN "Dependency check completed!"
}

# Load API keys
load_api_keys() {
    print_message $YELLOW "Checking API configuration..."
    
    # Check if API config file exists
    if [ -f "$API_CONFIG_FILE" ]; then
        # Source the config file to load variables
        source $API_CONFIG_FILE
        print_message $GREEN "API keys loaded from $API_CONFIG_FILE"
    else
        # If no config file, prompt for API keys
        print_message $YELLOW "No API configuration found. Would you like to configure API keys now? (y/n)"
        read configure_api
        
        if [[ $configure_api == "y" || $configure_api == "Y" ]]; then
            configure_api_keys
        else
            print_message $YELLOW "Continuing without API keys. Some features may not be available."
        fi
    fi
}

# Configure API keys
configure_api_keys() {
    print_message $GREEN "=== API Key Configuration ==="
    print_message $YELLOW "Enter your OpenAI API key (leave empty to skip):"
    read input_openai_key
    
    if [[ -n $input_openai_key ]]; then
        OPENAI_API_KEY=$input_openai_key
    fi
    
    print_message $YELLOW "Enter your Google API key (leave empty to skip):"
    read input_google_key
    
    if [[ -n $input_google_key ]]; then
        GOOGLE_API_KEY=$input_google_key
    fi
    
    # Save keys to config file
    echo "OPENAI_API_KEY=\"$OPENAI_API_KEY\"" > $API_CONFIG_FILE
    echo "GOOGLE_API_KEY=\"$GOOGLE_API_KEY\"" >> $API_CONFIG_FILE
    chmod 600 $API_CONFIG_FILE  # Secure the file
    
    print_message $GREEN "API keys saved to $API_CONFIG_FILE"
}

# Export API keys as environment variables
export_api_keys() {
    if [[ -n $OPENAI_API_KEY ]]; then
        export OPENAI_API_KEY=$OPENAI_API_KEY
        print_message $GREEN "OpenAI API key exported to environment"
    fi
    
    if [[ -n $GOOGLE_API_KEY ]]; then
        export GOOGLE_API_KEY=$GOOGLE_API_KEY
        print_message $GREEN "Google API key exported to environment"
    fi
}

# Start ScienceBeam (using conda environment)
start_sciencebeam() {
    print_message $YELLOW "Starting ScienceBeam service (using conda environment: $SCIENCEBEAM_CONDA_ENV)..."
    
    # Use conda to run ScienceBeam
    print_message $GREEN "Starting ScienceBeam service..."
    
    # Use nohup to run in background and redirect output to log file
    nohup bash -c "source $(conda info --base)/etc/profile.d/conda.sh && \
                  conda activate $SCIENCEBEAM_CONDA_ENV && \
                  python -m sciencebeam_parser.service.server --port=8080" > "$SCIENCEBEAM_LOG" 2>&1 &
    
    # Save ScienceBeam process PID
    SCIENCEBEAM_PID=$!
    echo $SCIENCEBEAM_PID > ".sciencebeam.pid"
    
    print_message $GREEN "ScienceBeam service starting! Process ID: $SCIENCEBEAM_PID"
    print_message $GREEN "Log file: $SCIENCEBEAM_LOG"
    
    # Wait for service to start
    local max_attempts=30
    local attempt=0
    local is_running=false
    
    print_message $YELLOW "Waiting for ScienceBeam service to start..."
    while [ $attempt -lt $max_attempts ]; do
        if curl -s http://localhost:8080/api/health > /dev/null || curl -s http://localhost:8080/health > /dev/null; then
            is_running=true
            break
        fi
        attempt=$((attempt+1))
        sleep 2
    done
    
    if [ "$is_running" = true ]; then
        print_message $GREEN "ScienceBeam service started successfully!"
    else
        print_message $RED "ScienceBeam service startup timed out. Please check the log file for more information:"
        print_message $YELLOW "tail -n 20 $SCIENCEBEAM_LOG"
        exit 1
    fi
}

# Start Ollama service
start_ollama() {
    print_message $YELLOW "Starting Ollama service..."
    
    # Check if Ollama is already running
    if curl -s http://localhost:$OLLAMA_PORT/api/tags > /dev/null; then
        print_message $GREEN "Ollama service is already running!"
        return 0
    fi
    
    # Start Ollama service in background
    print_message $GREEN "Starting Ollama service..."
    nohup ollama serve > "$OLLAMA_LOG" 2>&1 &
    
    # Save Ollama process PID
    OLLAMA_PID=$!
    echo $OLLAMA_PID > ".ollama.pid"
    
    print_message $GREEN "Ollama service starting! Process ID: $OLLAMA_PID"
    print_message $GREEN "Log file: $OLLAMA_LOG"
    
    # Wait for Ollama to start
    local max_attempts=30
    local attempt=0
    local is_running=false
    
    print_message $YELLOW "Waiting for Ollama service to start..."
    while [ $attempt -lt $max_attempts ]; do
        if curl -s http://localhost:$OLLAMA_PORT/api/tags > /dev/null; then
            is_running=true
            break
        fi
        attempt=$((attempt+1))
        sleep 2
    done
    
    if [ "$is_running" = true ]; then
        print_message $GREEN "Ollama service started successfully!"
    else
        print_message $RED "Ollama service startup timed out. Please check the log file for more information:"
        print_message $YELLOW "tail -n 20 $OLLAMA_LOG"
        exit 1
    fi
}

# Start Streamlit app
start_streamlit() {
    print_message $YELLOW "Starting Streamlit app (using conda environment: $STREAMLIT_CONDA_ENV)..."
    
    # Check if Streamlit app file exists
    if [ ! -f "$STREAMLIT_APP_PATH" ]; then
        print_message $RED "Error: Streamlit app file does not exist: $STREAMLIT_APP_PATH"
        exit 1
    fi
    
    # Start Streamlit app
    print_message $GREEN "Starting Streamlit app: $STREAMLIT_APP_PATH"
    nohup bash -c "source $(conda info --base)/etc/profile.d/conda.sh && \
                 conda activate $STREAMLIT_CONDA_ENV && \
                 streamlit run \"$STREAMLIT_APP_PATH\"" > "$STREAMLIT_LOG" 2>&1 &
    
    # Save Streamlit process PID
    STREAMLIT_PID=$!
    echo $STREAMLIT_PID > ".streamlit.pid"
    
    print_message $GREEN "Streamlit app started! Process ID: $STREAMLIT_PID"
    print_message $GREEN "Log file: $STREAMLIT_LOG"
    print_message $YELLOW "Waiting for Streamlit to start..."
    
    # Wait for Streamlit to start
    sleep 5
    if ps -p $STREAMLIT_PID > /dev/null; then
        print_message $GREEN "Streamlit app started successfully!"
        print_message $GREEN "Please visit http://localhost:8501 to use the app"
    else
        print_message $RED "Streamlit app failed to start. Please check the log file for more information:"
        print_message $YELLOW "tail -n 20 $STREAMLIT_LOG"
    fi
}

# Cleanup function
cleanup() {
    print_message $YELLOW "Cleaning up resources..."
    
    # Stop ScienceBeam process
    if [ -f ".sciencebeam.pid" ]; then
        local pid=$(cat ".sciencebeam.pid")
        if ps -p $pid > /dev/null; then
            kill $pid
            print_message $GREEN "ScienceBeam service stopped (PID: $pid)"
        fi
        rm ".sciencebeam.pid"
    fi
    
    # Stop Streamlit process
    if [ -f ".streamlit.pid" ]; then
        local pid=$(cat ".streamlit.pid")
        if ps -p $pid > /dev/null; then
            kill $pid
            print_message $GREEN "Streamlit app stopped (PID: $pid)"
        fi
        rm ".streamlit.pid"
    fi
    
    # Stop Ollama process
    if [ -f ".ollama.pid" ]; then
        local pid=$(cat ".ollama.pid")
        if ps -p $pid > /dev/null; then
            kill $pid
            print_message $GREEN "Ollama service stopped (PID: $pid)"
        fi
        rm ".ollama.pid"
    fi
    
    print_message $GREEN "Cleanup completed!"
    exit 0
}

# Register cleanup handler
trap cleanup SIGINT SIGTERM

# Main function
main() {
    print_message $GREEN "===== Service Startup Script ====="
    
    # Check dependencies
    check_dependencies
    
    # Load API keys
    load_api_keys
    
    # Export API keys
    export_api_keys
    
    # Start Ollama service
    start_ollama
    
    # Start ScienceBeam
    start_sciencebeam
    
    # Start Streamlit
    start_streamlit
    
    print_message $GREEN "===== All Services Started ====="
    print_message $GREEN "Ollama API: http://localhost:$OLLAMA_PORT"
    print_message $GREEN "ScienceBeam API: http://localhost:8080"
    print_message $GREEN "Streamlit app: http://localhost:8501"
    print_message $YELLOW "Press Ctrl+C to stop all services"
    
    # Keep script running until interrupt signal is received
    while true; do
        sleep 60
    done
}

# Execute main function
main
EOF
        
        chmod +x start_services.zsh
        print_message $GREEN "Created start_services.zsh script."
    else
        print_message $GREEN "start_services.zsh script already exists."
    fi
}

# Main function
main() {
    print_message $GREEN "===== Scientific Paper Review System - Environment Setup ====="
    
    # Check basic dependencies
    check_command wget || { print_message $RED "Please install wget and retry."; exit 1; }
    check_command curl || { print_message $RED "Please install curl and retry."; exit 1; }
    
    # Install Conda if needed
    install_conda
    
    # Install Ollama if needed
    install_ollama
    
    # Create directory structure
    create_directory_structure
    
    # Create conda environments
    create_sciencebeam_env
    create_llm_env
    
    # Create start script
    create_start_script
    
    # Download Ollama models
    download_ollama_models
    
    # Configure API keys
    configure_api_keys
    
    print_message $GREEN "===== Environment Setup Complete ====="
    print_message $GREEN "You can now start the system by running:"
    print_message $YELLOW "./start_services.zsh"
    print_message $GREEN "======================================"
}

# Run main function
main
```

## Manual Setup Instructions

If you prefer to set up the environment manually, follow these steps:

### 1. Install Conda (Miniconda)

```bash
# Download Miniconda installer
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# Make installer executable
chmod +x Miniconda3-latest-Linux-x86_64.sh

# Run installer
./Miniconda3-latest-Linux-x86_64.sh

# Follow the prompts to complete installation
# Restart your terminal after installation
```

### 2. Install Ollama

```bash
# Download and run Ollama installer
curl -fsSL https://ollama.com/install.sh | sh

# Verify installation
ollama --version
```

### 3. Create Conda Environments

#### ScienceBeam Environment (for PDF parsing)

```bash
# Create environment
conda create -n ScienceBeam python=3.9 -y

# Activate environment
conda activate ScienceBeam

# Install required packages
pip install sciencebeam-parser
pip install tornado
pip install lxml

# Verify installation
python -c "import sciencebeam_parser; print(sciencebeam_parser.__version__)"

# Deactivate environment
conda deactivate
```

#### LLM Environment (for Streamlit and AI models)

```bash
# Create environment
conda create -n llm python=3.10 -y

# Activate environment
conda activate llm

# Install base packages
pip install streamlit langchain langchain-community langchain-openai

# Install PDF and token handling
pip install tiktoken pikepdf

# Install vector search
pip install faiss-cpu

# Install data handling
pip install pandas numpy

# Install API clients
pip install openai google-generativeai requests

# Install additional RAG components
pip install sentence-transformers chromadb

# Deactivate environment
conda deactivate
```

### 4. Download Ollama Models

```bash
# Start Ollama service (if not already running)
ollama serve &

# Pull required models
ollama pull llama3.1:latest
ollama pull qwen2.5:latest
```

### 5. Set Up API Keys

```bash
# Create API keys file
cat > .api_keys << EOF
OPENAI_API_KEY="your_openai_api_key_here"
GOOGLE_API_KEY="your_google_api_key_here"
EOF

# Secure the file
chmod 600 .api_keys
```

### 6. Create Project Structure

```bash
# Create required directories
mkdir -p cache logs data finetune plots
```

## Starting the System

After setting up the environment (either using the script or manually), you can start the system using the `start_services.zsh` script:

```bash
./start_services.zsh
```

This script will:
1. Check all dependencies
2. Load API keys from the `.api_keys` file (or prompt for them)
3. Start the Ollama service for running local models
4. Start the ScienceBeam service for PDF parsing
5. Start the Streamlit application for the web interface

## Accessing the System

Once all services are running, access the system through your web browser at:
```
http://localhost:8501
```

## Troubleshooting

### Common Issues

1. **Missing Dependencies**:
   - If you encounter missing Python packages, activate the appropriate environment and install the missing packages.

2. **Port Conflicts**:
   - The system uses ports 8080 (ScienceBeam), 8501 (Streamlit), and 11434 (Ollama).
   - If these ports are already in use, you'll need to terminate the conflicting processes.

3. **Model Download Errors**:
   - If Ollama fails to download models, check your internet connection and retry.
   - You can manually download models with `ollama pull model_name`.

4. **API Key Issues**:
   - If features requiring API keys don't work, check that your API keys are correctly configured in the `.api_keys` file.

### Logs

Check the log files in the `logs` directory for detailed error information:
- `sciencebeam_*.log` for ScienceBeam issues
- `streamlit_*.log` for Streamlit application issues
- `ollama_*.log` for Ollama model issues

## System Maintenance

### Updating Models

To update Ollama models:

```bash
ollama pull llama3.1:latest --update
ollama pull qwen2.5:latest --update
```

### Cleaning Cache

The system automatically cleans the cache directory at startup, but you can manually clean it:

```bash
rm -rf cache/*
rm -rf logs/*
```

---

This setup guide should provide everything needed to create and configure the environment for the Scientific Paper Review System. If you encounter any issues during setup, refer to the troubleshooting section or check the log files for more detailed error information.