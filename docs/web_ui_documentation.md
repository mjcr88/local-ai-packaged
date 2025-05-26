Open Web UI Documentation

Open WebUI
Open WebUI is an extensible, feature-rich, and user-friendly self-hosted AI platform designed to operate entirely offline. It supports various LLM runners like Ollama and OpenAI-compatible APIs, with built-in inference engine for RAG, making it a powerful AI deployment solution.

Quick Start with Docker
WebSocket support is required for Open WebUI to function correctly. Ensure that your network configuration allows WebSocket connections.
If Ollama is on your computer, use this command:

Bash

docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
To run Open WebUI with Nvidia GPU support, use this command:

Bash

docker run -d -p 3000:8080 --gpus all --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:cuda
Open WebUI Bundled with Ollama
This installation method uses a single container image that bundles Open WebUI with Ollama, allowing for a streamlined setup via a single command. Choose the appropriate command based on your hardware setup:
With GPU Support: Utilize GPU resources by running the following command:

Bash

docker run -d -p 3000:8080 --gpus=all -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
For CPU Only: If you're not using a GPU, use this command instead:

Bash

docker run -d -p 3000:8080 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
Both commands facilitate a built-in, hassle-free installation of both Open WebUI and Ollama, ensuring that you can get everything up and running swiftly.
After installation, you can access Open WebUI at http://localhost:3000.

Using the Dev Branch
The :dev branch contains the latest unstable features and changes. Use it at your own risk as it may have bugs or incomplete features.
If you want to try out the latest bleeding-edge features and are okay with occasional instability, you can use the :dev tag like this:

Bash

docker run -d -p 3000:8080 -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:dev
Updating Open WebUI (Docker)
To update Open WebUI container easily, follow these steps:

Manual Update
Use Watchtower to update your Docker container manually:

Bash

docker run --rm -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --run-once open-webui
Automatic Updates
Keep your container updated automatically every 5 minutes:

Bash

docker run -d --name watchtower --restart unless-stopped -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --interval 300 open-webui
Note: Replace open-webui with your container name if it's different.

Manual Installation
There are two main ways to install and run Open WebUI: using the uv runtime manager or Python's pip. While both methods are effective, we strongly recommend using uv as it simplifies environment management and minimizes potential conflicts.

Installation with uv (Recommended)
The uv runtime manager ensures seamless Python environment management for applications like Open WebUI. Follow these steps to get started:

Install uv Pick the appropriate installation command for your operating system:
macOS/Linux:
Bash

curl -LsSf https://astral.sh/uv/install.sh | sh
Windows:
PowerShell

powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
Run Open WebUI Once uv is installed, running Open WebUI is a breeze. Use the command below, ensuring to set the DATA_DIR environment variable to avoid data loss. Example paths are provided for each platform:
macOS/Linux:
Bash

DATA_DIR=~/.open-webui uvx --python 3.11 open-webui@latest serve
Windows:
PowerShell

$env:DATA_DIR="C:\open-webui\data"; uvx --python 3.11 open-webui@latest serve
Installation with pip
For users installing Open WebUI with Python's package manager pip, it is strongly recommended to use Python runtime managers like uv or conda. These tools help manage Python environments effectively and avoid conflicts.
Python 3.11 is the development environment. Python 3.12 seems to work but has not been thoroughly tested. Python 3.13 is entirely untested—use at your own risk.
Install Open WebUI:
Open your terminal and run the following command:

Bash

pip install open-webui
Start Open WebUI:
Once installed, start the server using:

Bash

open-webui serve
Updating Open WebUI (pip)
To update to the latest version, simply run:

Bash

pip install --upgrade open-webui
This method installs all necessary dependencies and starts Open WebUI, allowing for a simple and efficient setup. After installation, you can access Open WebUI at http://localhost:8080.

Other Installation Methods
We offer various installation alternatives, including non-Docker native installation methods, Docker Compose, Kustomize, and Helm. Visit our Open WebUI Documentation or join our Discord community for comprehensive guidance.

Quick Start
Important Note on User Roles and Privacy:

Admin Creation: The first account created on Open WebUI gains Administrator privileges, controlling user management and system settings.
User Registrations: Subsequent sign-ups start with Pending status, requiring Administrator approval for access.
Privacy and Data Security: All your data, including login details, is locally stored on your device. Open WebUI ensures strict confidentiality and no external requests for enhanced privacy and security. All models are private by default. Models must be explicitly shared via groups or by being made public. If a model is assigned to a group, only members of that group can see it. If a model is made public, anyone on the instance can see it.
Choose your preferred installation method below:

