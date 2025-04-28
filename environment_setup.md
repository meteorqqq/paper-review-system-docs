# 科学论文评审系统环境设置指南

本指南将帮助您从头开始创建运行科学论文评审系统所需的虚拟环境。系统需要两个主要的Conda环境：一个用于ScienceBeam服务（PDF转XML），另一个用于Streamlit应用和大语言模型。

## 前提条件

在开始之前，请确保您的系统上已安装以下软件：

1. **Conda** (Miniconda或Anaconda)
2. **Git**（用于克隆代码库）
3. **Ollama**（用于本地运行开源大语言模型）

## 1. 安装Conda

如果您尚未安装Conda，可以按照以下步骤进行安装：

### Linux系统安装Miniconda

```bash
# 下载Miniconda安装脚本
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# 赋予执行权限
chmod +x Miniconda3-latest-Linux-x86_64.sh

# 运行安装脚本
./Miniconda3-latest-Linux-x86_64.sh

# 按照提示完成安装
# 安装完成后，重新打开终端或运行以下命令激活conda
source ~/.bashrc
```

## 2. 安装Ollama

Ollama是运行本地大语言模型的工具，按照以下步骤在Linux上安装：

```bash
# 下载并安装Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 验证安装
ollama --version
```

## 3. 创建ScienceBeam环境

ScienceBeam用于将PDF文件转换为XML格式，便于系统分析论文内容。

```bash
# 创建ScienceBeam环境
conda create -n ScienceBeam python=3.9 -y

# 激活环境
conda activate ScienceBeam

# 安装ScienceBeam及其依赖
pip install sciencebeam-parser
pip install tornado  # 用于运行Web服务
pip install lxml

# 验证安装
python -c "from sciencebeam_parser import __version__; print(__version__)"
```

## 4. 创建LLM环境

这个环境用于运行Streamlit应用程序和处理大语言模型交互。

```bash
# 创建LLM环境
conda create -n llm python=3.10 -y

# 激活环境
conda activate llm

# 安装基础依赖
pip install streamlit
pip install langchain langchain-community langchain-openai
pip install tiktoken
pip install pikepdf
pip install faiss-cpu  # 用于向量搜索
pip install pandas numpy

# 安装OpenAI SDK
pip install openai

# 安装Google GenerativeAI SDK
pip install google-generativeai

# 安装其他必要的库
pip install requests
```

## 5. 下载和准备模型

使用Ollama下载必要的模型：

```bash
# 下载Llama 3.1模型（用于生成评审）
ollama pull llama3.1:latest

# 下载Qwen2.5模型（用于论文评分）
ollama pull qwen2.5:latest
```

## 6. 环境变量设置

如果您计划使用OpenAI的GPT模型或Google的Gemini模型，需要设置相应的API密钥：

```bash
# 设置OpenAI API密钥（如果使用GPT模型）
export OPENAI_API_KEY="your_openai_api_key_here"

# 设置Google API密钥（如果使用Gemini模型）
export GOOGLE_API_KEY="your_google_api_key_here"
```

或者，您可以将以上环境变量添加到`~/.bashrc`文件中实现永久设置：

```bash
# 添加到~/.bashrc文件
echo 'export OPENAI_API_KEY="your_openai_api_key_here"' >> ~/.bashrc
echo 'export GOOGLE_API_KEY="your_google_api_key_here"' >> ~/.bashrc
source ~/.bashrc
```

## 7. 创建项目目录结构

为了确保系统正常运行，需要创建正确的目录结构：

```bash
# 创建项目目录
mkdir -p ~/review_model
cd ~/review_model

# 创建必要的子目录
mkdir -p cache
mkdir -p logs
mkdir -p data
```

## 8. 启动服务

现在您可以启动整个系统：

```bash
# 首先，启动ScienceBeam服务
conda activate ScienceBeam
python -m sciencebeam_parser.service.server --port=8080 &

# 在另一个终端中，启动Ollama服务（如果尚未运行）
ollama serve &

# 启动Streamlit应用
conda activate llm
streamlit run streamlit_rag.py
```

或者，您可以使用项目中提供的`start_services.zsh`脚本一键启动所有服务：

```bash
# 确保脚本有执行权限
chmod +x start_services.zsh

# 运行启动脚本
./start_services.zsh
```

## 9. 系统访问

启动成功后，您可以通过浏览器访问系统：

```
http://localhost:8501
```

## 10. 故障排除

如果遇到问题，请检查以下几点：

### ScienceBeam服务问题
- 确认ScienceBeam环境正确安装
- 检查端口8080是否被占用
- 查看logs目录下的ScienceBeam日志

### Ollama服务问题
- 确认Ollama已正确安装
- 检查端口11434是否被占用
- 确认所需模型已下载

### Streamlit应用问题
- 确认llm环境的所有依赖已安装
- 检查端口8501是否被占用
- 查看logs目录下的Streamlit日志

## 11. 清理环境

当您不再需要使用系统时，可以关闭所有服务并清理环境：

```bash
# 如果使用start_services.zsh启动，按Ctrl+C停止
# 或者手动关闭各个服务：

# 查找并关闭ScienceBeam进程
pkill -f "sciencebeam_parser.service.server"

# 关闭Ollama服务
pkill -f "ollama serve"

# 关闭Streamlit应用
pkill -f "streamlit run"
```

这样，您应该能够成功地从头创建并配置运行科学论文评审系统所需的虚拟环境了。

## 环境常用命令参考

```bash
# 列出所有conda环境
conda env list

# 激活ScienceBeam环境
conda activate ScienceBeam

# 激活LLM环境
conda activate llm

# 退出当前环境
conda deactivate

# 列出Ollama上安装的模型
ollama list

# 更新Ollama模型
ollama pull llama3.1:latest --update
```