Docker: Officially supported and recommended for most users
Python: Suitable for low-resource environments or those wanting a manual setup
Kubernetes: Ideal for enterprise deployments that require scaling and orchestration
Docker Compose
Podman
Docker Swarm
Docker
Quick Start with Docker
Follow these steps to install Open WebUI with Docker.
Step 1: Pull the Open WebUI Image
Start by pulling the latest Open WebUI Docker image from the GitHub Container Registry.

Bash

docker pull ghcr.io/open-webui/open-webui:main
Step 2: Run the Container
Run the container with default settings. This command includes a volume mapping to ensure persistent data storage.

Bash

docker run -d -p 3000:8080 -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
Important Flags

Volume Mapping (-v open-webui:/app/backend/data): Ensures persistent storage of your data. This prevents data loss between container restarts.
Port Mapping (-p 3000:8080): Exposes the WebUI on port 3000 of your local machine.
Using GPU Support
For Nvidia GPU support, add --gpus all to the docker run command:

Bash

docker run -d -p 3000:8080 --gpus all -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:cuda
Single-User Mode (Disabling Login)
To bypass the login page for a single-user setup, set the WEBUI_AUTH environment variable to False:

Bash

docker run -d -p 3000:8080 -e WEBUI_AUTH=False -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
You cannot switch between single-user mode and multi-account mode after this change.

Advanced Configuration: Connecting to Ollama on a Different Server
To connect Open WebUI to an Ollama server located on another host, add the OLLAMA_BASE_URL environment variable:

Bash

docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=https://example.com -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
Access the WebUI
After the container is running, access Open WebUI at:
http://localhost:3000
For detailed help on each Docker flag, see Docker's documentation.

Updating (Docker)
To update your local Docker installation to the latest version, you can either use Watchtower or manually update the container.
Option 1: Using Watchtower
With Watchtower, you can automate the update process:

Bash

docker run --rm --volume /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower --run-once open-webui
(Replace open-webui with your container's name if it's different.)
Option 2: Manual Update
Stop and remove the current container:

Bash

docker rm -f open-webui
Pull the latest version:

Bash

docker pull ghcr.io/open-webui/open-webui:main
Start the container again:

Bash

docker run -d -p 3000:8080 -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
Both methods will get your Docker instance updated and running with the latest build.

Next Steps
After installing, visit:
http://localhost:3000 to access Open WebUI.
or http://localhost:8080/ when using a Python deployment.
You are now ready to start using Open WebUI!

Using Open WebUI with Ollama
If you're using Open WebUI with Ollama, be sure to check out our Starting with Ollama Guide to learn how to manage your Ollama instances with Open WebUI.

Starting With Ollama
Overview
Open WebUI makes it easy to connect and manage your Ollama instance. This guide will walk you through setting up the connection, managing models, and getting started.
Step 1: Setting Up the Ollama Connection
Once Open WebUI is installed and running, it will automatically attempt to connect to your Ollama instance. If everything goes smoothly, you’ll be ready to manage and use models right away.
However, if you encounter connection issues, the most common cause is a network misconfiguration. You can refer to our connection troubleshooting guide for help resolving these problems.
Step 2: Managing Your Ollama Instance
To manage your Ollama instance in Open WebUI, follow these steps:

Go to Admin Settings in Open WebUI.
Navigate to Connections > Ollama > Manage (click the wrench icon).
From here, you can download models, configure settings, and manage your connection to Ollama.
A Quick and Efficient Way to Download Models
If you’re looking for a faster option to get started, you can download models directly from the Model Selector. Simply type the name of the model you want, and if it’s not already available, Open WebUI will prompt you to download it from Ollama.
This method is perfect if you want to skip navigating through the Admin Settings menu and get right to using your models.
Once your connection is configured and your models are downloaded, you’re ready to start using Ollama with Open WebUI. If you run into any issues or need more guidance, check out our help section for detailed solutions.

Starting With OpenAI
Overview
Open WebUI makes it easy to connect and use OpenAI and other OpenAI-compatible APIs. This guide will walk you through adding your API key, setting the correct endpoint, and selecting models — so you can start chatting right away.
Step 1: Get Your OpenAI API Key
To use OpenAI models (such as GPT-4 or o3-mini), you need an API key from a supported provider.
You can use:

OpenAI directly (https://platform.openai.com/account/api-keys)
Azure OpenAI
Any OpenAI-compatible service (e.g., LocalAI, FastChat, Helicone, LiteLLM, OpenRouter etc.) Once you have the key, copy it and keep it handy. For most OpenAI usage, the default API base URL is: https://api.openai.com/v1 Other providers use different URLs — check your provider’s documentation. Step 2: Add the API Connection in Open WebUI Once Open WebUI is running:
Go to the Admin Settings.
Navigate to Connections > OpenAI > Manage (look for the wrench icon).
Click Add New Connection.
Fill in the following:
API URL: https://api.openai.com/v1 (or the URL of your specific provider)
API Key: Paste your key here
Click Save. This securely stores your credentials and sets up the connection. Step 3: Start Using Models Once your connection is saved, you can start using models right inside Open WebUI. You don’t need to download any models — just select one from the Model Selector and start chatting. If a model is supported by your provider, you’ll be able to use it instantly via their API. Simply choose GPT-4, o3-mini, or any compatible model offered by your provider. Your OpenAI-compatible API connection is ready to use. If you run into issues or need additional support, visit our help section.
Starting with Llama.cpp
Overview
Open WebUI makes it simple and flexible to connect and manage a local Llama.cpp server to run efficient, quantized language models. Whether you’ve compiled Llama.cpp yourself or you're using precompiled binaries, this guide will walk you through how to:

Set up your Llama.cpp server
Load large models locally
Integrate with Open WebUI for a seamless interface Step 1: Install Llama.cpp To run models with Llama.cpp, you first need the Llama.cpp server installed locally. You can either:
Download prebuilt binaries
Or build it from source by following the official build instructions After installing, make sure llama-server is available in your local system path or take note of its location. Step 2: Download a Supported Model You can load and run various GGUF-format quantized LLMs using Llama.cpp. One impressive example is the DeepSeek-R1 1.58-bit model optimized by UnslothAI. To download this version:
Visit the Unsloth DeepSeek-R1 repository on Hugging Face
Download the 1.58-bit quantized version – around 131GB. Alternatively, use Python to download programmatically:
Python

# pip install huggingface_hub hf_transfer

from huggingface_hub import snapshot_download

snapshot_download(
    repo_id = "unsloth/DeepSeek-R1-GGUF",
    local_dir = "DeepSeek-R1-GGUF",
    allow_patterns = ["*UD-IQ1_S*"], # Download only 1.58-bit variant
)
This will download the model files into a directory like:

DeepSeek-R1-GGUF/
└── DeepSeek-R1-UD-IQ1_S/
    ├── DeepSeek-R1-UD-IQ1_S-00001-of-00003.gguf
    ├── DeepSeek-R1-UD-IQ1_S-00002-of-00003.gguf
    └── DeepSeek-R1-UD-IQ1_S-00003-of-00003.gguf
Keep track of the full path to the first GGUF file — you’ll need it in Step 3.
Step 3: Serve the Model with Llama.cpp
Start the model server using the llama-server binary. Navigate to your llama.cpp folder (e.g., build/bin) and run:

Bash

./llama-server \
 --model /your/full/path/to/DeepSeek-R1-UD-IQ1_S-00001-of-00003.gguf \
 --port 10000 \
 --ctx-size 1024 \
 --n-gpu-layers 40
Tweak the parameters to suit your machine:

--model: Path to your .gguf model file
--port: 10000 (or choose another open port)
--ctx-size: Token context length (can increase if RAM allows)
--n-gpu-layers: Layers offloaded to GPU for faster performance Once the server runs, it will expose a local OpenAI-compatible API on: http://127.0.0.1:10000 Step 4: Connect Llama.cpp to Open WebUI To control and query your locally running model directly from Open WebUI:
Open Open WebUI in your browser
Go to Admin Settings → Connections → OpenAI Connections
Click Add Connection and enter:
URL: http://127.0.0.1:10000/v1 (Or use http://host.docker.internal:10000/v1 if running WebUI inside Docker)
API Key: none (leave blank) Once saved, Open WebUI will begin using your local Llama.cpp server as a backend!
Quick Tip: Try Out the Model via Chat Interface
Once connected, select the model from the Open WebUI chat menu and start interacting!

Once configured, Open WebUI makes it easy to:

Manage and switch between local models served by Llama.cpp
Use the OpenAI-compatible API with no key needed
Experiment with massive models like DeepSeek-R1 — right from your machine!
Advanced Topics
Explore deeper concepts and advanced configurations of Open WebUI to enhance your setup.

Development: Understand the development setup and contribute to Open WebUI. (Development Guide)
Logging: Learn how to configure and manage logs for troubleshooting your system. (Logging Guide)
HTTPS Encryption: Ensure secure communication by implementing HTTPS encryption in your deployment. (HTTPS Encryption Guide)
Monitoring: Learn how to monitor your Open WebUI instance, including health checks, model connectivity, and response testing. (Monitoring Guide)
Network Diagrams
Each scenario is illustrated using Mermaid diagrams to show how the interactions are set up depending on the different system configurations and deployment strategies.
Mac OS/Windows Setup Options

Ollama on Host, Open WebUI in Container
Code snippet

graph TD
    User("<<person>>User") -->|Makes requests via exposed port -p 3000:8080<br>[http://localhost:3000]| OpenWebUIContainer["<<component>>Open WebUI<br>[Listening on port 8080]"]
    OpenWebUIContainer -->|Makes API calls via Docker proxy<br>[http://host.docker.internal:11434]| OllamaHost["<<component>>Ollama<br>[Listening on port 11434]"]

    subgraph Docker Desktop's Linux VM [system]
        OpenWebUIContainer
    end

    subgraph Hosting Machine - Mac OS/Windows [system]
        OllamaHost
        User
    end
Ollama and Open WebUI in Compose Stack
Code snippet

graph TD
    User("<<person>>User") -->|Makes requests via exposed port -p 3000:8080<br>[http://localhost:3000]| OpenWebUICompose["<<component>>Open WebUI<br>[Listening on port 8080]"]

    subgraph Compose Stack [system]
        OpenWebUICompose -->|Makes API calls via inter-container networking<br>[http://ollama:11434]| OllamaCompose["<<component>>Ollama<br>[Listening on port 11434]"]
    end

    subgraph Docker Desktop's Linux VM [system]
        ComposeStack
    end

    subgraph Hosting Machine - Mac OS/Windows [system]
        User
    end
Ollama and Open WebUI, Separate Networks
Code snippet

graph TD
    User("<<person>>User") -->|Makes requests via exposed port -p 3000:8080<br>[http://localhost:3000]| OpenWebUINetA["<<component>>Open WebUI<br>[Listening on port 8080]"]
    OllamaNetB["<<component>>Ollama<br>[Listening on port 11434]"]

    subgraph Network A [system]
        OpenWebUINetA
    end
    OpenWebUINetA -.->|Unable to connect| OllamaNetB

    subgraph Network B [system]
        OllamaNetB
    end

    subgraph Docker Desktop's Linux VM [system]
        NetworkA
        NetworkB
    end
    subgraph Hosting Machine - Mac OS/Windows [system]
        User
    end
Open WebUI in Host Network
Code snippet

graph TD
    User("<<person>>User")
    OpenWebUIHostNet["<<component>>Open WebUI<br>[Listening on port 8080]"]
    User -.->|Unable to connect, host network is the VM's network| OpenWebUIHostNet

    subgraph Docker Desktop's Linux VM [system]
       OpenWebUIHostNet
    end
    subgraph Hosting Machine - Mac OS/Windows [system]
        User
    end
Linux Setup Options

Ollama on Host, Open WebUI in Container (Linux)
Code snippet

graph TD
    User("<<person>>User") -->|Makes requests via exposed port -p 3000:8080<br>[http://localhost:3000]| OpenWebUILinuxContainer["<<component>>Open WebUI<br>[Listening on port 8080]"]
    OpenWebUILinuxContainer -->|Makes API calls via Docker proxy<br>[http://host.docker.internal:11434]| OllamaLinuxHost["<<component>>Ollama<br>[Listening on port 11434]"]

    subgraph Container Network [system]
        OpenWebUILinuxContainer
    end

    subgraph Hosting Machine - Linux [system]
        OllamaLinuxHost
        User
    end
Ollama and Open WebUI in Compose Stack (Linux)
Code snippet

graph TD
    User("<<person>>User") -->|Makes requests via exposed port -p 3000:8080<br>[http://localhost:3000]| OpenWebUILinuxCompose["<<component>>Open WebUI<br>[Listening on port 8080]"]

    subgraph Compose Stack [system]
        OpenWebUILinuxCompose -->|Makes API calls via inter-container networking<br>[http://ollama:11434]| OllamaLinuxCompose["<<component>>Ollama<br>[Listening on port 11434]"]
    end

    subgraph Container Network [system]
        ComposeStackLinux
    end

    subgraph Hosting Machine - Linux [system]
        User
    end
Ollama and Open WebUI, Separate Networks (Linux)
Code snippet

graph TD
    User("<<person>>User") -->|Makes requests via exposed port -p 3000:8080<br>[http://localhost:3000]| OpenWebUILinuxNetA["<<component>>Open WebUI<br>[Listening on port 8080]"]
    OllamaLinuxNetB["<<component>>Ollama<br>[Listening on port 11434]"]

    subgraph Container Network A [system]
        OpenWebUILinuxNetA
    end
    OpenWebUILinuxNetA -.->|Unable to connect| OllamaLinuxNetB

    subgraph Container Network B [system]
        OllamaLinuxNetB
    end
    subgraph Hosting Machine - Linux [system]
        User
    end
Open WebUI in Host Network, Ollama on Host (Linux)
Code snippet

graph TD
    User("<<person>>User") -->|Makes requests via listening port<br>[http://localhost:8080]| OpenWebUILinuxHostNet["<<component>>Open WebUI<br>[Listening on port 8080]"]
    OpenWebUILinuxHostNet -->|Makes API calls via localhost<br>[http://localhost:11434]| OllamaLinuxHostNet["<<component>>Ollama<br>[Listening on port 11434]"]

    subgraph Hosting Machine - Linux [system]
        User
        OpenWebUILinuxHostNet
        OllamaLinuxHostNet
    end
Each setup addresses different deployment strategies and networking configurations to help you choose the best layout for your requirements.

Development Setup Guide
This guide will walk you through setting up your local development environment.
Prerequisites
Before you begin, ensure your system meets these minimum requirements:

Operating System: Linux (or WSL on Windows), Windows 11, or macOS. (Recommended for best compatibility)
Python: Version 3.11 or higher. (Required for backend services)
Node.js: Version 22.10 or higher. (Required for frontend development)
IDE (Recommended): We recommend using an IDE like VSCode for code editing, debugging, and integrated terminal access.
[Optional] GitHub Desktop: For easier management of the Git repository.
Setting Up Your Local Environment
We'll set up both the frontend (user interface) and backend (API and server logic) of Open WebUI.
1. Clone the Repository
First, use git clone to download the Open WebUI repository to your local machine.
Open your terminal.
Navigate to the directory where you want to store the Open WebUI project.
Clone the repository: Run the following command:

Bash

git clone https://github.com/open-webui/open-webui.git
cd open-webui
2. Frontend Setup (User Interface)
Configure Environment Variables:
Copy the example environment file to .env:

Bash

cp -RPp .env.example .env
Customize .env: Open the .env file in your code editor. For local development, the default settings in .env.example are usually sufficient.
Important: Do not commit sensitive information to .env if you are contributing back to the repository.
Install Frontend Dependencies:
Navigate to the frontend directory (project root open-webui).
Install the required JavaScript packages:

Bash

npm install
Start the Frontend Development Server:

Bash

npm run dev
Access the Frontend: Open your web browser and go to http://localhost:5173.

3. Backend Setup (API and Server)
We strongly recommend using separate terminal instances for your frontend and backend processes.
Using VSCode Integrated Terminals (Recommended):

Frontend Terminal: Use your existing terminal at the project root (open-webui).
Backend Terminal (Open a New One): In VSCode, go to Terminal > New Terminal. Navigate to the backend directory: cd backend.
Backend Setup Steps (in your backend terminal):
Navigate to the Backend Directory: (If not already there)

Bash

cd backend
Create and Activate a Conda Environment (Recommended):

Bash

conda create --name open-webui python=3.11
conda activate open-webui
Install Backend Dependencies:
In your backend terminal (with the Conda environment activated if using Conda), run:

Bash

pip install -r requirements.txt -U
Start the Backend Development Server:
In your backend terminal, run:

Bash

sh dev.sh
Explore the API Documentation: Once the backend is running, you can access the API documentation at http://localhost:8080/docs.
You should now have both the frontend and backend development servers running locally. Refresh the frontend page (http://localhost:5173). You should see the full Open WebUI application running.

Troubleshooting Common Issues

"FATAL ERROR: Reached Heap Limit" (Frontend) This error indicates that Node.js is running out of memory. Solution: Increase the Node.js heap size.