Installation & Setup (2023 Edition)
1. Basic Installation
2. Initial Setup & Diagnostics
3. Verifying Installation: A Simple Crawl (Skip this step if you already run crawl4ai-doctor)
4. Advanced Installation (Optional)
5. Docker (Experimental)
6. Local Server Mode (Legacy)
Summary
Installation & Setup (2023 Edition)
1. Basic Installation
pip install crawl4ai
This installs the core Crawl4AI library along with essential dependencies. No advanced features (like transformers or PyTorch) are included yet.

2. Initial Setup & Diagnostics
2.1 Run the Setup Command
After installing, call:

crawl4ai-setup
What does it do? - Installs or updates required Playwright browsers (Chromium, Firefox, etc.) - Performs OS-level checks (e.g., missing libs on Linux) - Confirms your environment is ready to crawl

2.2 Diagnostics
Optionally, you can run diagnostics to confirm everything is functioning:

crawl4ai-doctor
This command attempts to: - Check Python version compatibility - Verify Playwright installation - Inspect environment variables or library conflicts

If any issues arise, follow its suggestions (e.g., installing additional system packages) and re-run crawl4ai-setup.

3. Verifying Installation: A Simple Crawl (Skip this step if you already run crawl4ai-doctor)
Below is a minimal Python script demonstrating a basic crawl. It uses our new BrowserConfig and CrawlerRunConfig for clarity, though no custom settings are passed in this example:

import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig

async def main():
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://www.example.com",
        )
        print(result.markdown[:300])  # Show the first 300 characters of extracted text

if __name__ == "__main__":
    asyncio.run(main())
Expected outcome: - A headless browser session loads example.com - Crawl4AI returns ~300 characters of markdown.
If errors occur, rerun crawl4ai-doctor or manually ensure Playwright is installed correctly.

4. Advanced Installation (Optional)
Warning: Only install these if you truly need them. They bring in larger dependencies, including big models, which can increase disk usage and memory load significantly.

4.1 Torch, Transformers, or All
Text Clustering (Torch)

pip install crawl4ai[torch]
crawl4ai-setup
Installs PyTorch-based features (e.g., cosine similarity or advanced semantic chunking).
Transformers

pip install crawl4ai[transformer]
crawl4ai-setup
Adds Hugging Face-based summarization or generation strategies.
All Features

pip install crawl4ai[all]
crawl4ai-setup
(Optional) Pre-Fetching Models
crawl4ai-download-models
This step caches large models locally (if needed). Only do this if your workflow requires them.
5. Docker (Experimental)
We provide a temporary Docker approach for testing. It’s not stable and may break with future releases. We plan a major Docker revamp in a future stable version, 2025 Q1. If you still want to try:

docker pull unclecode/crawl4ai:basic
docker run -p 11235:11235 unclecode/crawl4ai:basic
You can then make POST requests to http://localhost:11235/crawl to perform crawls. Production usage is discouraged until our new Docker approach is ready (planned in Jan or Feb 2025).

6. Local Server Mode (Legacy)
Some older docs mention running Crawl4AI as a local server. This approach has been partially replaced by the new Docker-based prototype and upcoming stable server release. You can experiment, but expect major changes. Official local server instructions will arrive once the new Docker architecture is finalized.

Summary
1. Install with pip install crawl4ai and run crawl4ai-setup. 2. Diagnose with crawl4ai-doctor if you see errors. 3. Verify by crawling example.com with minimal BrowserConfig + CrawlerRunConfig. 4. Advanced features (Torch, Transformers) are optional—avoid them if you don’t need them (they significantly increase resource usage). 5. Docker is experimental—use at your own risk until the stable version is released. 6. Local server references in older docs are largely deprecated; a new solution is in progress.
-----------------
Docker Deployment
Crawl4AI provides official Docker images for easy deployment and scalability. This guide covers installation, configuration, and usage of Crawl4AI in Docker environments.

Quick Start 🚀
Pull and run the basic version:

# Basic run without security
docker pull unclecode/crawl4ai:basic
docker run -p 11235:11235 unclecode/crawl4ai:basic

# Run with API security enabled
docker run -p 11235:11235 -e CRAWL4AI_API_TOKEN=your_secret_token unclecode/crawl4ai:basic
Running with Docker Compose 🐳
Use Docker Compose (From Local Dockerfile or Docker Hub)
Crawl4AI provides flexibility to use Docker Compose for managing your containerized services. You can either build the image locally from the provided Dockerfile or use the pre-built image from Docker Hub.

Option 1: Using Docker Compose to Build Locally
If you want to build the image locally, use the provided docker-compose.local.yml file.

docker-compose -f docker-compose.local.yml up -d
This will: 1. Build the Docker image from the provided Dockerfile. 2. Start the container and expose it on http://localhost:11235.

Option 2: Using Docker Compose with Pre-Built Image from Hub
If you prefer using the pre-built image on Docker Hub, use the docker-compose.hub.yml file.

docker-compose -f docker-compose.hub.yml up -d
This will: 1. Pull the pre-built image unclecode/crawl4ai:basic (or all, depending on your configuration). 2. Start the container and expose it on http://localhost:11235.

Stopping the Running Services
To stop the services started via Docker Compose, you can use:

docker-compose -f docker-compose.local.yml down
# OR
docker-compose -f docker-compose.hub.yml down
If the containers don’t stop and the application is still running, check the running containers:

docker ps
Find the CONTAINER ID of the running service and stop it forcefully:

docker stop <CONTAINER_ID>
Debugging with Docker Compose
Check Logs: To view the container logs:

docker-compose -f docker-compose.local.yml logs -f
Remove Orphaned Containers: If the service is still running unexpectedly:

docker-compose -f docker-compose.local.yml down --remove-orphans
Manually Remove Network: If the network is still in use:

docker network ls
docker network rm crawl4ai_default
Why Use Docker Compose?
Docker Compose is the recommended way to deploy Crawl4AI because: 1. It simplifies multi-container setups. 2. Allows you to define environment variables, resources, and ports in a single file. 3. Makes it easier to switch between local development and production-ready images.

For example, your docker-compose.yml could include API keys, token settings, and memory limits, making deployment quick and consistent.

API Security 🔒
Understanding CRAWL4AI_API_TOKEN
The CRAWL4AI_API_TOKEN provides optional security for your Crawl4AI instance:

If CRAWL4AI_API_TOKEN is set: All API endpoints (except /health) require authentication
If CRAWL4AI_API_TOKEN is not set: The API is publicly accessible
# Secured Instance
docker run -p 11235:11235 -e CRAWL4AI_API_TOKEN=your_secret_token unclecode/crawl4ai:all

# Unsecured Instance
docker run -p 11235:11235 unclecode/crawl4ai:all
Making API Calls
For secured instances, include the token in all requests:

import requests

# Setup headers if token is being used
api_token = "your_secret_token"  # Same token set in CRAWL4AI_API_TOKEN
headers = {"Authorization": f"Bearer {api_token}"} if api_token else {}

# Making authenticated requests
response = requests.post(
    "http://localhost:11235/crawl",
    headers=headers,
    json={
        "urls": "https://example.com",
        "priority": 10
    }
)

# Checking task status
task_id = response.json()["task_id"]
status = requests.get(
    f"http://localhost:11235/task/{task_id}",
    headers=headers
)
Using with Docker Compose
In your docker-compose.yml:

services:
  crawl4ai:
    image: unclecode/crawl4ai:all
    environment:
      - CRAWL4AI_API_TOKEN=${CRAWL4AI_API_TOKEN:-}  # Optional
    # ... other configuration
Then either: 1. Set in .env file:

CRAWL4AI_API_TOKEN=your_secret_token
Or set via command line:
CRAWL4AI_API_TOKEN=your_secret_token docker-compose up
Security Note: If you enable the API token, make sure to keep it secure and never commit it to version control. The token will be required for all API endpoints except the health check endpoint (/health).

Configuration Options 🔧
Environment Variables
You can configure the service using environment variables:

# Basic configuration
docker run -p 11235:11235 \
    -e MAX_CONCURRENT_TASKS=5 \
    unclecode/crawl4ai:all

# With security and LLM support
docker run -p 11235:11235 \
    -e CRAWL4AI_API_TOKEN=your_secret_token \
    -e OPENAI_API_KEY=sk-... \
    -e ANTHROPIC_API_KEY=sk-ant-... \
    unclecode/crawl4ai:all
Using Docker Compose (Recommended) 🐳
Create a docker-compose.yml:

version: '3.8'

services:
  crawl4ai:
    image: unclecode/crawl4ai:all
    ports:
      - "11235:11235"
    environment:
      - CRAWL4AI_API_TOKEN=${CRAWL4AI_API_TOKEN:-}  # Optional API security
      - MAX_CONCURRENT_TASKS=5
      # LLM Provider Keys
      - OPENAI_API_KEY=${OPENAI_API_KEY:-}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY:-}
    volumes:
      - /dev/shm:/dev/shm
    deploy:
      resources:
        limits:
          memory: 4G
        reservations:
          memory: 1G
You can run it in two ways:

Using environment variables directly:

CRAWL4AI_API_TOKEN=secret123 OPENAI_API_KEY=sk-... docker-compose up
Using a .env file (recommended): Create a .env file in the same directory:

# API Security (optional)
CRAWL4AI_API_TOKEN=your_secret_token

# LLM Provider Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Other Configuration
MAX_CONCURRENT_TASKS=5
Then simply run:

docker-compose up
Testing the Deployment 🧪
import requests

# For unsecured instances
def test_unsecured():
    # Health check
    health = requests.get("http://localhost:11235/health")
    print("Health check:", health.json())

    # Basic crawl
    response = requests.post(
        "http://localhost:11235/crawl",
        json={
            "urls": "https://www.nbcnews.com/business",
            "priority": 10
        }
    )
    task_id = response.json()["task_id"]
    print("Task ID:", task_id)

# For secured instances
def test_secured(api_token):
    headers = {"Authorization": f"Bearer {api_token}"}

    # Basic crawl with authentication
    response = requests.post(
        "http://localhost:11235/crawl",
        headers=headers,
        json={
            "urls": "https://www.nbcnews.com/business",
            "priority": 10
        }
    )
    task_id = response.json()["task_id"]
    print("Task ID:", task_id)
LLM Extraction Example 🤖
When you've configured your LLM provider keys (via environment variables or .env), you can use LLM extraction:

request = {
    "urls": "https://example.com",
    "extraction_config": {
        "type": "llm",
        "params": {
            "provider": "openai/gpt-4",
            "instruction": "Extract main topics from the page"
        }
    }
}

# Make the request (add headers if using API security)
response = requests.post("http://localhost:11235/crawl", json=request)
Note: Remember to add .env to your .gitignore to keep your API keys secure!

Usage Examples 📝
Basic Crawling
request = {
    "urls": "https://www.nbcnews.com/business",
    "priority": 10
}

response = requests.post("http://localhost:11235/crawl", json=request)
task_id = response.json()["task_id"]

# Get results
result = requests.get(f"http://localhost:11235/task/{task_id}")
Structured Data Extraction
schema = {
    "name": "Crypto Prices",
    "baseSelector": ".cds-tableRow-t45thuk",
    "fields": [
        {
            "name": "crypto",
            "selector": "td:nth-child(1) h2",
            "type": "text",
        },
        {
            "name": "price",
            "selector": "td:nth-child(2)",
            "type": "text",
        }
    ],
}

request = {
    "urls": "https://www.coinbase.com/explore",
    "extraction_config": {
        "type": "json_css",
        "params": {"schema": schema}
    }
}
Dynamic Content Handling
request = {
    "urls": "https://www.nbcnews.com/business",
    "js_code": [
        "const loadMoreButton = Array.from(document.querySelectorAll('button')).find(button => button.textContent.includes('Load More')); loadMoreButton && loadMoreButton.click();"
    ],
    "wait_for": "article.tease-card:nth-child(10)"
}
AI-Powered Extraction (Full Version)
request = {
    "urls": "https://www.nbcnews.com/business",
    "extraction_config": {
        "type": "cosine",
        "params": {
            "semantic_filter": "business finance economy",
            "word_count_threshold": 10,
            "max_dist": 0.2,
            "top_k": 3
        }
    }
}
Platform-Specific Instructions 💻
macOS
docker pull unclecode/crawl4ai:basic
docker run -p 11235:11235 unclecode/crawl4ai:basic
Ubuntu
# Basic version
docker pull unclecode/crawl4ai:basic
docker run -p 11235:11235 unclecode/crawl4ai:basic

# With GPU support
docker pull unclecode/crawl4ai:gpu
docker run --gpus all -p 11235:11235 unclecode/crawl4ai:gpu
Windows (PowerShell)
docker pull unclecode/crawl4ai:basic
docker run -p 11235:11235 unclecode/crawl4ai:basic
Testing 🧪
Save this as test_docker.py:

import requests
import json
import time
import sys

class Crawl4AiTester:
    def __init__(self, base_url: str = "http://localhost:11235"):
        self.base_url = base_url

    def submit_and_wait(self, request_data: dict, timeout: int = 300) -> dict:
        # Submit crawl job
        response = requests.post(f"{self.base_url}/crawl", json=request_data)
        task_id = response.json()["task_id"]
        print(f"Task ID: {task_id}")

        # Poll for result
        start_time = time.time()
        while True:
            if time.time() - start_time > timeout:
                raise TimeoutError(f"Task {task_id} timeout")

            result = requests.get(f"{self.base_url}/task/{task_id}")
            status = result.json()

            if status["status"] == "completed":
                return status

            time.sleep(2)

def test_deployment():
    tester = Crawl4AiTester()

    # Test basic crawl
    request = {
        "urls": "https://www.nbcnews.com/business",
        "priority": 10
    }

    result = tester.submit_and_wait(request)
    print("Basic crawl successful!")
    print(f"Content length: {len(result['result']['markdown'])}")

if __name__ == "__main__":
    test_deployment()
Advanced Configuration ⚙️
Crawler Parameters
The crawler_params field allows you to configure the browser instance and crawling behavior. Here are key parameters you can use:

request = {
    "urls": "https://example.com",
    "crawler_params": {
        # Browser Configuration
        "headless": True,                    # Run in headless mode
        "browser_type": "chromium",          # chromium/firefox/webkit
        "user_agent": "custom-agent",        # Custom user agent
        "proxy": "http://proxy:8080",        # Proxy configuration

        # Performance & Behavior
        "page_timeout": 30000,               # Page load timeout (ms)
        "verbose": True,                     # Enable detailed logging
        "semaphore_count": 5,               # Concurrent request limit

        # Anti-Detection Features
        "simulate_user": True,               # Simulate human behavior
        "magic": True,                       # Advanced anti-detection
        "override_navigator": True,          # Override navigator properties

        # Session Management
        "user_data_dir": "./browser-data",   # Browser profile location
        "use_managed_browser": True,         # Use persistent browser
    }
}
Extra Parameters
The extra field allows passing additional parameters directly to the crawler's arun function:

request = {
    "urls": "https://example.com",
    "extra": {
        "word_count_threshold": 10,          # Min words per block
        "only_text": True,                   # Extract only text
        "bypass_cache": True,                # Force fresh crawl
        "process_iframes": True,             # Include iframe content
    }
}
Complete Examples
1. Advanced News Crawling

request = {
    "urls": "https://www.nbcnews.com/business",
    "crawler_params": {
        "headless": True,
        "page_timeout": 30000,
        "remove_overlay_elements": True      # Remove popups
    },
    "extra": {
        "word_count_threshold": 50,          # Longer content blocks
        "bypass_cache": True                 # Fresh content
    },
    "css_selector": ".article-body"
}
2. Anti-Detection Configuration

request = {
    "urls": "https://example.com",
    "crawler_params": {
        "simulate_user": True,
        "magic": True,
        "override_navigator": True,
        "user_agent": "Mozilla/5.0 ...",
        "headers": {
            "Accept-Language": "en-US,en;q=0.9"
        }
    }
}
3. LLM Extraction with Custom Parameters

request = {
    "urls": "https://openai.com/pricing",
    "extraction_config": {
        "type": "llm",
        "params": {
            "provider": "openai/gpt-4",
            "schema": pricing_schema
        }
    },
    "crawler_params": {
        "verbose": True,
        "page_timeout": 60000
    },
    "extra": {
        "word_count_threshold": 1,
        "only_text": True
    }
}
4. Session-Based Dynamic Content

request = {
    "urls": "https://example.com",
    "crawler_params": {
        "session_id": "dynamic_session",
        "headless": False,
        "page_timeout": 60000
    },
    "js_code": ["window.scrollTo(0, document.body.scrollHeight);"],
    "wait_for": "js:() => document.querySelectorAll('.item').length > 10",
    "extra": {
        "delay_before_return_html": 2.0
    }
}
5. Screenshot with Custom Timing

request = {
    "urls": "https://example.com",
    "screenshot": True,
    "crawler_params": {
        "headless": True,
        "screenshot_wait_for": ".main-content"
    },
    "extra": {
        "delay_before_return_html": 3.0
    }
}
Parameter Reference Table
Category	Parameter	Type	Description
Browser	headless	bool	Run browser in headless mode
Browser	browser_type	str	Browser engine selection
Browser	user_agent	str	Custom user agent string
Network	proxy	str	Proxy server URL
Network	headers	dict	Custom HTTP headers
Timing	page_timeout	int	Page load timeout (ms)
Timing	delay_before_return_html	float	Wait before capture
Anti-Detection	simulate_user	bool	Human behavior simulation
Anti-Detection	magic	bool	Advanced protection
Session	session_id	str	Browser session ID
Session	user_data_dir	str	Profile directory
Content	word_count_threshold	int	Minimum words per block
Content	only_text	bool	Text-only extraction
Content	process_iframes	bool	Include iframe content
Debug	verbose	bool	Detailed logging
Debug	log_console	bool	Browser console logs
Troubleshooting 🔍
Common Issues
1. Connection Refused

Error: Connection refused at localhost:11235
Solution: Ensure the container is running and ports are properly mapped.
2. Resource Limits

Error: No available slots
Solution: Increase MAX_CONCURRENT_TASKS or container resources.
3. GPU Access

Error: GPU not found
Solution: Ensure proper NVIDIA drivers and use --gpus all flag.
Debug Mode
Access container for debugging:

docker run -it --entrypoint /bin/bash unclecode/crawl4ai:all
View container logs:

docker logs [container_id]
Best Practices 🌟
1. Resource Management - Set appropriate memory and CPU limits - Monitor resource usage via health endpoint - Use basic version for simple crawling tasks

2. Scaling - Use multiple containers for high load - Implement proper load balancing - Monitor performance metrics

3. Security - Use environment variables for sensitive data - Implement proper network isolation - Regular security updates

API Reference 📚
Health Check
GET /health
Submit Crawl Task
POST /crawl
Content-Type: application/json

{
    "urls": "string or array",
    "extraction_config": {
        "type": "basic|llm|cosine|json_css",
        "params": {}
    },
    "priority": 1-10,
    "ttl": 3600
}
Get Task Status
GET /task/{task_id}
-----------------
Getting Started with Crawl4AI
Welcome to Crawl4AI, an open-source LLM-friendly Web Crawler & Scraper. In this tutorial, you’ll:

Run your first crawl using minimal configuration.
Generate Markdown output (and learn how it’s influenced by content filters).
Experiment with a simple CSS-based extraction strategy.
See a glimpse of LLM-based extraction (including open-source and closed-source model options).
Crawl a dynamic page that loads content via JavaScript.
1. Introduction
Crawl4AI provides:

An asynchronous crawler, AsyncWebCrawler.
Configurable browser and run settings via BrowserConfig and CrawlerRunConfig.
Automatic HTML-to-Markdown conversion via DefaultMarkdownGenerator (supports optional filters).
Multiple extraction strategies (LLM-based or “traditional” CSS/XPath-based).
By the end of this guide, you’ll have performed a basic crawl, generated Markdown, tried out two extraction strategies, and crawled a dynamic page that uses “Load More” buttons or JavaScript updates.

2. Your First Crawl
Here’s a minimal Python script that creates an AsyncWebCrawler, fetches a webpage, and prints the first 300 characters of its Markdown output:

import asyncio
from crawl4ai import AsyncWebCrawler

async def main():
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://example.com")
        print(result.markdown[:300])  # Print first 300 chars

if __name__ == "__main__":
    asyncio.run(main())
What’s happening? - AsyncWebCrawler launches a headless browser (Chromium by default). - It fetches https://example.com. - Crawl4AI automatically converts the HTML into Markdown.

You now have a simple, working crawl!

3. Basic Configuration (Light Introduction)
Crawl4AI’s crawler can be heavily customized using two main classes:

1. BrowserConfig: Controls browser behavior (headless or full UI, user agent, JavaScript toggles, etc.).
2. CrawlerRunConfig: Controls how each crawl runs (caching, extraction, timeouts, hooking, etc.).

Below is an example with minimal usage:

import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode

async def main():
    browser_conf = BrowserConfig(headless=True)  # or False to see the browser
    run_conf = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS
    )

    async with AsyncWebCrawler(config=browser_conf) as crawler:
        result = await crawler.arun(
            url="https://example.com",
            config=run_conf
        )
        print(result.markdown)

if __name__ == "__main__":
    asyncio.run(main())
IMPORTANT: By default cache mode is set to CacheMode.ENABLED. So to have fresh content, you need to set it to CacheMode.BYPASS

We’ll explore more advanced config in later tutorials (like enabling proxies, PDF output, multi-tab sessions, etc.). For now, just note how you pass these objects to manage crawling.

4. Generating Markdown Output
By default, Crawl4AI automatically generates Markdown from each crawled page. However, the exact output depends on whether you specify a markdown generator or content filter.

result.markdown:
The direct HTML-to-Markdown conversion.
result.markdown.fit_markdown:
The same content after applying any configured content filter (e.g., PruningContentFilter).
Example: Using a Filter with DefaultMarkdownGenerator
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.content_filter_strategy import PruningContentFilter
from crawl4ai.markdown_generation_strategy import DefaultMarkdownGenerator

md_generator = DefaultMarkdownGenerator(
    content_filter=PruningContentFilter(threshold=0.4, threshold_type="fixed")
)

config = CrawlerRunConfig(
    cache_mode=CacheMode.BYPASS,
    markdown_generator=md_generator
)

async with AsyncWebCrawler() as crawler:
    result = await crawler.arun("https://news.ycombinator.com", config=config)
    print("Raw Markdown length:", len(result.markdown.raw_markdown))
    print("Fit Markdown length:", len(result.markdown.fit_markdown))
Note: If you do not specify a content filter or markdown generator, you’ll typically see only the raw Markdown. PruningContentFilter may adds around 50ms in processing time. We’ll dive deeper into these strategies in a dedicated Markdown Generation tutorial.

5. Simple Data Extraction (CSS-based)
Crawl4AI can also extract structured data (JSON) using CSS or XPath selectors. Below is a minimal CSS-based example:

New! Crawl4AI now provides a powerful utility to automatically generate extraction schemas using LLM. This is a one-time cost that gives you a reusable schema for fast, LLM-free extractions:

from crawl4ai.extraction_strategy import JsonCssExtractionStrategy
from crawl4ai import LLMConfig

# Generate a schema (one-time cost)
html = "<div class='product'><h2>Gaming Laptop</h2><span class='price'>$999.99</span></div>"

# Using OpenAI (requires API token)
schema = JsonCssExtractionStrategy.generate_schema(
    html,
    llm_config = LLMConfig(provider="openai/gpt-4o",api_token="your-openai-token")  # Required for OpenAI
)

# Or using Ollama (open source, no token needed)
schema = JsonCssExtractionStrategy.generate_schema(
    html,
    llm_config = LLMConfig(provider="ollama/llama3.3", api_token=None)  # Not needed for Ollama
)

# Use the schema for fast, repeated extractions
strategy = JsonCssExtractionStrategy(schema)
For a complete guide on schema generation and advanced usage, see No-LLM Extraction Strategies.

Here's a basic extraction example:

import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def main():
    schema = {
        "name": "Example Items",
        "baseSelector": "div.item",
        "fields": [
            {"name": "title", "selector": "h2", "type": "text"},
            {"name": "link", "selector": "a", "type": "attribute", "attribute": "href"}
        ]
    }

    raw_html = "<div class='item'><h2>Item 1</h2><a href='https://example.com/item1'>Link 1</a></div>"

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="raw://" + raw_html,
            config=CrawlerRunConfig(
                cache_mode=CacheMode.BYPASS,
                extraction_strategy=JsonCssExtractionStrategy(schema)
            )
        )
        # The JSON output is stored in 'extracted_content'
        data = json.loads(result.extracted_content)
        print(data)

if __name__ == "__main__":
    asyncio.run(main())
Why is this helpful? - Great for repetitive page structures (e.g., item listings, articles). - No AI usage or costs. - The crawler returns a JSON string you can parse or store.

Tips: You can pass raw HTML to the crawler instead of a URL. To do so, prefix the HTML with raw://.

6. Simple Data Extraction (LLM-based)
For more complex or irregular pages, a language model can parse text intelligently into a structure you define. Crawl4AI supports open-source or closed-source providers:

Open-Source Models (e.g., ollama/llama3.3, no_token)
OpenAI Models (e.g., openai/gpt-4, requires api_token)
Or any provider supported by the underlying library
Below is an example using open-source style (no token) and closed-source:

import os
import json
import asyncio
from pydantic import BaseModel, Field
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy

class OpenAIModelFee(BaseModel):
    model_name: str = Field(..., description="Name of the OpenAI model.")
    input_fee: str = Field(..., description="Fee for input token for the OpenAI model.")
    output_fee: str = Field(
        ..., description="Fee for output token for the OpenAI model."
    )

async def extract_structured_data_using_llm(
    provider: str, api_token: str = None, extra_headers: Dict[str, str] = None
):
    print(f"\n--- Extracting Structured Data with {provider} ---")

    if api_token is None and provider != "ollama":
        print(f"API token is required for {provider}. Skipping this example.")
        return

    browser_config = BrowserConfig(headless=True)

    extra_args = {"temperature": 0, "top_p": 0.9, "max_tokens": 2000}
    if extra_headers:
        extra_args["extra_headers"] = extra_headers

    crawler_config = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS,
        word_count_threshold=1,
        page_timeout=80000,
        extraction_strategy=LLMExtractionStrategy(
            llm_config = LLMConfig(provider=provider,api_token=api_token),
            schema=OpenAIModelFee.model_json_schema(),
            extraction_type="schema",
            instruction="""From the crawled content, extract all mentioned model names along with their fees for input and output tokens. 
            Do not miss any models in the entire content.""",
            extra_args=extra_args,
        ),
    )

    async with AsyncWebCrawler(config=browser_config) as crawler:
        result = await crawler.arun(
            url="https://openai.com/api/pricing/", config=crawler_config
        )
        print(result.extracted_content)

if __name__ == "__main__":

    asyncio.run(
        extract_structured_data_using_llm(
            provider="openai/gpt-4o", api_token=os.getenv("OPENAI_API_KEY")
        )
    )
What’s happening? - We define a Pydantic schema (PricingInfo) describing the fields we want. - The LLM extraction strategy uses that schema and your instructions to transform raw text into structured JSON. - Depending on the provider and api_token, you can use local models or a remote API.

7. Multi-URL Concurrency (Preview)
If you need to crawl multiple URLs in parallel, you can use arun_many(). By default, Crawl4AI employs a MemoryAdaptiveDispatcher, automatically adjusting concurrency based on system resources. Here’s a quick glimpse:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode

async def quick_parallel_example():
    urls = [
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3"
    ]

    run_conf = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS,
        stream=True  # Enable streaming mode
    )

    async with AsyncWebCrawler() as crawler:
        # Stream results as they complete
        async for result in await crawler.arun_many(urls, config=run_conf):
            if result.success:
                print(f"[OK] {result.url}, length: {len(result.markdown.raw_markdown)}")
            else:
                print(f"[ERROR] {result.url} => {result.error_message}")

        # Or get all results at once (default behavior)
        run_conf = run_conf.clone(stream=False)
        results = await crawler.arun_many(urls, config=run_conf)
        for res in results:
            if res.success:
                print(f"[OK] {res.url}, length: {len(res.markdown.raw_markdown)}")
            else:
                print(f"[ERROR] {res.url} => {res.error_message}")

if __name__ == "__main__":
    asyncio.run(quick_parallel_example())
The example above shows two ways to handle multiple URLs: 1. Streaming mode (stream=True): Process results as they become available using async for 2. Batch mode (stream=False): Wait for all results to complete

For more advanced concurrency (e.g., a semaphore-based approach, adaptive memory usage throttling, or customized rate limiting), see Advanced Multi-URL Crawling.

8. Dynamic Content Example
Some sites require multiple “page clicks” or dynamic JavaScript updates. Below is an example showing how to click a “Next Page” button and wait for new commits to load on GitHub, using BrowserConfig and CrawlerRunConfig:

import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def extract_structured_data_using_css_extractor():
    print("\n--- Using JsonCssExtractionStrategy for Fast Structured Output ---")
    schema = {
        "name": "KidoCode Courses",
        "baseSelector": "section.charge-methodology .w-tab-content > div",
        "fields": [
            {
                "name": "section_title",
                "selector": "h3.heading-50",
                "type": "text",
            },
            {
                "name": "section_description",
                "selector": ".charge-content",
                "type": "text",
            },
            {
                "name": "course_name",
                "selector": ".text-block-93",
                "type": "text",
            },
            {
                "name": "course_description",
                "selector": ".course-content-text",
                "type": "text",
            },
            {
                "name": "course_icon",
                "selector": ".image-92",
                "type": "attribute",
                "attribute": "src",
            },
        ],
    }

    browser_config = BrowserConfig(headless=True, java_script_enabled=True)

    js_click_tabs = """
    (async () => {
        const tabs = document.querySelectorAll("section.charge-methodology .tabs-menu-3 > div");
        for(let tab of tabs) {
            tab.scrollIntoView();
            tab.click();
            await new Promise(r => setTimeout(r, 500));
        }
    })();
    """

    crawler_config = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS,
        extraction_strategy=JsonCssExtractionStrategy(schema),
        js_code=[js_click_tabs],
    )

    async with AsyncWebCrawler(config=browser_config) as crawler:
        result = await crawler.arun(
            url="https://www.kidocode.com/degrees/technology", config=crawler_config
        )

        companies = json.loads(result.extracted_content)
        print(f"Successfully extracted {len(companies)} companies")
        print(json.dumps(companies[0], indent=2))

async def main():
    await extract_structured_data_using_css_extractor()

if __name__ == "__main__":
    asyncio.run(main())
Key Points:

BrowserConfig(headless=False): We want to watch it click “Next Page.”
CrawlerRunConfig(...): We specify the extraction strategy, pass session_id to reuse the same page.
js_code and wait_for are used for subsequent pages (page > 0) to click the “Next” button and wait for new commits to load.
js_only=True indicates we’re not re-navigating but continuing the existing session.
Finally, we call kill_session() to clean up the page and browser session.
9. Next Steps
Congratulations! You have:

Performed a basic crawl and printed Markdown.
Used content filters with a markdown generator.
Extracted JSON via CSS or LLM strategies.
Handled dynamic pages with JavaScript triggers.
If you’re ready for more, check out:

Installation: A deeper dive into advanced installs, Docker usage (experimental), or optional dependencies.
Hooks & Auth: Learn how to run custom JavaScript or handle logins with cookies, local storage, etc.
Deployment: Explore ephemeral testing in Docker or plan for the upcoming stable Docker release.
Browser Management: Delve into user simulation, stealth modes, and concurrency best practices.
Crawl4AI is a powerful, flexible tool. Enjoy building out your scrapers, data pipelines, or AI-driven extraction flows. Happy crawling!


-------------------

Crawl4AI CLI Guide
Table of Contents
Installation
Basic Usage
Configuration
Browser Configuration
Crawler Configuration
Extraction Configuration
Content Filtering
Advanced Features
LLM Q&A
Structured Data Extraction
Content Filtering
Output Formats
Examples
Configuration Reference
Best Practices & Tips
Basic Usage
The Crawl4AI CLI (crwl) provides a simple interface to the Crawl4AI library:

# Basic crawling
crwl https://example.com

# Get markdown output
crwl https://example.com -o markdown

# Verbose JSON output with cache bypass
crwl https://example.com -o json -v --bypass-cache

# See usage examples
crwl --example
Quick Example of Advanced Usage
If you clone the repository and run the following command, you will receive the content of the page in JSON format according to a JSON-CSS schema:

crwl "https://www.infoq.com/ai-ml-data-eng/" -e docs/examples/cli/extract_css.yml -s docs/examples/cli/css_schema.json -o json;
Configuration
Browser Configuration
Browser settings can be configured via YAML file or command line parameters:

# browser.yml
headless: true
viewport_width: 1280
user_agent_mode: "random"
verbose: true
ignore_https_errors: true
# Using config file
crwl https://example.com -B browser.yml

# Using direct parameters
crwl https://example.com -b "headless=true,viewport_width=1280,user_agent_mode=random"
Crawler Configuration
Control crawling behavior:

# crawler.yml
cache_mode: "bypass"
wait_until: "networkidle"
page_timeout: 30000
delay_before_return_html: 0.5
word_count_threshold: 100
scan_full_page: true
scroll_delay: 0.3
process_iframes: false
remove_overlay_elements: true
magic: true
verbose: true
# Using config file
crwl https://example.com -C crawler.yml

# Using direct parameters
crwl https://example.com -c "css_selector=#main,delay_before_return_html=2,scan_full_page=true"
Extraction Configuration
Two types of extraction are supported:

CSS/XPath-based extraction:
# extract_css.yml
type: "json-css"
params:
  verbose: true
// css_schema.json
{
  "name": "ArticleExtractor",
  "baseSelector": ".article",
  "fields": [
    {
      "name": "title",
      "selector": "h1.title",
      "type": "text"
    },
    {
      "name": "link",
      "selector": "a.read-more",
      "type": "attribute",
      "attribute": "href"
    }
  ]
}
LLM-based extraction:
# extract_llm.yml
type: "llm"
provider: "openai/gpt-4"
instruction: "Extract all articles with their titles and links"
api_token: "your-token"
params:
  temperature: 0.3
  max_tokens: 1000
// llm_schema.json
{
  "title": "Article",
  "type": "object",
  "properties": {
    "title": {
      "type": "string",
      "description": "The title of the article"
    },
    "link": {
      "type": "string",
      "description": "URL to the full article"
    }
  }
}
Advanced Features
LLM Q&A
Ask questions about crawled content:

# Simple question
crwl https://example.com -q "What is the main topic discussed?"

# View content then ask questions
crwl https://example.com -o markdown  # See content first
crwl https://example.com -q "Summarize the key points"
crwl https://example.com -q "What are the conclusions?"

# Combined with advanced crawling
crwl https://example.com \
    -B browser.yml \
    -c "css_selector=article,scan_full_page=true" \
    -q "What are the pros and cons mentioned?"
First-time setup: - Prompts for LLM provider and API token - Saves configuration in ~/.crawl4ai/global.yml - Supports various providers (openai/gpt-4, anthropic/claude-3-sonnet, etc.) - For case of ollama you do not need to provide API token. - See LiteLLM Providers for full list

Structured Data Extraction
Extract structured data using CSS selectors:

crwl https://example.com \
    -e extract_css.yml \
    -s css_schema.json \
    -o json
Or using LLM-based extraction:

crwl https://example.com \
    -e extract_llm.yml \
    -s llm_schema.json \
    -o json
Content Filtering
Filter content for relevance:

# filter_bm25.yml
type: "bm25"
query: "target content"
threshold: 1.0

# filter_pruning.yml
type: "pruning"
query: "focus topic"
threshold: 0.48
crwl https://example.com -f filter_bm25.yml -o markdown-fit
Output Formats
all - Full crawl result including metadata
json - Extracted structured data (when using extraction)
markdown / md - Raw markdown output
markdown-fit / md-fit - Filtered markdown for better readability
Complete Examples
Basic Extraction:

crwl https://example.com \
    -B browser.yml \
    -C crawler.yml \
    -o json
Structured Data Extraction:

crwl https://example.com \
    -e extract_css.yml \
    -s css_schema.json \
    -o json \
    -v
LLM Extraction with Filtering:

crwl https://example.com \
    -B browser.yml \
    -e extract_llm.yml \
    -s llm_schema.json \
    -f filter_bm25.yml \
    -o json
Interactive Q&A:

# First crawl and view
crwl https://example.com -o markdown

# Then ask questions
crwl https://example.com -q "What are the main points?"
crwl https://example.com -q "Summarize the conclusions"
Best Practices & Tips
Configuration Management:
Keep common configurations in YAML files
Use CLI parameters for quick overrides
Store sensitive data (API tokens) in ~/.crawl4ai/global.yml

Performance Optimization:

Use --bypass-cache for fresh content
Enable scan_full_page for infinite scroll pages
Adjust delay_before_return_html for dynamic content

Content Extraction:

Use CSS extraction for structured content
Use LLM extraction for unstructured content
Combine with filters for focused results

Q&A Workflow:

View content first with -o markdown
Ask specific questions
Use broader context with appropriate selectors
Recap
The Crawl4AI CLI provides: - Flexible configuration via files and parameters - Multiple extraction strategies (CSS, XPath, LLM) - Content filtering and optimization - Interactive Q&A capabilities - Various output formats


-------------
Deep Crawling
One of Crawl4AI's most powerful features is its ability to perform configurable deep crawling that can explore websites beyond a single page. With fine-tuned control over crawl depth, domain boundaries, and content filtering, Crawl4AI gives you the tools to extract precisely the content you need.

In this tutorial, you'll learn:

How to set up a Basic Deep Crawler with BFS strategy
Understanding the difference between streamed and non-streamed output
Implementing filters and scorers to target specific content
Creating advanced filtering chains for sophisticated crawls
Using BestFirstCrawling for intelligent exploration prioritization
Prerequisites
- You’ve completed or read AsyncWebCrawler Basics to understand how to run a simple crawl.
- You know how to configure CrawlerRunConfig.

1. Quick Example
Here's a minimal code snippet that implements a basic deep crawl using the BFSDeepCrawlStrategy:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.deep_crawling import BFSDeepCrawlStrategy
from crawl4ai.content_scraping_strategy import LXMLWebScrapingStrategy

async def main():
    # Configure a 2-level deep crawl
    config = CrawlerRunConfig(
        deep_crawl_strategy=BFSDeepCrawlStrategy(
            max_depth=2, 
            include_external=False
        ),
        scraping_strategy=LXMLWebScrapingStrategy(),
        verbose=True
    )

    async with AsyncWebCrawler() as crawler:
        results = await crawler.arun("https://example.com", config=config)

        print(f"Crawled {len(results)} pages in total")

        # Access individual results
        for result in results[:3]:  # Show first 3 results
            print(f"URL: {result.url}")
            print(f"Depth: {result.metadata.get('depth', 0)}")

if __name__ == "__main__":
    asyncio.run(main())
What's happening?
- BFSDeepCrawlStrategy(max_depth=2, include_external=False) instructs Crawl4AI to: - Crawl the starting page (depth 0) plus 2 more levels - Stay within the same domain (don't follow external links) - Each result contains metadata like the crawl depth - Results are returned as a list after all crawling is complete

2. Understanding Deep Crawling Strategy Options
2.1 BFSDeepCrawlStrategy (Breadth-First Search)
The BFSDeepCrawlStrategy uses a breadth-first approach, exploring all links at one depth before moving deeper:

from crawl4ai.deep_crawling import BFSDeepCrawlStrategy

# Basic configuration
strategy = BFSDeepCrawlStrategy(
    max_depth=2,               # Crawl initial page + 2 levels deep
    include_external=False,    # Stay within the same domain
    max_pages=50,              # Maximum number of pages to crawl (optional)
    score_threshold=0.3,       # Minimum score for URLs to be crawled (optional)
)
Key parameters: - max_depth: Number of levels to crawl beyond the starting page - include_external: Whether to follow links to other domains - max_pages: Maximum number of pages to crawl (default: infinite) - score_threshold: Minimum score for URLs to be crawled (default: -inf) - filter_chain: FilterChain instance for URL filtering - url_scorer: Scorer instance for evaluating URLs

2.2 DFSDeepCrawlStrategy (Depth-First Search)
The DFSDeepCrawlStrategy uses a depth-first approach, explores as far down a branch as possible before backtracking.

from crawl4ai.deep_crawling import DFSDeepCrawlStrategy

# Basic configuration
strategy = DFSDeepCrawlStrategy(
    max_depth=2,               # Crawl initial page + 2 levels deep
    include_external=False,    # Stay within the same domain
    max_pages=30,              # Maximum number of pages to crawl (optional)
    score_threshold=0.5,       # Minimum score for URLs to be crawled (optional)
)
Key parameters: - max_depth: Number of levels to crawl beyond the starting page - include_external: Whether to follow links to other domains - max_pages: Maximum number of pages to crawl (default: infinite) - score_threshold: Minimum score for URLs to be crawled (default: -inf) - filter_chain: FilterChain instance for URL filtering - url_scorer: Scorer instance for evaluating URLs

2.3 BestFirstCrawlingStrategy (⭐️ - Recommended Deep crawl strategy)
For more intelligent crawling, use BestFirstCrawlingStrategy with scorers to prioritize the most relevant pages:

from crawl4ai.deep_crawling import BestFirstCrawlingStrategy
from crawl4ai.deep_crawling.scorers import KeywordRelevanceScorer

# Create a scorer
scorer = KeywordRelevanceScorer(
    keywords=["crawl", "example", "async", "configuration"],
    weight=0.7
)

# Configure the strategy
strategy = BestFirstCrawlingStrategy(
    max_depth=2,
    include_external=False,
    url_scorer=scorer,
    max_pages=25,              # Maximum number of pages to crawl (optional)
)
This crawling approach: - Evaluates each discovered URL based on scorer criteria - Visits higher-scoring pages first - Helps focus crawl resources on the most relevant content - Can limit total pages crawled with max_pages - Does not need score_threshold as it naturally prioritizes by score

3. Streaming vs. Non-Streaming Results
Crawl4AI can return results in two modes:

3.1 Non-Streaming Mode (Default)
config = CrawlerRunConfig(
    deep_crawl_strategy=BFSDeepCrawlStrategy(max_depth=1),
    stream=False  # Default behavior
)

async with AsyncWebCrawler() as crawler:
    # Wait for ALL results to be collected before returning
    results = await crawler.arun("https://example.com", config=config)

    for result in results:
        process_result(result)
When to use non-streaming mode: - You need the complete dataset before processing - You're performing batch operations on all results together - Crawl time isn't a critical factor

3.2 Streaming Mode
config = CrawlerRunConfig(
    deep_crawl_strategy=BFSDeepCrawlStrategy(max_depth=1),
    stream=True  # Enable streaming
)

async with AsyncWebCrawler() as crawler:
    # Returns an async iterator
    async for result in await crawler.arun("https://example.com", config=config):
        # Process each result as it becomes available
        process_result(result)
Benefits of streaming mode: - Process results immediately as they're discovered - Start working with early results while crawling continues - Better for real-time applications or progressive display - Reduces memory pressure when handling many pages

4. Filtering Content with Filter Chains
Filters help you narrow down which pages to crawl. Combine multiple filters using FilterChain for powerful targeting.

4.1 Basic URL Pattern Filter
from crawl4ai.deep_crawling.filters import FilterChain, URLPatternFilter

# Only follow URLs containing "blog" or "docs"
url_filter = URLPatternFilter(patterns=["*blog*", "*docs*"])

config = CrawlerRunConfig(
    deep_crawl_strategy=BFSDeepCrawlStrategy(
        max_depth=1,
        filter_chain=FilterChain([url_filter])
    )
)
4.2 Combining Multiple Filters
from crawl4ai.deep_crawling.filters import (
    FilterChain,
    URLPatternFilter,
    DomainFilter,
    ContentTypeFilter
)

# Create a chain of filters
filter_chain = FilterChain([
    # Only follow URLs with specific patterns
    URLPatternFilter(patterns=["*guide*", "*tutorial*"]),

    # Only crawl specific domains
    DomainFilter(
        allowed_domains=["docs.example.com"],
        blocked_domains=["old.docs.example.com"]
    ),

    # Only include specific content types
    ContentTypeFilter(allowed_types=["text/html"])
])

config = CrawlerRunConfig(
    deep_crawl_strategy=BFSDeepCrawlStrategy(
        max_depth=2,
        filter_chain=filter_chain
    )
)
4.3 Available Filter Types
Crawl4AI includes several specialized filters:

URLPatternFilter: Matches URL patterns using wildcard syntax
DomainFilter: Controls which domains to include or exclude
ContentTypeFilter: Filters based on HTTP Content-Type
ContentRelevanceFilter: Uses similarity to a text query
SEOFilter: Evaluates SEO elements (meta tags, headers, etc.)
5. Using Scorers for Prioritized Crawling
Scorers assign priority values to discovered URLs, helping the crawler focus on the most relevant content first.

5.1 KeywordRelevanceScorer
from crawl4ai.deep_crawling.scorers import KeywordRelevanceScorer
from crawl4ai.deep_crawling import BestFirstCrawlingStrategy

# Create a keyword relevance scorer
keyword_scorer = KeywordRelevanceScorer(
    keywords=["crawl", "example", "async", "configuration"],
    weight=0.7  # Importance of this scorer (0.0 to 1.0)
)

config = CrawlerRunConfig(
    deep_crawl_strategy=BestFirstCrawlingStrategy(
        max_depth=2,
        url_scorer=keyword_scorer
    ),
    stream=True  # Recommended with BestFirstCrawling
)

# Results will come in order of relevance score
async with AsyncWebCrawler() as crawler:
    async for result in await crawler.arun("https://example.com", config=config):
        score = result.metadata.get("score", 0)
        print(f"Score: {score:.2f} | {result.url}")
How scorers work: - Evaluate each discovered URL before crawling - Calculate relevance based on various signals - Help the crawler make intelligent choices about traversal order

6. Advanced Filtering Techniques
6.1 SEO Filter for Quality Assessment
The SEOFilter helps you identify pages with strong SEO characteristics:

from crawl4ai.deep_crawling.filters import FilterChain, SEOFilter

# Create an SEO filter that looks for specific keywords in page metadata
seo_filter = SEOFilter(
    threshold=0.5,  # Minimum score (0.0 to 1.0)
    keywords=["tutorial", "guide", "documentation"]
)

config = CrawlerRunConfig(
    deep_crawl_strategy=BFSDeepCrawlStrategy(
        max_depth=1,
        filter_chain=FilterChain([seo_filter])
    )
)
6.2 Content Relevance Filter
The ContentRelevanceFilter analyzes the actual content of pages:

from crawl4ai.deep_crawling.filters import FilterChain, ContentRelevanceFilter

# Create a content relevance filter
relevance_filter = ContentRelevanceFilter(
    query="Web crawling and data extraction with Python",
    threshold=0.7  # Minimum similarity score (0.0 to 1.0)
)

config = CrawlerRunConfig(
    deep_crawl_strategy=BFSDeepCrawlStrategy(
        max_depth=1,
        filter_chain=FilterChain([relevance_filter])
    )
)
This filter: - Measures semantic similarity between query and page content - It's a BM25-based relevance filter using head section content

7. Building a Complete Advanced Crawler
This example combines multiple techniques for a sophisticated crawl:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.content_scraping_strategy import LXMLWebScrapingStrategy
from crawl4ai.deep_crawling import BestFirstCrawlingStrategy
from crawl4ai.deep_crawling.filters import (
    FilterChain,
    DomainFilter,
    URLPatternFilter,
    ContentTypeFilter
)
from crawl4ai.deep_crawling.scorers import KeywordRelevanceScorer

async def run_advanced_crawler():
    # Create a sophisticated filter chain
    filter_chain = FilterChain([
        # Domain boundaries
        DomainFilter(
            allowed_domains=["docs.example.com"],
            blocked_domains=["old.docs.example.com"]
        ),

        # URL patterns to include
        URLPatternFilter(patterns=["*guide*", "*tutorial*", "*blog*"]),

        # Content type filtering
        ContentTypeFilter(allowed_types=["text/html"])
    ])

    # Create a relevance scorer
    keyword_scorer = KeywordRelevanceScorer(
        keywords=["crawl", "example", "async", "configuration"],
        weight=0.7
    )

    # Set up the configuration
    config = CrawlerRunConfig(
        deep_crawl_strategy=BestFirstCrawlingStrategy(
            max_depth=2,
            include_external=False,
            filter_chain=filter_chain,
            url_scorer=keyword_scorer
        ),
        scraping_strategy=LXMLWebScrapingStrategy(),
        stream=True,
        verbose=True
    )

    # Execute the crawl
    results = []
    async with AsyncWebCrawler() as crawler:
        async for result in await crawler.arun("https://docs.example.com", config=config):
            results.append(result)
            score = result.metadata.get("score", 0)
            depth = result.metadata.get("depth", 0)
            print(f"Depth: {depth} | Score: {score:.2f} | {result.url}")

    # Analyze the results
    print(f"Crawled {len(results)} high-value pages")
    print(f"Average score: {sum(r.metadata.get('score', 0) for r in results) / len(results):.2f}")

    # Group by depth
    depth_counts = {}
    for result in results:
        depth = result.metadata.get("depth", 0)
        depth_counts[depth] = depth_counts.get(depth, 0) + 1

    print("Pages crawled by depth:")
    for depth, count in sorted(depth_counts.items()):
        print(f"  Depth {depth}: {count} pages")

if __name__ == "__main__":
    asyncio.run(run_advanced_crawler())
8. Limiting and Controlling Crawl Size
8.1 Using max_pages
You can limit the total number of pages crawled with the max_pages parameter:

# Limit to exactly 20 pages regardless of depth
strategy = BFSDeepCrawlStrategy(
    max_depth=3,
    max_pages=20
)
This feature is useful for: - Controlling API costs - Setting predictable execution times - Focusing on the most important content - Testing crawl configurations before full execution

8.2 Using score_threshold
For BFS and DFS strategies, you can set a minimum score threshold to only crawl high-quality pages:

# Only follow links with scores above 0.4
strategy = DFSDeepCrawlStrategy(
    max_depth=2,
    url_scorer=KeywordRelevanceScorer(keywords=["api", "guide", "reference"]),
    score_threshold=0.4  # Skip URLs with scores below this value
)
Note that for BestFirstCrawlingStrategy, score_threshold is not needed since pages are already processed in order of highest score first.

9. Common Pitfalls & Tips
1.Set realistic limits. Be cautious with max_depth values > 3, which can exponentially increase crawl size. Use max_pages to set hard limits.

2.Don't neglect the scoring component. BestFirstCrawling works best with well-tuned scorers. Experiment with keyword weights for optimal prioritization.

3.Be a good web citizen. Respect robots.txt. (disabled by default)

4.Handle page errors gracefully. Not all pages will be accessible. Check result.status when processing results.

5.Balance breadth vs. depth. Choose your strategy wisely - BFS for comprehensive coverage, DFS for deep exploration, BestFirst for focused relevance-based crawling.

10. Summary & Next Steps
In this Deep Crawling with Crawl4AI tutorial, you learned to:

Configure BFSDeepCrawlStrategy, DFSDeepCrawlStrategy, and BestFirstCrawlingStrategy
Process results in streaming or non-streaming mode
Apply filters to target specific content
Use scorers to prioritize the most relevant pages
Limit crawls with max_pages and score_threshold parameters
Build a complete advanced crawler with combined techniques
With these tools, you can efficiently extract structured data from websites at scale, focusing precisely on the content you need for your specific use case.


----------------------
Crawl Result and Output
When you call arun() on a page, Crawl4AI returns a CrawlResult object containing everything you might need—raw HTML, a cleaned version, optional screenshots or PDFs, structured extraction results, and more. This document explains those fields and how they map to different output types.

1. The CrawlResult Model
Below is the core schema. Each field captures a different aspect of the crawl’s result:

class MarkdownGenerationResult(BaseModel):
    raw_markdown: str
    markdown_with_citations: str
    references_markdown: str
    fit_markdown: Optional[str] = None
    fit_html: Optional[str] = None

class CrawlResult(BaseModel):
    url: str
    html: str
    success: bool
    cleaned_html: Optional[str] = None
    media: Dict[str, List[Dict]] = {}
    links: Dict[str, List[Dict]] = {}
    downloaded_files: Optional[List[str]] = None
    screenshot: Optional[str] = None
    pdf : Optional[bytes] = None
    markdown: Optional[Union[str, MarkdownGenerationResult]] = None
    extracted_content: Optional[str] = None
    metadata: Optional[dict] = None
    error_message: Optional[str] = None
    session_id: Optional[str] = None
    response_headers: Optional[dict] = None
    status_code: Optional[int] = None
    ssl_certificate: Optional[SSLCertificate] = None
    class Config:
        arbitrary_types_allowed = True
Table: Key Fields in CrawlResult
Field (Name & Type)	Description
url (str)	The final or actual URL crawled (in case of redirects).
html (str)	Original, unmodified page HTML. Good for debugging or custom processing.
success (bool)	True if the crawl completed without major errors, else False.
cleaned_html (Optional[str])	Sanitized HTML with scripts/styles removed; can exclude tags if configured via excluded_tags etc.
media (Dict[str, List[Dict]])	Extracted media info (images, audio, etc.), each with attributes like src, alt, score, etc.
links (Dict[str, List[Dict]])	Extracted link data, split by internal and external. Each link usually has href, text, etc.
downloaded_files (Optional[List[str]])	If accept_downloads=True in BrowserConfig, this lists the filepaths of saved downloads.
screenshot (Optional[str])	Screenshot of the page (base64-encoded) if screenshot=True.
pdf (Optional[bytes])	PDF of the page if pdf=True.
markdown (Optional[str or MarkdownGenerationResult])	It holds a MarkdownGenerationResult. Over time, this will be consolidated into markdown. The generator can provide raw markdown, citations, references, and optionally fit_markdown.
extracted_content (Optional[str])	The output of a structured extraction (CSS/LLM-based) stored as JSON string or other text.
metadata (Optional[dict])	Additional info about the crawl or extracted data.
error_message (Optional[str])	If success=False, contains a short description of what went wrong.
session_id (Optional[str])	The ID of the session used for multi-page or persistent crawling.
response_headers (Optional[dict])	HTTP response headers, if captured.
status_code (Optional[int])	HTTP status code (e.g., 200 for OK).
ssl_certificate (Optional[SSLCertificate])	SSL certificate info if fetch_ssl_certificate=True.
2. HTML Variants
html: Raw HTML
Crawl4AI preserves the exact HTML as result.html. Useful for:

Debugging page issues or checking the original content.
Performing your own specialized parse if needed.
cleaned_html: Sanitized
If you specify any cleanup or exclusion parameters in CrawlerRunConfig (like excluded_tags, remove_forms, etc.), you’ll see the result here:

config = CrawlerRunConfig(
    excluded_tags=["form", "header", "footer"],
    keep_data_attributes=False
)
result = await crawler.arun("https://example.com", config=config)
print(result.cleaned_html)  # Freed of forms, header, footer, data-* attributes
3. Markdown Generation
3.1 markdown
markdown: The current location for detailed markdown output, returning a MarkdownGenerationResult object.
markdown_v2: Deprecated since v0.5.
MarkdownGenerationResult Fields:

Field	Description
raw_markdown	The basic HTML→Markdown conversion.
markdown_with_citations	Markdown including inline citations that reference links at the end.
references_markdown	The references/citations themselves (if citations=True).
fit_markdown	The filtered/“fit” markdown if a content filter was used.
fit_html	The filtered HTML that generated fit_markdown.
3.2 Basic Example with a Markdown Generator
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.markdown_generation_strategy import DefaultMarkdownGenerator

config = CrawlerRunConfig(
    markdown_generator=DefaultMarkdownGenerator(
        options={"citations": True, "body_width": 80}  # e.g. pass html2text style options
    )
)
result = await crawler.arun(url="https://example.com", config=config)

md_res = result.markdown  # or eventually 'result.markdown'
print(md_res.raw_markdown[:500])
print(md_res.markdown_with_citations)
print(md_res.references_markdown)
Note: If you use a filter like PruningContentFilter, you’ll get fit_markdown and fit_html as well.

4. Structured Extraction: extracted_content
If you run a JSON-based extraction strategy (CSS, XPath, LLM, etc.), the structured data is not stored in markdown—it’s placed in result.extracted_content as a JSON string (or sometimes plain text).

Example: CSS Extraction with raw:// HTML
import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def main():
    schema = {
        "name": "Example Items",
        "baseSelector": "div.item",
        "fields": [
            {"name": "title", "selector": "h2", "type": "text"},
            {"name": "link", "selector": "a", "type": "attribute", "attribute": "href"}
        ]
    }
    raw_html = "<div class='item'><h2>Item 1</h2><a href='https://example.com/item1'>Link 1</a></div>"

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="raw://" + raw_html,
            config=CrawlerRunConfig(
                cache_mode=CacheMode.BYPASS,
                extraction_strategy=JsonCssExtractionStrategy(schema)
            )
        )
        data = json.loads(result.extracted_content)
        print(data)

if __name__ == "__main__":
    asyncio.run(main())
Here: - url="raw://..." passes the HTML content directly, no network requests.
- The CSS extraction strategy populates result.extracted_content with the JSON array [{"title": "...", "link": "..."}].

5. More Fields: Links, Media, and More
5.1 links
A dictionary, typically with "internal" and "external" lists. Each entry might have href, text, title, etc. This is automatically captured if you haven’t disabled link extraction.

print(result.links["internal"][:3])  # Show first 3 internal links
5.2 media
Similarly, a dictionary with "images", "audio", "video", etc. Each item could include src, alt, score, and more, if your crawler is set to gather them.

images = result.media.get("images", [])
for img in images:
    print("Image URL:", img["src"], "Alt:", img.get("alt"))
5.3 screenshot and pdf
If you set screenshot=True or pdf=True in CrawlerRunConfig, then:

result.screenshot contains a base64-encoded PNG string.
result.pdf contains raw PDF bytes (you can write them to a file).
with open("page.pdf", "wb") as f:
    f.write(result.pdf)
5.4 ssl_certificate
If fetch_ssl_certificate=True, result.ssl_certificate holds details about the site’s SSL cert, such as issuer, validity dates, etc.

6. Accessing These Fields
After you run:

result = await crawler.arun(url="https://example.com", config=some_config)
Check any field:

if result.success:
    print(result.status_code, result.response_headers)
    print("Links found:", len(result.links.get("internal", [])))
    if result.markdown:
        print("Markdown snippet:", result.markdown.raw_markdown[:200])
    if result.extracted_content:
        print("Structured JSON:", result.extracted_content)
else:
    print("Error:", result.error_message)
Deprecation: Since v0.5 result.markdown_v2, result.fit_html,result.fit_markdown are deprecated. Use result.markdown instead! It holds MarkdownGenerationResult, which includes fit_html and fit_markdown as it's properties.

7. Next Steps
Markdown Generation: Dive deeper into how to configure DefaultMarkdownGenerator and various filters.
Content Filtering: Learn how to use BM25ContentFilter and PruningContentFilter.
Session & Hooks: If you want to manipulate the page or preserve state across multiple arun() calls, see the hooking or session docs.
LLM Extraction: For complex or unstructured content requiring AI-driven parsing, check the LLM-based strategies doc.
Enjoy exploring all that CrawlResult offers—whether you need raw HTML, sanitized output, markdown, or fully structured data, Crawl4AI has you covered!


------------------
Browser, Crawler & LLM Configuration (Quick Overview)
Crawl4AI’s flexibility stems from two key classes:

1. BrowserConfig – Dictates how the browser is launched and behaves (e.g., headless or visible, proxy, user agent).
2. CrawlerRunConfig – Dictates how each crawl operates (e.g., caching, extraction, timeouts, JavaScript code to run, etc.).
3. LLMConfig - Dictates how LLM providers are configured. (model, api token, base url, temperature etc.)

In most examples, you create one BrowserConfig for the entire crawler session, then pass a fresh or re-used CrawlerRunConfig whenever you call arun(). This tutorial shows the most commonly used parameters. If you need advanced or rarely used fields, see the Configuration Parameters.

1. BrowserConfig Essentials
class BrowserConfig:
    def __init__(
        browser_type="chromium",
        headless=True,
        proxy_config=None,
        viewport_width=1080,
        viewport_height=600,
        verbose=True,
        use_persistent_context=False,
        user_data_dir=None,
        cookies=None,
        headers=None,
        user_agent=None,
        text_mode=False,
        light_mode=False,
        extra_args=None,
        # ... other advanced parameters omitted here
    ):
        ...
Key Fields to Note
1. browser_type
- Options: "chromium", "firefox", or "webkit".
- Defaults to "chromium".
- If you need a different engine, specify it here.

2. headless
- True: Runs the browser in headless mode (invisible browser).
- False: Runs the browser in visible mode, which helps with debugging.

3. proxy_config
- A dictionary with fields like:

{
    "server": "http://proxy.example.com:8080", 
    "username": "...", 
    "password": "..."
}
- Leave as None if a proxy is not required.
4. viewport_width & viewport_height:
- The initial window size.
- Some sites behave differently with smaller or bigger viewports.

5. verbose:
- If True, prints extra logs.
- Handy for debugging.

6. use_persistent_context:
- If True, uses a persistent browser profile, storing cookies/local storage across runs.
- Typically also set user_data_dir to point to a folder.

7. cookies & headers:
- If you want to start with specific cookies or add universal HTTP headers, set them here.
- E.g. cookies=[{"name": "session", "value": "abc123", "domain": "example.com"}].

8. user_agent:
- Custom User-Agent string. If None, a default is used.
- You can also set user_agent_mode="random" for randomization (if you want to fight bot detection).

9. text_mode & light_mode:
- text_mode=True disables images, possibly speeding up text-only crawls.
- light_mode=True turns off certain background features for performance.

10. extra_args:
- Additional flags for the underlying browser.
- E.g. ["--disable-extensions"].

Helper Methods
Both configuration classes provide a clone() method to create modified copies:

# Create a base browser config
base_browser = BrowserConfig(
    browser_type="chromium",
    headless=True,
    text_mode=True
)

# Create a visible browser config for debugging
debug_browser = base_browser.clone(
    headless=False,
    verbose=True
)
Minimal Example:

from crawl4ai import AsyncWebCrawler, BrowserConfig

browser_conf = BrowserConfig(
    browser_type="firefox",
    headless=False,
    text_mode=True
)

async with AsyncWebCrawler(config=browser_conf) as crawler:
    result = await crawler.arun("https://example.com")
    print(result.markdown[:300])
2. CrawlerRunConfig Essentials
class CrawlerRunConfig:
    def __init__(
        word_count_threshold=200,
        extraction_strategy=None,
        markdown_generator=None,
        cache_mode=None,
        js_code=None,
        wait_for=None,
        screenshot=False,
        pdf=False,
        enable_rate_limiting=False,
        rate_limit_config=None,
        memory_threshold_percent=70.0,
        check_interval=1.0,
        max_session_permit=20,
        display_mode=None,
        verbose=True,
        stream=False,  # Enable streaming for arun_many()
        # ... other advanced parameters omitted
    ):
        ...
Key Fields to Note
1. word_count_threshold:
- The minimum word count before a block is considered.
- If your site has lots of short paragraphs or items, you can lower it.

2. extraction_strategy:
- Where you plug in JSON-based extraction (CSS, LLM, etc.).
- If None, no structured extraction is done (only raw/cleaned HTML + markdown).

3. markdown_generator:
- E.g., DefaultMarkdownGenerator(...), controlling how HTML→Markdown conversion is done.
- If None, a default approach is used.

4. cache_mode:
- Controls caching behavior (ENABLED, BYPASS, DISABLED, etc.).
- If None, defaults to some level of caching or you can specify CacheMode.ENABLED.

5. js_code:
- A string or list of JS strings to execute.
- Great for “Load More” buttons or user interactions.

6. wait_for:
- A CSS or JS expression to wait for before extracting content.
- Common usage: wait_for="css:.main-loaded" or wait_for="js:() => window.loaded === true".

7. screenshot & pdf:
- If True, captures a screenshot or PDF after the page is fully loaded.
- The results go to result.screenshot (base64) or result.pdf (bytes).

8. verbose:
- Logs additional runtime details.
- Overlaps with the browser’s verbosity if also set to True in BrowserConfig.

9. enable_rate_limiting:
- If True, enables rate limiting for batch processing.
- Requires rate_limit_config to be set.

10. memory_threshold_percent:
- The memory threshold (as a percentage) to monitor.
- If exceeded, the crawler will pause or slow down.

11. check_interval:
- The interval (in seconds) to check system resources.
- Affects how often memory and CPU usage are monitored.

12. max_session_permit:
- The maximum number of concurrent crawl sessions.
- Helps prevent overwhelming the system.

13. display_mode:
- The display mode for progress information (DETAILED, BRIEF, etc.).
- Affects how much information is printed during the crawl.

Helper Methods
The clone() method is particularly useful for creating variations of your crawler configuration:

# Create a base configuration
base_config = CrawlerRunConfig(
    cache_mode=CacheMode.ENABLED,
    word_count_threshold=200,
    wait_until="networkidle"
)

# Create variations for different use cases
stream_config = base_config.clone(
    stream=True,  # Enable streaming mode
    cache_mode=CacheMode.BYPASS
)

debug_config = base_config.clone(
    page_timeout=120000,  # Longer timeout for debugging
    verbose=True
)
The clone() method: - Creates a new instance with all the same settings - Updates only the specified parameters - Leaves the original configuration unchanged - Perfect for creating variations without repeating all parameters

3. LLMConfig Essentials
Key fields to note
1. provider:
- Which LLM provoder to use. - Possible values are "ollama/llama3","groq/llama3-70b-8192","groq/llama3-8b-8192", "openai/gpt-4o-mini" ,"openai/gpt-4o","openai/o1-mini","openai/o1-preview","openai/o3-mini","openai/o3-mini-high","anthropic/claude-3-haiku-20240307","anthropic/claude-3-opus-20240229","anthropic/claude-3-sonnet-20240229","anthropic/claude-3-5-sonnet-20240620","gemini/gemini-pro","gemini/gemini-1.5-pro","gemini/gemini-2.0-flash","gemini/gemini-2.0-flash-exp","gemini/gemini-2.0-flash-lite-preview-02-05","deepseek/deepseek-chat"
(default: "openai/gpt-4o-mini")

2. api_token:
- Optional. When not provided explicitly, api_token will be read from environment variables based on provider. For example: If a gemini model is passed as provider then,"GEMINI_API_KEY" will be read from environment variables
- API token of LLM provider
eg: api_token = "gsk_1ClHGGJ7Lpn4WGybR7vNWGdyb3FY7zXEw3SCiy0BAVM9lL8CQv" - Environment variable - use with prefix "env:"
eg:api_token = "env: GROQ_API_KEY"

3. base_url:
- If your provider has a custom endpoint

llm_config = LLMConfig(provider="openai/gpt-4o-mini", api_token=os.getenv("OPENAI_API_KEY"))
4. Putting It All Together
In a typical scenario, you define one BrowserConfig for your crawler session, then create one or more CrawlerRunConfig & LLMConfig depending on each call’s needs:

import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode, LLMConfig
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def main():
    # 1) Browser config: headless, bigger viewport, no proxy
    browser_conf = BrowserConfig(
        headless=True,
        viewport_width=1280,
        viewport_height=720
    )

    # 2) Example extraction strategy
    schema = {
        "name": "Articles",
        "baseSelector": "div.article",
        "fields": [
            {"name": "title", "selector": "h2", "type": "text"},
            {"name": "link", "selector": "a", "type": "attribute", "attribute": "href"}
        ]
    }
    extraction = JsonCssExtractionStrategy(schema)

    # 3) Example LLM content filtering

    gemini_config = LLMConfig(
        provider="gemini/gemini-1.5-pro" 
        api_token = "env:GEMINI_API_TOKEN"
    )

    # Initialize LLM filter with specific instruction
    filter = LLMContentFilter(
        llm_config=gemini_config,  # or your preferred provider
        instruction="""
        Focus on extracting the core educational content.
        Include:
        - Key concepts and explanations
        - Important code examples
        - Essential technical details
        Exclude:
        - Navigation elements
        - Sidebars
        - Footer content
        Format the output as clean markdown with proper code blocks and headers.
        """,
        chunk_token_threshold=500,  # Adjust based on your needs
        verbose=True
    )

    md_generator = DefaultMarkdownGenerator(
    content_filter=filter,
    options={"ignore_links": True}

    # 4) Crawler run config: skip cache, use extraction
    run_conf = CrawlerRunConfig(
        markdown_generator=md_generator,
        extraction_strategy=extraction,
        cache_mode=CacheMode.BYPASS,
    )

    async with AsyncWebCrawler(config=browser_conf) as crawler:
        # 4) Execute the crawl
        result = await crawler.arun(url="https://example.com/news", config=run_conf)

        if result.success:
            print("Extracted content:", result.extracted_content)
        else:
            print("Error:", result.error_message)

if __name__ == "__main__":
    asyncio.run(main())
5. Next Steps
For a detailed list of available parameters (including advanced ones), see:

BrowserConfig, CrawlerRunConfig & LLMConfig Reference
You can explore topics like:

Custom Hooks & Auth (Inject JavaScript or handle login forms).
Session Management (Re-use pages, preserve state across multiple calls).
Magic Mode or Identity-based Crawling (Fight bot detection by simulating user behavior).
Advanced Caching (Fine-tune read/write cache modes).
6. Conclusion
BrowserConfig, CrawlerRunConfig and LLMConfig give you straightforward ways to define:

Which browser to launch, how it should run, and any proxy or user agent needs.
How each crawl should behave—caching, timeouts, JavaScript code, extraction strategies, etc.
Which LLM provider to use, api token, temperature and base url for custom endpoints
Use them together for clear, maintainable code, and when you need more specialized behavior, check out the advanced parameters in the reference docs. Happy crawling!


-----------------
Markdown Generation Basics
One of Crawl4AI’s core features is generating clean, structured markdown from web pages. Originally built to solve the problem of extracting only the “actual” content and discarding boilerplate or noise, Crawl4AI’s markdown system remains one of its biggest draws for AI workflows.

In this tutorial, you’ll learn:

How to configure the Default Markdown Generator
How content filters (BM25 or Pruning) help you refine markdown and discard junk
The difference between raw markdown (result.markdown) and filtered markdown (fit_markdown)
Prerequisites
- You’ve completed or read AsyncWebCrawler Basics to understand how to run a simple crawl.
- You know how to configure CrawlerRunConfig.

1. Quick Example
Here’s a minimal code snippet that uses the DefaultMarkdownGenerator with no additional filtering:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.markdown_generation_strategy import DefaultMarkdownGenerator

async def main():
    config = CrawlerRunConfig(
        markdown_generator=DefaultMarkdownGenerator()
    )
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://example.com", config=config)

        if result.success:
            print("Raw Markdown Output:\n")
            print(result.markdown)  # The unfiltered markdown from the page
        else:
            print("Crawl failed:", result.error_message)

if __name__ == "__main__":
    asyncio.run(main())
What’s happening?
- CrawlerRunConfig( markdown_generator = DefaultMarkdownGenerator() ) instructs Crawl4AI to convert the final HTML into markdown at the end of each crawl.
- The resulting markdown is accessible via result.markdown.

2. How Markdown Generation Works
2.1 HTML-to-Text Conversion (Forked & Modified)
Under the hood, DefaultMarkdownGenerator uses a specialized HTML-to-text approach that:

Preserves headings, code blocks, bullet points, etc.
Removes extraneous tags (scripts, styles) that don’t add meaningful content.
Can optionally generate references for links or skip them altogether.
A set of options (passed as a dict) allows you to customize precisely how HTML converts to markdown. These map to standard html2text-like configuration plus your own enhancements (e.g., ignoring internal links, preserving certain tags verbatim, or adjusting line widths).

2.2 Link Citations & References
By default, the generator can convert <a href="..."> elements into [text][1] citations, then place the actual links at the bottom of the document. This is handy for research workflows that demand references in a structured manner.

2.3 Optional Content Filters
Before or after the HTML-to-Markdown step, you can apply a content filter (like BM25 or Pruning) to reduce noise and produce a “fit_markdown”—a heavily pruned version focusing on the page’s main text. We’ll cover these filters shortly.

3. Configuring the Default Markdown Generator
You can tweak the output by passing an options dict to DefaultMarkdownGenerator. For example:

from crawl4ai.markdown_generation_strategy import DefaultMarkdownGenerator
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig

async def main():
    # Example: ignore all links, don't escape HTML, and wrap text at 80 characters
    md_generator = DefaultMarkdownGenerator(
        options={
            "ignore_links": True,
            "escape_html": False,
            "body_width": 80
        }
    )

    config = CrawlerRunConfig(
        markdown_generator=md_generator
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://example.com/docs", config=config)
        if result.success:
            print("Markdown:\n", result.markdown[:500])  # Just a snippet
        else:
            print("Crawl failed:", result.error_message)

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
Some commonly used options:

ignore_links (bool): Whether to remove all hyperlinks in the final markdown.
ignore_images (bool): Remove all ![image]() references.
escape_html (bool): Turn HTML entities into text (default is often True).
body_width (int): Wrap text at N characters. 0 or None means no wrapping.
skip_internal_links (bool): If True, omit #localAnchors or internal links referencing the same page.
include_sup_sub (bool): Attempt to handle <sup> / <sub> in a more readable way.
4. Content Filters
Content filters selectively remove or rank sections of text before turning them into Markdown. This is especially helpful if your page has ads, nav bars, or other clutter you don’t want.

4.1 BM25ContentFilter
If you have a search query, BM25 is a good choice:

from crawl4ai.markdown_generation_strategy import DefaultMarkdownGenerator
from crawl4ai.content_filter_strategy import BM25ContentFilter
from crawl4ai import CrawlerRunConfig

bm25_filter = BM25ContentFilter(
    user_query="machine learning",
    bm25_threshold=1.2,
    use_stemming=True
)

md_generator = DefaultMarkdownGenerator(
    content_filter=bm25_filter,
    options={"ignore_links": True}
)

config = CrawlerRunConfig(markdown_generator=md_generator)
user_query: The term you want to focus on. BM25 tries to keep only content blocks relevant to that query.
bm25_threshold: Raise it to keep fewer blocks; lower it to keep more.
use_stemming: If True, variations of words match (e.g., “learn,” “learning,” “learnt”).
No query provided? BM25 tries to glean a context from page metadata, or you can simply treat it as a scorched-earth approach that discards text with low generic score. Realistically, you want to supply a query for best results.

4.2 PruningContentFilter
If you don’t have a specific query, or if you just want a robust “junk remover,” use PruningContentFilter. It analyzes text density, link density, HTML structure, and known patterns (like “nav,” “footer”) to systematically prune extraneous or repetitive sections.

from crawl4ai.content_filter_strategy import PruningContentFilter

prune_filter = PruningContentFilter(
    threshold=0.5,
    threshold_type="fixed",  # or "dynamic"
    min_word_threshold=50
)
threshold: Score boundary. Blocks below this score get removed.
threshold_type:
"fixed": Straight comparison (score >= threshold keeps the block).
"dynamic": The filter adjusts threshold in a data-driven manner.
min_word_threshold: Discard blocks under N words as likely too short or unhelpful.
When to Use PruningContentFilter
- You want a broad cleanup without a user query.
- The page has lots of repeated sidebars, footers, or disclaimers that hamper text extraction.

4.3 LLMContentFilter
For intelligent content filtering and high-quality markdown generation, you can use the LLMContentFilter. This filter leverages LLMs to generate relevant markdown while preserving the original content's meaning and structure:

from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, LLMConfig
from crawl4ai.content_filter_strategy import LLMContentFilter

async def main():
    # Initialize LLM filter with specific instruction
    filter = LLMContentFilter(
        llm_config = LLMConfig(provider="openai/gpt-4o",api_token="your-api-token"), #or use environment variable
        instruction="""
        Focus on extracting the core educational content.
        Include:
        - Key concepts and explanations
        - Important code examples
        - Essential technical details
        Exclude:
        - Navigation elements
        - Sidebars
        - Footer content
        Format the output as clean markdown with proper code blocks and headers.
        """,
        chunk_token_threshold=4096,  # Adjust based on your needs
        verbose=True
    )

    config = CrawlerRunConfig(
        content_filter=filter
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://example.com", config=config)
        print(result.markdown.fit_markdown)  # Filtered markdown content
Key Features: - Intelligent Filtering: Uses LLMs to understand and extract relevant content while maintaining context - Customizable Instructions: Tailor the filtering process with specific instructions - Chunk Processing: Handles large documents by processing them in chunks (controlled by chunk_token_threshold) - Parallel Processing: For better performance, use smaller chunk_token_threshold (e.g., 2048 or 4096) to enable parallel processing of content chunks

Two Common Use Cases:

Exact Content Preservation:

filter = LLMContentFilter(
    instruction="""
    Extract the main educational content while preserving its original wording and substance completely.
    1. Maintain the exact language and terminology
    2. Keep all technical explanations and examples intact
    3. Preserve the original flow and structure
    4. Remove only clearly irrelevant elements like navigation menus and ads
    """,
    chunk_token_threshold=4096
)
Focused Content Extraction:

filter = LLMContentFilter(
    instruction="""
    Focus on extracting specific types of content:
    - Technical documentation
    - Code examples
    - API references
    Reformat the content into clear, well-structured markdown
    """,
    chunk_token_threshold=4096
)
Performance Tip: Set a smaller chunk_token_threshold (e.g., 2048 or 4096) to enable parallel processing of content chunks. The default value is infinity, which processes the entire content as a single chunk.

5. Using Fit Markdown
When a content filter is active, the library produces two forms of markdown inside result.markdown:

1. raw_markdown: The full unfiltered markdown.
2. fit_markdown: A “fit” version where the filter has removed or trimmed noisy segments.

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.markdown_generation_strategy import DefaultMarkdownGenerator
from crawl4ai.content_filter_strategy import PruningContentFilter

async def main():
    config = CrawlerRunConfig(
        markdown_generator=DefaultMarkdownGenerator(
            content_filter=PruningContentFilter(threshold=0.6),
            options={"ignore_links": True}
        )
    )
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://news.example.com/tech", config=config)
        if result.success:
            print("Raw markdown:\n", result.markdown)

            # If a filter is used, we also have .fit_markdown:
            md_object = result.markdown  # or your equivalent
            print("Filtered markdown:\n", md_object.fit_markdown)
        else:
            print("Crawl failed:", result.error_message)

if __name__ == "__main__":
    asyncio.run(main())
6. The MarkdownGenerationResult Object
If your library stores detailed markdown output in an object like MarkdownGenerationResult, you’ll see fields such as:

raw_markdown: The direct HTML-to-markdown transformation (no filtering).
markdown_with_citations: A version that moves links to reference-style footnotes.
references_markdown: A separate string or section containing the gathered references.
fit_markdown: The filtered markdown if you used a content filter.
fit_html: The corresponding HTML snippet used to generate fit_markdown (helpful for debugging or advanced usage).
Example:

md_obj = result.markdown  # your library’s naming may vary
print("RAW:\n", md_obj.raw_markdown)
print("CITED:\n", md_obj.markdown_with_citations)
print("REFERENCES:\n", md_obj.references_markdown)
print("FIT:\n", md_obj.fit_markdown)
Why Does This Matter?
- You can supply raw_markdown to an LLM if you want the entire text.
- Or feed fit_markdown into a vector database to reduce token usage.
- references_markdown can help you keep track of link provenance.

Below is a revised section under “Combining Filters (BM25 + Pruning)” that demonstrates how you can run two passes of content filtering without re-crawling, by taking the HTML (or text) from a first pass and feeding it into the second filter. It uses real code patterns from the snippet you provided for BM25ContentFilter, which directly accepts HTML strings (and can also handle plain text with minimal adaptation).

7. Combining Filters (BM25 + Pruning) in Two Passes
You might want to prune out noisy boilerplate first (with PruningContentFilter), and then rank what’s left against a user query (with BM25ContentFilter). You don’t have to crawl the page twice. Instead:

1. First pass: Apply PruningContentFilter directly to the raw HTML from result.html (the crawler’s downloaded HTML).
2. Second pass: Take the pruned HTML (or text) from step 1, and feed it into BM25ContentFilter, focusing on a user query.

Two-Pass Example
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.content_filter_strategy import PruningContentFilter, BM25ContentFilter
from bs4 import BeautifulSoup

async def main():
    # 1. Crawl with minimal or no markdown generator, just get raw HTML
    config = CrawlerRunConfig(
        # If you only want raw HTML, you can skip passing a markdown_generator
        # or provide one but focus on .html in this example
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://example.com/tech-article", config=config)

        if not result.success or not result.html:
            print("Crawl failed or no HTML content.")
            return

        raw_html = result.html

        # 2. First pass: PruningContentFilter on raw HTML
        pruning_filter = PruningContentFilter(threshold=0.5, min_word_threshold=50)

        # filter_content returns a list of "text chunks" or cleaned HTML sections
        pruned_chunks = pruning_filter.filter_content(raw_html)
        # This list is basically pruned content blocks, presumably in HTML or text form

        # For demonstration, let's combine these chunks back into a single HTML-like string
        # or you could do further processing. It's up to your pipeline design.
        pruned_html = "\n".join(pruned_chunks)

        # 3. Second pass: BM25ContentFilter with a user query
        bm25_filter = BM25ContentFilter(
            user_query="machine learning",
            bm25_threshold=1.2,
            language="english"
        )

        # returns a list of text chunks
        bm25_chunks = bm25_filter.filter_content(pruned_html)  

        if not bm25_chunks:
            print("Nothing matched the BM25 query after pruning.")
            return

        # 4. Combine or display final results
        final_text = "\n---\n".join(bm25_chunks)

        print("==== PRUNED OUTPUT (first pass) ====")
        print(pruned_html[:500], "... (truncated)")  # preview

        print("\n==== BM25 OUTPUT (second pass) ====")
        print(final_text[:500], "... (truncated)")

if __name__ == "__main__":
    asyncio.run(main())
What’s Happening?
1. Raw HTML: We crawl once and store the raw HTML in result.html.
2. PruningContentFilter: Takes HTML + optional parameters. It extracts blocks of text or partial HTML, removing headings/sections deemed “noise.” It returns a list of text chunks.
3. Combine or Transform: We join these pruned chunks back into a single HTML-like string. (Alternatively, you could store them in a list for further logic—whatever suits your pipeline.)
4. BM25ContentFilter: We feed the pruned string into BM25ContentFilter with a user query. This second pass further narrows the content to chunks relevant to “machine learning.”

No Re-Crawling: We used raw_html from the first pass, so there’s no need to run arun() again—no second network request.

Tips & Variations
Plain Text vs. HTML: If your pruned output is mostly text, BM25 can still handle it; just keep in mind it expects a valid string input. If you supply partial HTML (like "<p>some text</p>"), it will parse it as HTML.
Chaining in a Single Pipeline: If your code supports it, you can chain multiple filters automatically. Otherwise, manual two-pass filtering (as shown) is straightforward.
Adjust Thresholds: If you see too much or too little text in step one, tweak threshold=0.5 or min_word_threshold=50. Similarly, bm25_threshold=1.2 can be raised/lowered for more or fewer chunks in step two.
One-Pass Combination?
If your codebase or pipeline design allows applying multiple filters in one pass, you could do so. But often it’s simpler—and more transparent—to run them sequentially, analyzing each step’s result.

Bottom Line: By manually chaining your filtering logic in two passes, you get powerful incremental control over the final content. First, remove “global” clutter with Pruning, then refine further with BM25-based query relevance—without incurring a second network crawl.

8. Common Pitfalls & Tips
1. No Markdown Output?
- Make sure the crawler actually retrieved HTML. If the site is heavily JS-based, you may need to enable dynamic rendering or wait for elements.
- Check if your content filter is too aggressive. Lower thresholds or disable the filter to see if content reappears.

2. Performance Considerations
- Very large pages with multiple filters can be slower. Consider cache_mode to avoid re-downloading.
- If your final use case is LLM ingestion, consider summarizing further or chunking big texts.

3. Take Advantage of fit_markdown
- Great for RAG pipelines, semantic search, or any scenario where extraneous boilerplate is unwanted.
- Still verify the textual quality—some sites have crucial data in footers or sidebars.

4. Adjusting html2text Options
- If you see lots of raw HTML slipping into the text, turn on escape_html.
- If code blocks look messy, experiment with mark_code or handle_code_in_pre.

9. Summary & Next Steps
In this Markdown Generation Basics tutorial, you learned to:

Configure the DefaultMarkdownGenerator with HTML-to-text options.
Use BM25ContentFilter for query-specific extraction or PruningContentFilter for general noise removal.
Distinguish between raw and filtered markdown (fit_markdown).
Leverage the MarkdownGenerationResult object to handle different forms of output (citations, references, etc.).
Now you can produce high-quality Markdown from any website, focusing on exactly the content you need—an essential step for powering AI models, summarization pipelines, or knowledge-base queries.


----------------
Content Selection
Crawl4AI provides multiple ways to select, filter, and refine the content from your crawls. Whether you need to target a specific CSS region, exclude entire tags, filter out external links, or remove certain domains and images, CrawlerRunConfig offers a wide range of parameters.

Below, we show how to configure these parameters and combine them for precise control.

1. CSS-Based Selection
There are two ways to select content from a page: using css_selector or the more flexible target_elements.

1.1 Using css_selector
A straightforward way to limit your crawl results to a certain region of the page is css_selector in CrawlerRunConfig:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig

async def main():
    config = CrawlerRunConfig(
        # e.g., first 30 items from Hacker News
        css_selector=".athing:nth-child(-n+30)"  
    )
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://news.ycombinator.com/newest", 
            config=config
        )
        print("Partial HTML length:", len(result.cleaned_html))

if __name__ == "__main__":
    asyncio.run(main())
Result: Only elements matching that selector remain in result.cleaned_html.

1.2 Using target_elements
The target_elements parameter provides more flexibility by allowing you to target multiple elements for content extraction while preserving the entire page context for other features:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig

async def main():
    config = CrawlerRunConfig(
        # Target article body and sidebar, but not other content
        target_elements=["article.main-content", "aside.sidebar"]
    )
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/blog-post", 
            config=config
        )
        print("Markdown focused on target elements")
        print("Links from entire page still available:", len(result.links.get("internal", [])))

if __name__ == "__main__":
    asyncio.run(main())
Key difference: With target_elements, the markdown generation and structural data extraction focus on those elements, but other page elements (like links, images, and tables) are still extracted from the entire page. This gives you fine-grained control over what appears in your markdown content while preserving full page context for link analysis and media collection.

2. Content Filtering & Exclusions
2.1 Basic Overview
config = CrawlerRunConfig(
    # Content thresholds
    word_count_threshold=10,        # Minimum words per block

    # Tag exclusions
    excluded_tags=['form', 'header', 'footer', 'nav'],

    # Link filtering
    exclude_external_links=True,    
    exclude_social_media_links=True,
    # Block entire domains
    exclude_domains=["adtrackers.com", "spammynews.org"],    
    exclude_social_media_domains=["facebook.com", "twitter.com"],

    # Media filtering
    exclude_external_images=True
)
Explanation:

word_count_threshold: Ignores text blocks under X words. Helps skip trivial blocks like short nav or disclaimers.
excluded_tags: Removes entire tags (<form>, <header>, <footer>, etc.).
Link Filtering:
exclude_external_links: Strips out external links and may remove them from result.links.
exclude_social_media_links: Removes links pointing to known social media domains.
exclude_domains: A custom list of domains to block if discovered in links.
exclude_social_media_domains: A curated list (override or add to it) for social media sites.
Media Filtering:
exclude_external_images: Discards images not hosted on the same domain as the main page (or its subdomains).
By default in case you set exclude_social_media_links=True, the following social media domains are excluded:

[
    'facebook.com',
    'twitter.com',
    'x.com',
    'linkedin.com',
    'instagram.com',
    'pinterest.com',
    'tiktok.com',
    'snapchat.com',
    'reddit.com',
]
2.2 Example Usage
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode

async def main():
    config = CrawlerRunConfig(
        css_selector="main.content", 
        word_count_threshold=10,
        excluded_tags=["nav", "footer"],
        exclude_external_links=True,
        exclude_social_media_links=True,
        exclude_domains=["ads.com", "spammytrackers.net"],
        exclude_external_images=True,
        cache_mode=CacheMode.BYPASS
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url="https://news.ycombinator.com", config=config)
        print("Cleaned HTML length:", len(result.cleaned_html))

if __name__ == "__main__":
    asyncio.run(main())
Note: If these parameters remove too much, reduce or disable them accordingly.

3. Handling Iframes
Some sites embed content in <iframe> tags. If you want that inline:

config = CrawlerRunConfig(
    # Merge iframe content into the final output
    process_iframes=True,    
    remove_overlay_elements=True
)
Usage:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig

async def main():
    config = CrawlerRunConfig(
        process_iframes=True,
        remove_overlay_elements=True
    )
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.org/iframe-demo", 
            config=config
        )
        print("Iframe-merged length:", len(result.cleaned_html))

if __name__ == "__main__":
    asyncio.run(main())
4. Structured Extraction Examples
You can combine content selection with a more advanced extraction strategy. For instance, a CSS-based or LLM-based extraction strategy can run on the filtered HTML.

4.1 Pattern-Based with JsonCssExtractionStrategy
import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def main():
    # Minimal schema for repeated items
    schema = {
        "name": "News Items",
        "baseSelector": "tr.athing",
        "fields": [
            {"name": "title", "selector": "span.titleline a", "type": "text"},
            {
                "name": "link", 
                "selector": "span.titleline a", 
                "type": "attribute", 
                "attribute": "href"
            }
        ]
    }

    config = CrawlerRunConfig(
        # Content filtering
        excluded_tags=["form", "header"],
        exclude_domains=["adsite.com"],

        # CSS selection or entire page
        css_selector="table.itemlist",

        # No caching for demonstration
        cache_mode=CacheMode.BYPASS,

        # Extraction strategy
        extraction_strategy=JsonCssExtractionStrategy(schema)
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://news.ycombinator.com/newest", 
            config=config
        )
        data = json.loads(result.extracted_content)
        print("Sample extracted item:", data[:1])  # Show first item

if __name__ == "__main__":
    asyncio.run(main())
4.2 LLM-Based Extraction
import asyncio
import json
from pydantic import BaseModel, Field
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LLMConfig
from crawl4ai.extraction_strategy import LLMExtractionStrategy

class ArticleData(BaseModel):
    headline: str
    summary: str

async def main():
    llm_strategy = LLMExtractionStrategy(
        llm_config = LLMConfig(provider="openai/gpt-4",api_token="sk-YOUR_API_KEY")
        schema=ArticleData.schema(),
        extraction_type="schema",
        instruction="Extract 'headline' and a short 'summary' from the content."
    )

    config = CrawlerRunConfig(
        exclude_external_links=True,
        word_count_threshold=20,
        extraction_strategy=llm_strategy
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url="https://news.ycombinator.com", config=config)
        article = json.loads(result.extracted_content)
        print(article)

if __name__ == "__main__":
    asyncio.run(main())
Here, the crawler:

Filters out external links (exclude_external_links=True).
Ignores very short text blocks (word_count_threshold=20).
Passes the final HTML to your LLM strategy for an AI-driven parse.
5. Comprehensive Example
Below is a short function that unifies CSS selection, exclusion logic, and a pattern-based extraction, demonstrating how you can fine-tune your final data:

import asyncio
import json
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def extract_main_articles(url: str):
    schema = {
        "name": "ArticleBlock",
        "baseSelector": "div.article-block",
        "fields": [
            {"name": "headline", "selector": "h2", "type": "text"},
            {"name": "summary", "selector": ".summary", "type": "text"},
            {
                "name": "metadata",
                "type": "nested",
                "fields": [
                    {"name": "author", "selector": ".author", "type": "text"},
                    {"name": "date", "selector": ".date", "type": "text"}
                ]
            }
        ]
    }

    config = CrawlerRunConfig(
        # Keep only #main-content
        css_selector="#main-content",

        # Filtering
        word_count_threshold=10,
        excluded_tags=["nav", "footer"],  
        exclude_external_links=True,
        exclude_domains=["somebadsite.com"],
        exclude_external_images=True,

        # Extraction
        extraction_strategy=JsonCssExtractionStrategy(schema),

        cache_mode=CacheMode.BYPASS
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url=url, config=config)
        if not result.success:
            print(f"Error: {result.error_message}")
            return None
        return json.loads(result.extracted_content)

async def main():
    articles = await extract_main_articles("https://news.ycombinator.com/newest")
    if articles:
        print("Extracted Articles:", articles[:2])  # Show first 2

if __name__ == "__main__":
    asyncio.run(main())
Why This Works: - CSS scoping with #main-content.
- Multiple exclude_ parameters to remove domains, external images, etc.
- A JsonCssExtractionStrategy to parse repeated article blocks.

6. Scraping Modes
Crawl4AI provides two different scraping strategies for HTML content processing: WebScrapingStrategy (BeautifulSoup-based, default) and LXMLWebScrapingStrategy (LXML-based). The LXML strategy offers significantly better performance, especially for large HTML documents.

from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, LXMLWebScrapingStrategy

async def main():
    config = CrawlerRunConfig(
        scraping_strategy=LXMLWebScrapingStrategy()  # Faster alternative to default BeautifulSoup
    )
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com", 
            config=config
        )
You can also create your own custom scraping strategy by inheriting from ContentScrapingStrategy. The strategy must return a ScrapingResult object with the following structure:

from crawl4ai import ContentScrapingStrategy, ScrapingResult, MediaItem, Media, Link, Links

class CustomScrapingStrategy(ContentScrapingStrategy):
    def scrap(self, url: str, html: str, **kwargs) -> ScrapingResult:
        # Implement your custom scraping logic here
        return ScrapingResult(
            cleaned_html="<html>...</html>",  # Cleaned HTML content
            success=True,                     # Whether scraping was successful
            media=Media(
                images=[                      # List of images found
                    MediaItem(
                        src="https://example.com/image.jpg",
                        alt="Image description",
                        desc="Surrounding text",
                        score=1,
                        type="image",
                        group_id=1,
                        format="jpg",
                        width=800
                    )
                ],
                videos=[],                    # List of videos (same structure as images)
                audios=[]                     # List of audio files (same structure as images)
            ),
            links=Links(
                internal=[                    # List of internal links
                    Link(
                        href="https://example.com/page",
                        text="Link text",
                        title="Link title",
                        base_domain="example.com"
                    )
                ],
                external=[]                   # List of external links (same structure)
            ),
            metadata={                        # Additional metadata
                "title": "Page Title",
                "description": "Page description"
            }
        )

    async def ascrap(self, url: str, html: str, **kwargs) -> ScrapingResult:
        # For simple cases, you can use the sync version
        return await asyncio.to_thread(self.scrap, url, html, **kwargs)
Performance Considerations
The LXML strategy can be up to 10-20x faster than BeautifulSoup strategy, particularly when processing large HTML documents. However, please note:

LXML strategy is currently experimental
In some edge cases, the parsing results might differ slightly from BeautifulSoup
If you encounter any inconsistencies between LXML and BeautifulSoup results, please raise an issue with a reproducible example
Choose LXML strategy when: - Processing large HTML documents (recommended for >100KB) - Performance is critical - Working with well-formed HTML

Stick to BeautifulSoup strategy (default) when: - Maximum compatibility is needed - Working with malformed HTML - Exact parsing behavior is critical

7. Combining CSS Selection Methods
You can combine css_selector and target_elements in powerful ways to achieve fine-grained control over your output:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode

async def main():
    # Target specific content but preserve page context
    config = CrawlerRunConfig(
        # Focus markdown on main content and sidebar
        target_elements=["#main-content", ".sidebar"],

        # Global filters applied to entire page
        excluded_tags=["nav", "footer", "header"],
        exclude_external_links=True,

        # Use basic content thresholds
        word_count_threshold=15,

        cache_mode=CacheMode.BYPASS
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com/article",
            config=config
        )

        print(f"Content focuses on specific elements, but all links still analyzed")
        print(f"Internal links: {len(result.links.get('internal', []))}")
        print(f"External links: {len(result.links.get('external', []))}")

if __name__ == "__main__":
    asyncio.run(main())
This approach gives you the best of both worlds: - Markdown generation and content extraction focus on the elements you care about - Links, images and other page data still give you the full context of the page - Content filtering still applies globally

8. Conclusion
By mixing target_elements or css_selector scoping, content filtering parameters, and advanced extraction strategies, you can precisely choose which data to keep. Key parameters in CrawlerRunConfig for content selection include:

target_elements – Array of CSS selectors to focus markdown generation and data extraction, while preserving full page context for links and media.
css_selector – Basic scoping to an element or region for all extraction processes.
word_count_threshold – Skip short blocks.
excluded_tags – Remove entire HTML tags.
exclude_external_links, exclude_social_media_links, exclude_domains – Filter out unwanted links or domains.
exclude_external_images – Remove images from external sources.
process_iframes – Merge iframe content if needed.
Combine these with structured extraction (CSS, LLM-based, or others) to build powerful crawls that yield exactly the content you want, from raw or cleaned HTML up to sophisticated JSON structures. For more detail, see Configuration Reference. Enjoy curating your data to the max!

-------------
Prefix-Based Input Handling in Crawl4AI
This guide will walk you through using the Crawl4AI library to crawl web pages, local HTML files, and raw HTML strings. We'll demonstrate these capabilities using a Wikipedia page as an example.

Crawling a Web URL
To crawl a live web page, provide the URL starting with http:// or https://, using a CrawlerRunConfig object:

import asyncio
from crawl4ai import AsyncWebCrawler
from crawl4ai.async_configs import CrawlerRunConfig

async def crawl_web():
    config = CrawlerRunConfig(bypass_cache=True)
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://en.wikipedia.org/wiki/apple", 
            config=config
        )
        if result.success:
            print("Markdown Content:")
            print(result.markdown)
        else:
            print(f"Failed to crawl: {result.error_message}")

asyncio.run(crawl_web())
Crawling a Local HTML File
To crawl a local HTML file, prefix the file path with file://.

import asyncio
from crawl4ai import AsyncWebCrawler
from crawl4ai.async_configs import CrawlerRunConfig

async def crawl_local_file():
    local_file_path = "/path/to/apple.html"  # Replace with your file path
    file_url = f"file://{local_file_path}"
    config = CrawlerRunConfig(bypass_cache=True)

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url=file_url, config=config)
        if result.success:
            print("Markdown Content from Local File:")
            print(result.markdown)
        else:
            print(f"Failed to crawl local file: {result.error_message}")

asyncio.run(crawl_local_file())
Crawling Raw HTML Content
To crawl raw HTML content, prefix the HTML string with raw:.

import asyncio
from crawl4ai import AsyncWebCrawler
from crawl4ai.async_configs import CrawlerRunConfig

async def crawl_raw_html():
    raw_html = "<html><body><h1>Hello, World!</h1></body></html>"
    raw_html_url = f"raw:{raw_html}"
    config = CrawlerRunConfig(bypass_cache=True)

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url=raw_html_url, config=config)
        if result.success:
            print("Markdown Content from Raw HTML:")
            print(result.markdown)
        else:
            print(f"Failed to crawl raw HTML: {result.error_message}")

asyncio.run(crawl_raw_html())
Complete Example
Below is a comprehensive script that:

Crawls the Wikipedia page for "Apple."
Saves the HTML content to a local file (apple.html).
Crawls the local HTML file and verifies the markdown length matches the original crawl.
Crawls the raw HTML content from the saved file and verifies consistency.
import os
import sys
import asyncio
from pathlib import Path
from crawl4ai import AsyncWebCrawler
from crawl4ai.async_configs import CrawlerRunConfig

async def main():
    wikipedia_url = "https://en.wikipedia.org/wiki/apple"
    script_dir = Path(__file__).parent
    html_file_path = script_dir / "apple.html"

    async with AsyncWebCrawler() as crawler:
        # Step 1: Crawl the Web URL
        print("\n=== Step 1: Crawling the Wikipedia URL ===")
        web_config = CrawlerRunConfig(bypass_cache=True)
        result = await crawler.arun(url=wikipedia_url, config=web_config)

        if not result.success:
            print(f"Failed to crawl {wikipedia_url}: {result.error_message}")
            return

        with open(html_file_path, 'w', encoding='utf-8') as f:
            f.write(result.html)
        web_crawl_length = len(result.markdown)
        print(f"Length of markdown from web crawl: {web_crawl_length}\n")

        # Step 2: Crawl from the Local HTML File
        print("=== Step 2: Crawling from the Local HTML File ===")
        file_url = f"file://{html_file_path.resolve()}"
        file_config = CrawlerRunConfig(bypass_cache=True)
        local_result = await crawler.arun(url=file_url, config=file_config)

        if not local_result.success:
            print(f"Failed to crawl local file {file_url}: {local_result.error_message}")
            return

        local_crawl_length = len(local_result.markdown)
        assert web_crawl_length == local_crawl_length, "Markdown length mismatch"
        print("✅ Markdown length matches between web and local file crawl.\n")

        # Step 3: Crawl Using Raw HTML Content
        print("=== Step 3: Crawling Using Raw HTML Content ===")
        with open(html_file_path, 'r', encoding='utf-8') as f:
            raw_html_content = f.read()
        raw_html_url = f"raw:{raw_html_content}"
        raw_config = CrawlerRunConfig(bypass_cache=True)
        raw_result = await crawler.arun(url=raw_html_url, config=raw_config)

        if not raw_result.success:
            print(f"Failed to crawl raw HTML content: {raw_result.error_message}")
            return

        raw_crawl_length = len(raw_result.markdown)
        assert web_crawl_length == raw_crawl_length, "Markdown length mismatch"
        print("✅ Markdown length matches between web and raw HTML crawl.\n")

        print("All tests passed successfully!")
    if html_file_path.exists():
        os.remove(html_file_path)

if __name__ == "__main__":
    asyncio.run(main())
Conclusion
With the unified url parameter and prefix-based handling in Crawl4AI, you can seamlessly handle web URLs, local HTML files, and raw HTML content. Use CrawlerRunConfig for flexible and consistent configuration in all scenarios.


--------------
Link & Media
In this tutorial, you’ll learn how to:

Extract links (internal, external) from crawled pages
Filter or exclude specific domains (e.g., social media or custom domains)
Access and manage media data (especially images) in the crawl result
Configure your crawler to exclude or prioritize certain images
Prerequisites
- You have completed or are familiar with the AsyncWebCrawler Basics tutorial.
- You can run Crawl4AI in your environment (Playwright, Python, etc.).

Below is a revised version of the Link Extraction and Media Extraction sections that includes example data structures showing how links and media items are stored in CrawlResult. Feel free to adjust any field names or descriptions to match your actual output.

1. Link Extraction
1.1 result.links
When you call arun() or arun_many() on a URL, Crawl4AI automatically extracts links and stores them in the links field of CrawlResult. By default, the crawler tries to distinguish internal links (same domain) from external links (different domains).

Basic Example:

from crawl4ai import AsyncWebCrawler

async with AsyncWebCrawler() as crawler:
    result = await crawler.arun("https://www.example.com")
    if result.success:
        internal_links = result.links.get("internal", [])
        external_links = result.links.get("external", [])
        print(f"Found {len(internal_links)} internal links.")
        print(f"Found {len(internal_links)} external links.")
        print(f"Found {len(result.media)} media items.")

        # Each link is typically a dictionary with fields like:
        # { "href": "...", "text": "...", "title": "...", "base_domain": "..." }
        if internal_links:
            print("Sample Internal Link:", internal_links[0])
    else:
        print("Crawl failed:", result.error_message)
Structure Example:

result.links = {
  "internal": [
    {
      "href": "https://kidocode.com/",
      "text": "",
      "title": "",
      "base_domain": "kidocode.com"
    },
    {
      "href": "https://kidocode.com/degrees/technology",
      "text": "Technology Degree",
      "title": "KidoCode Tech Program",
      "base_domain": "kidocode.com"
    },
    # ...
  ],
  "external": [
    # possibly other links leading to third-party sites
  ]
}
href: The raw hyperlink URL.
text: The link text (if any) within the <a> tag.
title: The title attribute of the link (if present).
base_domain: The domain extracted from href. Helpful for filtering or grouping by domain.
2. Domain Filtering
Some websites contain hundreds of third-party or affiliate links. You can filter out certain domains at crawl time by configuring the crawler. The most relevant parameters in CrawlerRunConfig are:

exclude_external_links: If True, discard any link pointing outside the root domain.
exclude_social_media_domains: Provide a list of social media platforms (e.g., ["facebook.com", "twitter.com"]) to exclude from your crawl.
exclude_social_media_links: If True, automatically skip known social platforms.
exclude_domains: Provide a list of custom domains you want to exclude (e.g., ["spammyads.com", "tracker.net"]).
2.1 Example: Excluding External & Social Media Links
import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig

async def main():
    crawler_cfg = CrawlerRunConfig(
        exclude_external_links=True,          # No links outside primary domain
        exclude_social_media_links=True       # Skip recognized social media domains
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            "https://www.example.com",
            config=crawler_cfg
        )
        if result.success:
            print("[OK] Crawled:", result.url)
            print("Internal links count:", len(result.links.get("internal", [])))
            print("External links count:", len(result.links.get("external", [])))  
            # Likely zero external links in this scenario
        else:
            print("[ERROR]", result.error_message)

if __name__ == "__main__":
    asyncio.run(main())
2.2 Example: Excluding Specific Domains
If you want to let external links in, but specifically exclude a domain (e.g., suspiciousads.com), do this:

crawler_cfg = CrawlerRunConfig(
    exclude_domains=["suspiciousads.com"]
)
This approach is handy when you still want external links but need to block certain sites you consider spammy.

3. Media Extraction
3.1 Accessing result.media
By default, Crawl4AI collects images, audio, video URLs, and data tables it finds on the page. These are stored in result.media, a dictionary keyed by media type (e.g., images, videos, audio, tables).

Basic Example:

if result.success:
    # Get images
    images_info = result.media.get("images", [])
    print(f"Found {len(images_info)} images in total.")
    for i, img in enumerate(images_info[:3]):  # Inspect just the first 3
        print(f"[Image {i}] URL: {img['src']}")
        print(f"           Alt text: {img.get('alt', '')}")
        print(f"           Score: {img.get('score')}")
        print(f"           Description: {img.get('desc', '')}\n")

    # Get tables
    tables = result.media.get("tables", [])
    print(f"Found {len(tables)} data tables in total.")
    for i, table in enumerate(tables):
        print(f"[Table {i}] Caption: {table.get('caption', 'No caption')}")
        print(f"           Columns: {len(table.get('headers', []))}")
        print(f"           Rows: {len(table.get('rows', []))}")
Structure Example:

result.media = {
  "images": [
    {
      "src": "https://cdn.prod.website-files.com/.../Group%2089.svg",
      "alt": "coding school for kids",
      "desc": "Trial Class Degrees degrees All Degrees AI Degree Technology ...",
      "score": 3,
      "type": "image",
      "group_id": 0,
      "format": None,
      "width": None,
      "height": None
    },
    # ...
  ],
  "videos": [
    # Similar structure but with video-specific fields
  ],
  "audio": [
    # Similar structure but with audio-specific fields
  ],
  "tables": [
    {
      "headers": ["Name", "Age", "Location"],
      "rows": [
        ["John Doe", "34", "New York"],
        ["Jane Smith", "28", "San Francisco"],
        ["Alex Johnson", "42", "Chicago"]
      ],
      "caption": "Employee Directory",
      "summary": "Directory of company employees"
    },
    # More tables if present
  ]
}
Depending on your Crawl4AI version or scraping strategy, these dictionaries can include fields like:

src: The media URL (e.g., image source)
alt: The alt text for images (if present)
desc: A snippet of nearby text or a short description (optional)
score: A heuristic relevance score if you’re using content-scoring features
width, height: If the crawler detects dimensions for the image/video
type: Usually "image", "video", or "audio"
group_id: If you’re grouping related media items, the crawler might assign an ID
With these details, you can easily filter out or focus on certain images (for instance, ignoring images with very low scores or a different domain), or gather metadata for analytics.

3.2 Excluding External Images
If you’re dealing with heavy pages or want to skip third-party images (advertisements, for example), you can turn on:

crawler_cfg = CrawlerRunConfig(
    exclude_external_images=True
)
This setting attempts to discard images from outside the primary domain, keeping only those from the site you’re crawling.

3.3 Working with Tables
Crawl4AI can detect and extract structured data from HTML tables. Tables are analyzed based on various criteria to determine if they are actual data tables (as opposed to layout tables), including:

Presence of thead and tbody sections
Use of th elements for headers
Column consistency
Text density
And other factors
Tables that score above the threshold (default: 7) are extracted and stored in result.media.tables.

Accessing Table Data:

if result.success:
    tables = result.media.get("tables", [])
    print(f"Found {len(tables)} data tables on the page")

    if tables:
        # Access the first table
        first_table = tables[0]
        print(f"Table caption: {first_table.get('caption', 'No caption')}")
        print(f"Headers: {first_table.get('headers', [])}")

        # Print the first 3 rows
        for i, row in enumerate(first_table.get('rows', [])[:3]):
            print(f"Row {i+1}: {row}")
Configuring Table Extraction:

You can adjust the sensitivity of the table detection algorithm with:

crawler_cfg = CrawlerRunConfig(
    table_score_threshold=5  # Lower value = more tables detected (default: 7)
)
Each extracted table contains: - headers: Column header names - rows: List of rows, each containing cell values - caption: Table caption text (if available) - summary: Table summary attribute (if specified)

3.4 Additional Media Config
screenshot: Set to True if you want a full-page screenshot stored as base64 in result.screenshot.
pdf: Set to True if you want a PDF version of the page in result.pdf.
wait_for_images: If True, attempts to wait until images are fully loaded before final extraction.
4. Putting It All Together: Link & Media Filtering
Here’s a combined example demonstrating how to filter out external links, skip certain domains, and exclude external images:

import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig

async def main():
    # Suppose we want to keep only internal links, remove certain domains, 
    # and discard external images from the final crawl data.
    crawler_cfg = CrawlerRunConfig(
        exclude_external_links=True,
        exclude_domains=["spammyads.com"],
        exclude_social_media_links=True,   # skip Twitter, Facebook, etc.
        exclude_external_images=True,      # keep only images from main domain
        wait_for_images=True,             # ensure images are loaded
        verbose=True
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://www.example.com", config=crawler_cfg)

        if result.success:
            print("[OK] Crawled:", result.url)

            # 1. Links
            in_links = result.links.get("internal", [])
            ext_links = result.links.get("external", [])
            print("Internal link count:", len(in_links))
            print("External link count:", len(ext_links))  # should be zero with exclude_external_links=True

            # 2. Images
            images = result.media.get("images", [])
            print("Images found:", len(images))

            # Let's see a snippet of these images
            for i, img in enumerate(images[:3]):
                print(f"  - {img['src']} (alt={img.get('alt','')}, score={img.get('score','N/A')})")
        else:
            print("[ERROR] Failed to crawl. Reason:", result.error_message)

if __name__ == "__main__":
    asyncio.run(main())
5. Common Pitfalls & Tips
1. Conflicting Flags:
- exclude_external_links=True but then also specifying exclude_social_media_links=True is typically fine, but understand that the first setting already discards all external links. The second becomes somewhat redundant.
- exclude_external_images=True but want to keep some external images? Currently no partial domain-based setting for images, so you might need a custom approach or hook logic.

2. Relevancy Scores:
- If your version of Crawl4AI or your scraping strategy includes an img["score"], it’s typically a heuristic based on size, position, or content analysis. Evaluate carefully if you rely on it.

3. Performance:
- Excluding certain domains or external images can speed up your crawl, especially for large, media-heavy pages.
- If you want a “full” link map, do not exclude them. Instead, you can post-filter in your own code.

4. Social Media Lists:
- exclude_social_media_links=True typically references an internal list of known social domains like Facebook, Twitter, LinkedIn, etc. If you need to add or remove from that list, look for library settings or a local config file (depending on your version).

That’s it for Link & Media Analysis! You’re now equipped to filter out unwanted sites and zero in on the images and videos that matter for your project.

Table Extraction Tips
Not all HTML tables are extracted - only those detected as "data tables" vs. layout tables.
Tables with inconsistent cell counts, nested tables, or those used purely for layout may be skipped.
If you're missing tables, try adjusting the table_score_threshold to a lower value (default is 7).
The table detection algorithm scores tables based on features like consistent columns, presence of headers, text density, and more. Tables scoring above the threshold are considered data tables worth extracting.


----------------
Handling Lazy-Loaded Images
Many websites now load images lazily as you scroll. If you need to ensure they appear in your final crawl (and in result.media), consider:

1. wait_for_images=True – Wait for images to fully load.
2. scan_full_page – Force the crawler to scroll the entire page, triggering lazy loads.
3. scroll_delay – Add small delays between scroll steps.

Note: If the site requires multiple “Load More” triggers or complex interactions, see the Page Interaction docs.

Example: Ensuring Lazy Images Appear
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, BrowserConfig
from crawl4ai.async_configs import CacheMode

async def main():
    config = CrawlerRunConfig(
        # Force the crawler to wait until images are fully loaded
        wait_for_images=True,

        # Option 1: If you want to automatically scroll the page to load images
        scan_full_page=True,  # Tells the crawler to try scrolling the entire page
        scroll_delay=0.5,     # Delay (seconds) between scroll steps

        # Option 2: If the site uses a 'Load More' or JS triggers for images,
        # you can also specify js_code or wait_for logic here.

        cache_mode=CacheMode.BYPASS,
        verbose=True
    )

    async with AsyncWebCrawler(config=BrowserConfig(headless=True)) as crawler:
        result = await crawler.arun("https://www.example.com/gallery", config=config)

        if result.success:
            images = result.media.get("images", [])
            print("Images found:", len(images))
            for i, img in enumerate(images[:5]):
                print(f"[Image {i}] URL: {img['src']}, Score: {img.get('score','N/A')}")
        else:
            print("Error:", result.error_message)

if __name__ == "__main__":
    asyncio.run(main())
Explanation:

wait_for_images=True
The crawler tries to ensure images have finished loading before finalizing the HTML.
scan_full_page=True
Tells the crawler to attempt scrolling from top to bottom. Each scroll step helps trigger lazy loading.
scroll_delay=0.5
Pause half a second between each scroll step. Helps the site load images before continuing.
When to Use:

Lazy-Loading: If images appear only when the user scrolls into view, scan_full_page + scroll_delay helps the crawler see them.
Heavier Pages: If a page is extremely long, be mindful that scanning the entire page can be slow. Adjust scroll_delay or the max scroll steps as needed.
Combining with Other Link & Media Filters
You can still combine lazy-load logic with the usual exclude_external_images, exclude_domains, or link filtration:

config = CrawlerRunConfig(
    wait_for_images=True,
    scan_full_page=True,
    scroll_delay=0.5,

    # Filter out external images if you only want local ones
    exclude_external_images=True,

    # Exclude certain domains for links
    exclude_domains=["spammycdn.com"],
)
This approach ensures you see all images from the main domain while ignoring external ones, and the crawler physically scrolls the entire page so that lazy-loading triggers.

Tips & Troubleshooting
1. Long Pages
- Setting scan_full_page=True on extremely long or infinite-scroll pages can be resource-intensive.
- Consider using hooks or specialized logic to load specific sections or “Load More” triggers repeatedly.

2. Mixed Image Behavior
- Some sites load images in batches as you scroll. If you’re missing images, increase your scroll_delay or call multiple partial scrolls in a loop with JS code or hooks.

3. Combining with Dynamic Wait
- If the site has a placeholder that only changes to a real image after a certain event, you might do wait_for="css:img.loaded" or a custom JS wait_for.

4. Caching
- If cache_mode is enabled, repeated crawls might skip some network fetches. If you suspect caching is missing new images, set cache_mode=CacheMode.BYPASS for fresh fetches.

With lazy-loading support, wait_for_images, and scan_full_page settings, you can capture the entire gallery or feed of images you expect—even if the site only loads them as the user scrolls. Combine these with the standard media filtering and domain exclusion for a complete link & media handling strategy.

---------------
Advanced Multi-URL Crawling with Dispatchers
Heads Up: Crawl4AI supports advanced dispatchers for parallel or throttled crawling, providing dynamic rate limiting and memory usage checks. The built-in arun_many() function uses these dispatchers to handle concurrency efficiently.

1. Introduction
When crawling many URLs:

Basic: Use arun() in a loop (simple but less efficient)
Better: Use arun_many(), which efficiently handles multiple URLs with proper concurrency control
Best: Customize dispatcher behavior for your specific needs (memory management, rate limits, etc.)
Why Dispatchers?

Adaptive: Memory-based dispatchers can pause or slow down based on system resources
Rate-limiting: Built-in rate limiting with exponential backoff for 429/503 responses
Real-time Monitoring: Live dashboard of ongoing tasks, memory usage, and performance
Flexibility: Choose between memory-adaptive or semaphore-based concurrency
2. Core Components
2.1 Rate Limiter
class RateLimiter:
    def __init__(
        # Random delay range between requests
        base_delay: Tuple[float, float] = (1.0, 3.0),  

        # Maximum backoff delay
        max_delay: float = 60.0,                        

        # Retries before giving up
        max_retries: int = 3,                          

        # Status codes triggering backoff
        rate_limit_codes: List[int] = [429, 503]        
    )
Here’s the revised and simplified explanation of the RateLimiter, focusing on constructor parameters and adhering to your markdown style and mkDocs guidelines.

RateLimiter Constructor Parameters
The RateLimiter is a utility that helps manage the pace of requests to avoid overloading servers or getting blocked due to rate limits. It operates internally to delay requests and handle retries but can be configured using its constructor parameters.

Parameters of the RateLimiter constructor:

1. base_delay (Tuple[float, float], default: (1.0, 3.0))
  The range for a random delay (in seconds) between consecutive requests to the same domain.

A random delay is chosen between base_delay[0] and base_delay[1] for each request.
This prevents sending requests at a predictable frequency, reducing the chances of triggering rate limits.
Example:
If base_delay = (2.0, 5.0), delays could be randomly chosen as 2.3s, 4.1s, etc.

2. max_delay (float, default: 60.0)
  The maximum allowable delay when rate-limiting errors occur.

When servers return rate-limit responses (e.g., 429 or 503), the delay increases exponentially with jitter.
The max_delay ensures the delay doesn’t grow unreasonably high, capping it at this value.
Example:
For a max_delay = 30.0, even if backoff calculations suggest a delay of 45s, it will cap at 30s.

3. max_retries (int, default: 3)
  The maximum number of retries for a request if rate-limiting errors occur.

After encountering a rate-limit response, the RateLimiter retries the request up to this number of times.
If all retries fail, the request is marked as failed, and the process continues.
Example:
If max_retries = 3, the system retries a failed request three times before giving up.

4. rate_limit_codes (List[int], default: [429, 503])
  A list of HTTP status codes that trigger the rate-limiting logic.

These status codes indicate the server is overwhelmed or actively limiting requests.
You can customize this list to include other codes based on specific server behavior.
Example:
If rate_limit_codes = [429, 503, 504], the crawler will back off on these three error codes.

How to Use the RateLimiter:

Here’s an example of initializing and using a RateLimiter in your project:

from crawl4ai import RateLimiter

# Create a RateLimiter with custom settings
rate_limiter = RateLimiter(
    base_delay=(2.0, 4.0),  # Random delay between 2-4 seconds
    max_delay=30.0,         # Cap delay at 30 seconds
    max_retries=5,          # Retry up to 5 times on rate-limiting errors
    rate_limit_codes=[429, 503]  # Handle these HTTP status codes
)

# RateLimiter will handle delays and retries internally
# No additional setup is required for its operation
The RateLimiter integrates seamlessly with dispatchers like MemoryAdaptiveDispatcher and SemaphoreDispatcher, ensuring requests are paced correctly without user intervention. Its internal mechanisms manage delays and retries to avoid overwhelming servers while maximizing efficiency.

2.2 Crawler Monitor
The CrawlerMonitor provides real-time visibility into crawling operations:

from crawl4ai import CrawlerMonitor, DisplayMode
monitor = CrawlerMonitor(
    # Maximum rows in live display
    max_visible_rows=15,          

    # DETAILED or AGGREGATED view
    display_mode=DisplayMode.DETAILED  
)
Display Modes:

DETAILED: Shows individual task status, memory usage, and timing
AGGREGATED: Displays summary statistics and overall progress
3. Available Dispatchers
3.1 MemoryAdaptiveDispatcher (Default)
Automatically manages concurrency based on system memory usage:

from crawl4ai.async_dispatcher import MemoryAdaptiveDispatcher

dispatcher = MemoryAdaptiveDispatcher(
    memory_threshold_percent=90.0,  # Pause if memory exceeds this
    check_interval=1.0,             # How often to check memory
    max_session_permit=10,          # Maximum concurrent tasks
    rate_limiter=RateLimiter(       # Optional rate limiting
        base_delay=(1.0, 2.0),
        max_delay=30.0,
        max_retries=2
    ),
    monitor=CrawlerMonitor(         # Optional monitoring
        max_visible_rows=15,
        display_mode=DisplayMode.DETAILED
    )
)
Constructor Parameters:

1. memory_threshold_percent (float, default: 90.0)
  Specifies the memory usage threshold (as a percentage). If system memory usage exceeds this value, the dispatcher pauses crawling to prevent system overload.

2. check_interval (float, default: 1.0)
  The interval (in seconds) at which the dispatcher checks system memory usage.

3. max_session_permit (int, default: 10)
  The maximum number of concurrent crawling tasks allowed. This ensures resource limits are respected while maintaining concurrency.

4. memory_wait_timeout (float, default: 300.0)
  Optional timeout (in seconds). If memory usage exceeds memory_threshold_percent for longer than this duration, a MemoryError is raised.

5. rate_limiter (RateLimiter, default: None)
  Optional rate-limiting logic to avoid server-side blocking (e.g., for handling 429 or 503 errors). See RateLimiter for details.

6. monitor (CrawlerMonitor, default: None)
  Optional monitoring for real-time task tracking and performance insights. See CrawlerMonitor for details.

3.2 SemaphoreDispatcher
Provides simple concurrency control with a fixed limit:

from crawl4ai.async_dispatcher import SemaphoreDispatcher

dispatcher = SemaphoreDispatcher(
    max_session_permit=20,         # Maximum concurrent tasks
    rate_limiter=RateLimiter(      # Optional rate limiting
        base_delay=(0.5, 1.0),
        max_delay=10.0
    ),
    monitor=CrawlerMonitor(        # Optional monitoring
        max_visible_rows=15,
        display_mode=DisplayMode.DETAILED
    )
)
Constructor Parameters:

1. max_session_permit (int, default: 20)
  The maximum number of concurrent crawling tasks allowed, irrespective of semaphore slots.

2. rate_limiter (RateLimiter, default: None)
  Optional rate-limiting logic to avoid overwhelming servers. See RateLimiter for details.

3. monitor (CrawlerMonitor, default: None)
  Optional monitoring for tracking task progress and resource usage. See CrawlerMonitor for details.

4. Usage Examples
4.1 Batch Processing (Default)
async def crawl_batch():
    browser_config = BrowserConfig(headless=True, verbose=False)
    run_config = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS,
        stream=False  # Default: get all results at once
    )

    dispatcher = MemoryAdaptiveDispatcher(
        memory_threshold_percent=70.0,
        check_interval=1.0,
        max_session_permit=10,
        monitor=CrawlerMonitor(
            display_mode=DisplayMode.DETAILED
        )
    )

    async with AsyncWebCrawler(config=browser_config) as crawler:
        # Get all results at once
        results = await crawler.arun_many(
            urls=urls,
            config=run_config,
            dispatcher=dispatcher
        )

        # Process all results after completion
        for result in results:
            if result.success:
                await process_result(result)
            else:
                print(f"Failed to crawl {result.url}: {result.error_message}")
Review:
- Purpose: Executes a batch crawl with all URLs processed together after crawling is complete.
- Dispatcher: Uses MemoryAdaptiveDispatcher to manage concurrency and system memory.
- Stream: Disabled (stream=False), so all results are collected at once for post-processing.
- Best Use Case: When you need to analyze results in bulk rather than individually during the crawl.

4.2 Streaming Mode
async def crawl_streaming():
    browser_config = BrowserConfig(headless=True, verbose=False)
    run_config = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS,
        stream=True  # Enable streaming mode
    )

    dispatcher = MemoryAdaptiveDispatcher(
        memory_threshold_percent=70.0,
        check_interval=1.0,
        max_session_permit=10,
        monitor=CrawlerMonitor(
            display_mode=DisplayMode.DETAILED
        )
    )

    async with AsyncWebCrawler(config=browser_config) as crawler:
        # Process results as they become available
        async for result in await crawler.arun_many(
            urls=urls,
            config=run_config,
            dispatcher=dispatcher
        ):
            if result.success:
                # Process each result immediately
                await process_result(result)
            else:
                print(f"Failed to crawl {result.url}: {result.error_message}")
Review:
- Purpose: Enables streaming to process results as soon as they’re available.
- Dispatcher: Uses MemoryAdaptiveDispatcher for concurrency and memory management.
- Stream: Enabled (stream=True), allowing real-time processing during crawling.
- Best Use Case: When you need to act on results immediately, such as for real-time analytics or progressive data storage.

4.3 Semaphore-based Crawling
async def crawl_with_semaphore(urls):
    browser_config = BrowserConfig(headless=True, verbose=False)
    run_config = CrawlerRunConfig(cache_mode=CacheMode.BYPASS)

    dispatcher = SemaphoreDispatcher(
        semaphore_count=5,
        rate_limiter=RateLimiter(
            base_delay=(0.5, 1.0),
            max_delay=10.0
        ),
        monitor=CrawlerMonitor(
            max_visible_rows=15,
            display_mode=DisplayMode.DETAILED
        )
    )

    async with AsyncWebCrawler(config=browser_config) as crawler:
        results = await crawler.arun_many(
            urls, 
            config=run_config,
            dispatcher=dispatcher
        )
        return results
Review:
- Purpose: Uses SemaphoreDispatcher to limit concurrency with a fixed number of slots.
- Dispatcher: Configured with a semaphore to control parallel crawling tasks.
- Rate Limiter: Prevents servers from being overwhelmed by pacing requests.
- Best Use Case: When you want precise control over the number of concurrent requests, independent of system memory.

4.4 Robots.txt Consideration
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode

async def main():
    urls = [
        "https://example1.com",
        "https://example2.com",
        "https://example3.com"
    ]

    config = CrawlerRunConfig(
        cache_mode=CacheMode.ENABLED,
        check_robots_txt=True,  # Will respect robots.txt for each URL
        semaphore_count=3      # Max concurrent requests
    )

    async with AsyncWebCrawler() as crawler:
        async for result in crawler.arun_many(urls, config=config):
            if result.success:
                print(f"Successfully crawled {result.url}")
            elif result.status_code == 403 and "robots.txt" in result.error_message:
                print(f"Skipped {result.url} - blocked by robots.txt")
            else:
                print(f"Failed to crawl {result.url}: {result.error_message}")

if __name__ == "__main__":
    asyncio.run(main())
Review:
- Purpose: Ensures compliance with robots.txt rules for ethical and legal web crawling.
- Configuration: Set check_robots_txt=True to validate each URL against robots.txt before crawling.
- Dispatcher: Handles requests with concurrency limits (semaphore_count=3).
- Best Use Case: When crawling websites that strictly enforce robots.txt policies or for responsible crawling practices.

5. Dispatch Results
Each crawl result includes dispatch information:

@dataclass
class DispatchResult:
    task_id: str
    memory_usage: float
    peak_memory: float
    start_time: datetime
    end_time: datetime
    error_message: str = ""
Access via result.dispatch_result:

for result in results:
    if result.success:
        dr = result.dispatch_result
        print(f"URL: {result.url}")
        print(f"Memory: {dr.memory_usage:.1f}MB")
        print(f"Duration: {dr.end_time - dr.start_time}")
6. Summary
1. Two Dispatcher Types:

MemoryAdaptiveDispatcher (default): Dynamic concurrency based on memory
SemaphoreDispatcher: Fixed concurrency limit
2. Optional Components:

RateLimiter: Smart request pacing and backoff
CrawlerMonitor: Real-time progress visualization
3. Key Benefits:

Automatic memory management
Built-in rate limiting
Live progress monitoring
Flexible concurrency control
Choose the dispatcher that best fits your needs:

MemoryAdaptiveDispatcher: For large crawls or limited resources
SemaphoreDispatcher: For simple, fixed-concurrency scenarios
--------------
Extracting JSON (No LLM, so its most efficient we should use it)
One of Crawl4AI’s most powerful features is extracting structured JSON from websites without relying on large language models. By defining a schema with CSS or XPath selectors, you can extract data instantly—even from complex or nested HTML structures—without the cost, latency, or environmental impact of an LLM.

Why avoid LLM for basic extractions?

1. Faster & Cheaper: No API calls or GPU overhead.
2. Lower Carbon Footprint: LLM inference can be energy-intensive. A well-defined schema is practically carbon-free.
3. Precise & Repeatable: CSS/XPath selectors do exactly what you specify. LLM outputs can vary or hallucinate.
4. Scales Readily: For thousands of pages, schema-based extraction runs quickly and in parallel.

Below, we’ll explore how to craft these schemas and use them with JsonCssExtractionStrategy (or JsonXPathExtractionStrategy if you prefer XPath). We’ll also highlight advanced features like nested fields and base element attributes.

1. Intro to Schema-Based Extraction
A schema defines:

A base selector that identifies each “container” element on the page (e.g., a product row, a blog post card).
2. Fields describing which CSS/XPath selectors to use for each piece of data you want to capture (text, attribute, HTML block, etc.).
3. Nested or list types for repeated or hierarchical structures.
For example, if you have a list of products, each one might have a name, price, reviews, and “related products.” This approach is faster and more reliable than an LLM for consistent, structured pages.

2. Simple Example: Crypto Prices
Let’s begin with a simple schema-based extraction using the JsonCssExtractionStrategy. Below is a snippet that extracts cryptocurrency prices from a site (similar to the legacy Coinbase example). Notice we don’t call any LLM:

import json
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def extract_crypto_prices():
    # 1. Define a simple extraction schema
    schema = {
        "name": "Crypto Prices",
        "baseSelector": "div.crypto-row",    # Repeated elements
        "fields": [
            {
                "name": "coin_name",
                "selector": "h2.coin-name",
                "type": "text"
            },
            {
                "name": "price",
                "selector": "span.coin-price",
                "type": "text"
            }
        ]
    }

    # 2. Create the extraction strategy
    extraction_strategy = JsonCssExtractionStrategy(schema, verbose=True)

    # 3. Set up your crawler config (if needed)
    config = CrawlerRunConfig(
        # e.g., pass js_code or wait_for if the page is dynamic
        # wait_for="css:.crypto-row:nth-child(20)"
        cache_mode = CacheMode.BYPASS,
        extraction_strategy=extraction_strategy,
    )

    async with AsyncWebCrawler(verbose=True) as crawler:
        # 4. Run the crawl and extraction
        result = await crawler.arun(
            url="https://example.com/crypto-prices",

            config=config
        )

        if not result.success:
            print("Crawl failed:", result.error_message)
            return

        # 5. Parse the extracted JSON
        data = json.loads(result.extracted_content)
        print(f"Extracted {len(data)} coin entries")
        print(json.dumps(data[0], indent=2) if data else "No data found")

asyncio.run(extract_crypto_prices())
Highlights:

baseSelector: Tells us where each “item” (crypto row) is.
fields: Two fields (coin_name, price) using simple CSS selectors.
Each field defines a type (e.g., text, attribute, html, regex, etc.).
No LLM is needed, and the performance is near-instant for hundreds or thousands of items.

XPath Example with raw:// HTML
Below is a short example demonstrating XPath extraction plus the raw:// scheme. We’ll pass a dummy HTML directly (no network request) and define the extraction strategy in CrawlerRunConfig.

import json
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.extraction_strategy import JsonXPathExtractionStrategy

async def extract_crypto_prices_xpath():
    # 1. Minimal dummy HTML with some repeating rows
    dummy_html = """
    <html>
      <body>
        <div class='crypto-row'>
          <h2 class='coin-name'>Bitcoin</h2>
          <span class='coin-price'>$28,000</span>
        </div>
        <div class='crypto-row'>
          <h2 class='coin-name'>Ethereum</h2>
          <span class='coin-price'>$1,800</span>
        </div>
      </body>
    </html>
    """

    # 2. Define the JSON schema (XPath version)
    schema = {
        "name": "Crypto Prices via XPath",
        "baseSelector": "//div[@class='crypto-row']",
        "fields": [
            {
                "name": "coin_name",
                "selector": ".//h2[@class='coin-name']",
                "type": "text"
            },
            {
                "name": "price",
                "selector": ".//span[@class='coin-price']",
                "type": "text"
            }
        ]
    }

    # 3. Place the strategy in the CrawlerRunConfig
    config = CrawlerRunConfig(
        extraction_strategy=JsonXPathExtractionStrategy(schema, verbose=True)
    )

    # 4. Use raw:// scheme to pass dummy_html directly
    raw_url = f"raw://{dummy_html}"

    async with AsyncWebCrawler(verbose=True) as crawler:
        result = await crawler.arun(
            url=raw_url,
            config=config
        )

        if not result.success:
            print("Crawl failed:", result.error_message)
            return

        data = json.loads(result.extracted_content)
        print(f"Extracted {len(data)} coin rows")
        if data:
            print("First item:", data[0])

asyncio.run(extract_crypto_prices_xpath())
Key Points:

1. JsonXPathExtractionStrategy is used instead of JsonCssExtractionStrategy.
2. baseSelector and each field’s "selector" use XPath instead of CSS.
3. raw:// lets us pass dummy_html with no real network request—handy for local testing.
4. Everything (including the extraction strategy) is in CrawlerRunConfig.

That’s how you keep the config self-contained, illustrate XPath usage, and demonstrate the raw scheme for direct HTML input—all while avoiding the old approach of passing extraction_strategy directly to arun().

3. Advanced Schema & Nested Structures
Real sites often have nested or repeated data—like categories containing products, which themselves have a list of reviews or features. For that, we can define nested or list (and even nested_list) fields.

Sample E-Commerce HTML
We have a sample e-commerce HTML file on GitHub (example):

https://gist.githubusercontent.com/githubusercontent/2d7b8ba3cd8ab6cf3c8da771ddb36878/raw/1ae2f90c6861ce7dd84cc50d3df9920dee5e1fd2/sample_ecommerce.html
This snippet includes categories, products, features, reviews, and related items. Let’s see how to define a schema that fully captures that structure without LLM.
schema = {
    "name": "E-commerce Product Catalog",
    "baseSelector": "div.category",
    # (1) We can define optional baseFields if we want to extract attributes 
    # from the category container
    "baseFields": [
        {"name": "data_cat_id", "type": "attribute", "attribute": "data-cat-id"}, 
    ],
    "fields": [
        {
            "name": "category_name",
            "selector": "h2.category-name",
            "type": "text"
        },
        {
            "name": "products",
            "selector": "div.product",
            "type": "nested_list",    # repeated sub-objects
            "fields": [
                {
                    "name": "name",
                    "selector": "h3.product-name",
                    "type": "text"
                },
                {
                    "name": "price",
                    "selector": "p.product-price",
                    "type": "text"
                },
                {
                    "name": "details",
                    "selector": "div.product-details",
                    "type": "nested",  # single sub-object
                    "fields": [
                        {
                            "name": "brand",
                            "selector": "span.brand",
                            "type": "text"
                        },
                        {
                            "name": "model",
                            "selector": "span.model",
                            "type": "text"
                        }
                    ]
                },
                {
                    "name": "features",
                    "selector": "ul.product-features li",
                    "type": "list",
                    "fields": [
                        {"name": "feature", "type": "text"} 
                    ]
                },
                {
                    "name": "reviews",
                    "selector": "div.review",
                    "type": "nested_list",
                    "fields": [
                        {
                            "name": "reviewer", 
                            "selector": "span.reviewer", 
                            "type": "text"
                        },
                        {
                            "name": "rating", 
                            "selector": "span.rating", 
                            "type": "text"
                        },
                        {
                            "name": "comment", 
                            "selector": "p.review-text", 
                            "type": "text"
                        }
                    ]
                },
                {
                    "name": "related_products",
                    "selector": "ul.related-products li",
                    "type": "list",
                    "fields": [
                        {
                            "name": "name", 
                            "selector": "span.related-name", 
                            "type": "text"
                        },
                        {
                            "name": "price", 
                            "selector": "span.related-price", 
                            "type": "text"
                        }
                    ]
                }
            ]
        }
    ]
}
Key Takeaways:

Nested vs. List:
type: "nested" means a single sub-object (like details).
type: "list" means multiple items that are simple dictionaries or single text fields.
type: "nested_list" means repeated complex objects (like products or reviews).
Base Fields: We can extract attributes from the container element via "baseFields". For instance, "data_cat_id" might be data-cat-id="elect123".
Transforms: We can also define a transform if we want to lower/upper case, strip whitespace, or even run a custom function.
Running the Extraction
import json
import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

ecommerce_schema = {
    # ... the advanced schema from above ...
}

async def extract_ecommerce_data():
    strategy = JsonCssExtractionStrategy(ecommerce_schema, verbose=True)

    config = CrawlerRunConfig()

    async with AsyncWebCrawler(verbose=True) as crawler:
        result = await crawler.arun(
            url="https://gist.githubusercontent.com/githubusercontent/2d7b8ba3cd8ab6cf3c8da771ddb36878/raw/1ae2f90c6861ce7dd84cc50d3df9920dee5e1fd2/sample_ecommerce.html",
            extraction_strategy=strategy,
            config=config
        )

        if not result.success:
            print("Crawl failed:", result.error_message)
            return

        # Parse the JSON output
        data = json.loads(result.extracted_content)
        print(json.dumps(data, indent=2) if data else "No data found.")

asyncio.run(extract_ecommerce_data())
If all goes well, you get a structured JSON array with each “category,” containing an array of products. Each product includes details, features, reviews, etc. All of that without an LLM.

4. Why “No LLM” Is Often Better
1. Zero Hallucination: Schema-based extraction doesn’t guess text. It either finds it or not.
2. Guaranteed Structure: The same schema yields consistent JSON across many pages, so your downstream pipeline can rely on stable keys.
3. Speed: LLM-based extraction can be 10–1000x slower for large-scale crawling.
4. Scalable: Adding or updating a field is a matter of adjusting the schema, not re-tuning a model.

When might you consider an LLM? Possibly if the site is extremely unstructured or you want AI summarization. But always try a schema approach first for repeated or consistent data patterns.

5. Base Element Attributes & Additional Fields
It’s easy to extract attributes (like href, src, or data-xxx) from your base or nested elements using:

{
  "name": "href",
  "type": "attribute",
  "attribute": "href",
  "default": null
}
You can define them in baseFields (extracted from the main container element) or in each field’s sub-lists. This is especially helpful if you need an item’s link or ID stored in the parent <div>.

6. Putting It All Together: Larger Example
Consider a blog site. We have a schema that extracts the URL from each post card (via baseFields with an "attribute": "href"), plus the title, date, summary, and author:

schema = {
  "name": "Blog Posts",
  "baseSelector": "a.blog-post-card",
  "baseFields": [
    {"name": "post_url", "type": "attribute", "attribute": "href"}
  ],
  "fields": [
    {"name": "title", "selector": "h2.post-title", "type": "text", "default": "No Title"},
    {"name": "date", "selector": "time.post-date", "type": "text", "default": ""},
    {"name": "summary", "selector": "p.post-summary", "type": "text", "default": ""},
    {"name": "author", "selector": "span.post-author", "type": "text", "default": ""}
  ]
}
Then run with JsonCssExtractionStrategy(schema) to get an array of blog post objects, each with "post_url", "title", "date", "summary", "author".

7. Tips & Best Practices
1. Inspect the DOM in Chrome DevTools or Firefox’s Inspector to find stable selectors.
2. Start Simple: Verify you can extract a single field. Then add complexity like nested objects or lists.
3. Test your schema on partial HTML or a test page before a big crawl.
4. Combine with JS Execution if the site loads content dynamically. You can pass js_code or wait_for in CrawlerRunConfig.
5. Look at Logs when verbose=True: if your selectors are off or your schema is malformed, it’ll often show warnings.
6. Use baseFields if you need attributes from the container element (e.g., href, data-id), especially for the “parent” item.
7. Performance: For large pages, make sure your selectors are as narrow as possible.

8. Schema Generation Utility
While manually crafting schemas is powerful and precise, Crawl4AI now offers a convenient utility to automatically generate extraction schemas using LLM. This is particularly useful when:

You're dealing with a new website structure and want a quick starting point
You need to extract complex nested data structures
You want to avoid the learning curve of CSS/XPath selector syntax
Using the Schema Generator
The schema generator is available as a static method on both JsonCssExtractionStrategy and JsonXPathExtractionStrategy. You can choose between OpenAI's GPT-4 or the open-source Ollama for schema generation:

from crawl4ai.extraction_strategy import JsonCssExtractionStrategy, JsonXPathExtractionStrategy
from crawl4ai import LLMConfig

# Sample HTML with product information
html = """
<div class="product-card">
    <h2 class="title">Gaming Laptop</h2>
    <div class="price">$999.99</div>
    <div class="specs">
        <ul>
            <li>16GB RAM</li>
            <li>1TB SSD</li>
        </ul>
    </div>
</div>
"""

# Option 1: Using OpenAI (requires API token)
css_schema = JsonCssExtractionStrategy.generate_schema(
    html,
    schema_type="css", 
    llm_config = LLMConfig(provider="openai/gpt-4o",api_token="your-openai-token")
)

# Option 2: Using Ollama (open source, no token needed)
xpath_schema = JsonXPathExtractionStrategy.generate_schema(
    html,
    schema_type="xpath",
    llm_config = LLMConfig(provider="ollama/llama3.3", api_token=None)  # Not needed for Ollama
)

# Use the generated schema for fast, repeated extractions
strategy = JsonCssExtractionStrategy(css_schema)
LLM Provider Options
OpenAI GPT-4 (openai/gpt4o)
Default provider
Requires an API token
Generally provides more accurate schemas
Set via environment variable: OPENAI_API_KEY

Ollama (ollama/llama3.3)

Open source alternative
No API token required
Self-hosted option
Good for development and testing
Benefits of Schema Generation
One-Time Cost: While schema generation uses LLM, it's a one-time cost. The generated schema can be reused for unlimited extractions without further LLM calls.
Smart Pattern Recognition: The LLM analyzes the HTML structure and identifies common patterns, often producing more robust selectors than manual attempts.
Automatic Nesting: Complex nested structures are automatically detected and properly represented in the schema.
Learning Tool: The generated schemas serve as excellent examples for learning how to write your own schemas.
Best Practices
Review Generated Schemas: While the generator is smart, always review and test the generated schema before using it in production.
Provide Representative HTML: The better your sample HTML represents the overall structure, the more accurate the generated schema will be.
Consider Both CSS and XPath: Try both schema types and choose the one that works best for your specific case.
Cache Generated Schemas: Since generation uses LLM, save successful schemas for reuse.
API Token Security: Never hardcode API tokens. Use environment variables or secure configuration management.
Choose Provider Wisely:
Use OpenAI for production-quality schemas
Use Ollama for development, testing, or when you need a self-hosted solution
That's it for Extracting JSON (No LLM)! You've seen how schema-based approaches (either CSS or XPath) can handle everything from simple lists to deeply nested product catalogs—instantly, with minimal overhead. Enjoy building robust scrapers that produce consistent, structured JSON for your data pipelines!

9. Conclusion
With JsonCssExtractionStrategy (or JsonXPathExtractionStrategy), you can build powerful, LLM-free pipelines that:

Scrape any consistent site for structured data.
Support nested objects, repeating lists, or advanced transformations.
Scale to thousands of pages quickly and reliably.
Next Steps:

Combine your extracted JSON with advanced filtering or summarization in a second pass if needed.
For dynamic pages, combine strategies with js_code or infinite scroll hooking to ensure all content is loaded.
Remember: For repeated, structured data, you don’t need to pay for or wait on an LLM. A well-crafted schema plus CSS or XPath gets you the data faster, cleaner, and cheaper—the real power of Crawl4AI.

Last Updated: 2025-01-01

That’s it for Extracting JSON (No LLM)! You’ve seen how schema-based approaches (either CSS or XPath) can handle everything from simple lists to deeply nested product catalogs—instantly, with minimal overhead. Enjoy building robust scrapers that produce consistent, structured JSON for your data pipelines!


----------------
AsyncWebCrawler
1. Constructor Overview
2. Lifecycle: Start/Close or Context Manager
3. Primary Method: arun()
4. Batch Processing: arun_many()
7. Best Practices & Migration Notes
8. Summary
AsyncWebCrawler
The AsyncWebCrawler is the core class for asynchronous web crawling in Crawl4AI. You typically create it once, optionally customize it with a BrowserConfig (e.g., headless, user agent), then run multiple arun() calls with different CrawlerRunConfig objects.

Recommended usage:

1. Create a BrowserConfig for global browser settings. 

2. Instantiate AsyncWebCrawler(config=browser_config). 

3. Use the crawler in an async context manager (async with) or manage start/close manually. 

4. Call arun(url, config=crawler_run_config) for each page you want.

1. Constructor Overview
class AsyncWebCrawler:
    def __init__(
        self,
        crawler_strategy: Optional[AsyncCrawlerStrategy] = None,
        config: Optional[BrowserConfig] = None,
        always_bypass_cache: bool = False,           # deprecated
        always_by_pass_cache: Optional[bool] = None, # also deprecated
        base_directory: str = ...,
        thread_safe: bool = False,
        **kwargs,
    ):
        """
        Create an AsyncWebCrawler instance.

        Args:
            crawler_strategy: 
                (Advanced) Provide a custom crawler strategy if needed.
            config: 
                A BrowserConfig object specifying how the browser is set up.
            always_bypass_cache: 
                (Deprecated) Use CrawlerRunConfig.cache_mode instead.
            base_directory:     
                Folder for storing caches/logs (if relevant).
            thread_safe: 
                If True, attempts some concurrency safeguards. Usually False.
            **kwargs: 
                Additional legacy or debugging parameters.
        """
    )

### Typical Initialization

```python
from crawl4ai import AsyncWebCrawler, BrowserConfig

browser_cfg = BrowserConfig(
    browser_type="chromium",
    headless=True,
    verbose=True
)

crawler = AsyncWebCrawler(config=browser_cfg)
Notes:

Legacy parameters like always_bypass_cache remain for backward compatibility, but prefer to set caching in CrawlerRunConfig.
2. Lifecycle: Start/Close or Context Manager
2.1 Context Manager (Recommended)
async with AsyncWebCrawler(config=browser_cfg) as crawler:
    result = await crawler.arun("https://example.com")
    # The crawler automatically starts/closes resources
When the async with block ends, the crawler cleans up (closes the browser, etc.).

2.2 Manual Start & Close
crawler = AsyncWebCrawler(config=browser_cfg)
await crawler.start()

result1 = await crawler.arun("https://example.com")
result2 = await crawler.arun("https://another.com")

await crawler.close()
Use this style if you have a long-running application or need full control of the crawler’s lifecycle.

3. Primary Method: arun()
async def arun(
    self,
    url: str,
    config: Optional[CrawlerRunConfig] = None,
    # Legacy parameters for backward compatibility...
) -> CrawlResult:
    ...
3.1 New Approach
You pass a CrawlerRunConfig object that sets up everything about a crawl—content filtering, caching, session reuse, JS code, screenshots, etc.

import asyncio
from crawl4ai import CrawlerRunConfig, CacheMode

run_cfg = CrawlerRunConfig(
    cache_mode=CacheMode.BYPASS,
    css_selector="main.article",
    word_count_threshold=10,
    screenshot=True
)

async with AsyncWebCrawler(config=browser_cfg) as crawler:
    result = await crawler.arun("https://example.com/news", config=run_cfg)
    print("Crawled HTML length:", len(result.cleaned_html))
    if result.screenshot:
        print("Screenshot base64 length:", len(result.screenshot))
3.2 Legacy Parameters Still Accepted
For backward compatibility, arun() can still accept direct arguments like css_selector=..., word_count_threshold=..., etc., but we strongly advise migrating them into a CrawlerRunConfig.

4. Batch Processing: arun_many()
async def arun_many(
    self,
    urls: List[str],
    config: Optional[CrawlerRunConfig] = None,
    # Legacy parameters maintained for backwards compatibility...
) -> List[CrawlResult]:
    """
    Process multiple URLs with intelligent rate limiting and resource monitoring.
    """
4.1 Resource-Aware Crawling
The arun_many() method now uses an intelligent dispatcher that:

Monitors system memory usage
Implements adaptive rate limiting
Provides detailed progress monitoring
Manages concurrent crawls efficiently
4.2 Example Usage
Check page Multi-url Crawling for a detailed example of how to use arun_many().

### 4.3 Key Features

1. **Rate Limiting**

   - Automatic delay between requests
   - Exponential backoff on rate limit detection
   - Domain-specific rate limiting
   - Configurable retry strategy

2. **Resource Monitoring**

   - Memory usage tracking
   - Adaptive concurrency based on system load
   - Automatic pausing when resources are constrained

3. **Progress Monitoring**

   - Detailed or aggregated progress display
   - Real-time status updates
   - Memory usage statistics

4. **Error Handling**

   - Graceful handling of rate limits
   - Automatic retries with backoff
   - Detailed error reporting

---

## 5. `CrawlResult` Output

Each `arun()` returns a **`CrawlResult`** containing:

- `url`: Final URL (if redirected).
- `html`: Original HTML.
- `cleaned_html`: Sanitized HTML.
- `markdown_v2`: Deprecated. Instead just use regular `markdown`
- `extracted_content`: If an extraction strategy was used (JSON for CSS/LLM strategies).
- `screenshot`, `pdf`: If screenshots/PDF requested.
- `media`, `links`: Information about discovered images/links.
- `success`, `error_message`: Status info.

For details, see [CrawlResult doc](./crawl-result.md).

---

## 6. Quick Example

Below is an example hooking it all together:

```python
import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy
import json

async def main():
    # 1. Browser config
    browser_cfg = BrowserConfig(
        browser_type="firefox",
        headless=False,
        verbose=True
    )

    # 2. Run config
    schema = {
        "name": "Articles",
        "baseSelector": "article.post",
        "fields": [
            {
                "name": "title", 
                "selector": "h2", 
                "type": "text"
            },
            {
                "name": "url", 
                "selector": "a", 
                "type": "attribute", 
                "attribute": "href"
            }
        ]
    }

    run_cfg = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS,
        extraction_strategy=JsonCssExtractionStrategy(schema),
        word_count_threshold=15,
        remove_overlay_elements=True,
        wait_for="css:.post"  # Wait for posts to appear
    )

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(
            url="https://example.com/blog",
            config=run_cfg
        )

        if result.success:
            print("Cleaned HTML length:", len(result.cleaned_html))
            if result.extracted_content:
                articles = json.loads(result.extracted_content)
                print("Extracted articles:", articles[:2])
        else:
            print("Error:", result.error_message)

asyncio.run(main())
Explanation:

We define a BrowserConfig with Firefox, no headless, and verbose=True. 
We define a CrawlerRunConfig that bypasses cache, uses a CSS extraction schema, has a word_count_threshold=15, etc. 
We pass them to AsyncWebCrawler(config=...) and arun(url=..., config=...).
7. Best Practices & Migration Notes
1. Use BrowserConfig for global settings about the browser’s environment.  2. Use CrawlerRunConfig for per-crawl logic (caching, content filtering, extraction strategies, wait conditions).  3. Avoid legacy parameters like css_selector or word_count_threshold directly in arun(). Instead:

run_cfg = CrawlerRunConfig(css_selector=".main-content", word_count_threshold=20)
result = await crawler.arun(url="...", config=run_cfg)
4. Context Manager usage is simplest unless you want a persistent crawler across many calls.

8. Summary
AsyncWebCrawler is your entry point to asynchronous crawling:

Constructor accepts BrowserConfig (or defaults). 
arun(url, config=CrawlerRunConfig) is the main method for single-page crawls. 
arun_many(urls, config=CrawlerRunConfig) handles concurrency across multiple URLs. 
For advanced lifecycle control, use start() and close() explicitly. 
Migration:

If you used AsyncWebCrawler(browser_type="chromium", css_selector="..."), move browser settings to BrowserConfig(...) and content/crawl logic to CrawlerRunConfig(...).
This modular approach ensures your code is clean, scalable, and easy to maintain. For any advanced or rarely used parameters, see the BrowserConfig docs.

-----------------
arun() Parameter Guide (New Approach)
1. Core Usage
2. Cache Control
3. Content Processing & Selection
4. Page Navigation & Timing
5. Session Management
6. Screenshot, PDF & Media Options
7. Extraction Strategy
8. Comprehensive Example
9. Best Practices
10. Conclusion
arun() Parameter Guide (New Approach)
In Crawl4AI’s latest configuration model, nearly all parameters that once went directly to arun() are now part of CrawlerRunConfig. When calling arun(), you provide:

await crawler.arun(
    url="https://example.com",  
    config=my_run_config
)
Below is an organized look at the parameters that can go inside CrawlerRunConfig, divided by their functional areas. For Browser settings (e.g., headless, browser_type), see BrowserConfig.

1. Core Usage
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode

async def main():
    run_config = CrawlerRunConfig(
        verbose=True,            # Detailed logging
        cache_mode=CacheMode.ENABLED,  # Use normal read/write cache
        check_robots_txt=True,   # Respect robots.txt rules
        # ... other parameters
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(
            url="https://example.com",
            config=run_config
        )

        # Check if blocked by robots.txt
        if not result.success and result.status_code == 403:
            print(f"Error: {result.error_message}")
Key Fields: - verbose=True logs each crawl step.  - cache_mode decides how to read/write the local crawl cache.

2. Cache Control
cache_mode (default: CacheMode.ENABLED)
Use a built-in enum from CacheMode:

ENABLED: Normal caching—reads if available, writes if missing.
DISABLED: No caching—always refetch pages.
READ_ONLY: Reads from cache only; no new writes.
WRITE_ONLY: Writes to cache but doesn’t read existing data.
BYPASS: Skips reading cache for this crawl (though it might still write if set up that way).
run_config = CrawlerRunConfig(
    cache_mode=CacheMode.BYPASS
)
Additional flags:

bypass_cache=True acts like CacheMode.BYPASS.
disable_cache=True acts like CacheMode.DISABLED.
no_cache_read=True acts like CacheMode.WRITE_ONLY.
no_cache_write=True acts like CacheMode.READ_ONLY.
3. Content Processing & Selection
3.1 Text Processing
run_config = CrawlerRunConfig(
    word_count_threshold=10,   # Ignore text blocks <10 words
    only_text=False,           # If True, tries to remove non-text elements
    keep_data_attributes=False # Keep or discard data-* attributes
)
3.2 Content Selection
run_config = CrawlerRunConfig(
    css_selector=".main-content",  # Focus on .main-content region only
    excluded_tags=["form", "nav"], # Remove entire tag blocks
    remove_forms=True,             # Specifically strip <form> elements
    remove_overlay_elements=True,  # Attempt to remove modals/popups
)
3.3 Link Handling
run_config = CrawlerRunConfig(
    exclude_external_links=True,         # Remove external links from final content
    exclude_social_media_links=True,     # Remove links to known social sites
    exclude_domains=["ads.example.com"], # Exclude links to these domains
    exclude_social_media_domains=["facebook.com","twitter.com"], # Extend the default list
)
3.4 Media Filtering
run_config = CrawlerRunConfig(
    exclude_external_images=True  # Strip images from other domains
)
4. Page Navigation & Timing
4.1 Basic Browser Flow
run_config = CrawlerRunConfig(
    wait_for="css:.dynamic-content", # Wait for .dynamic-content
    delay_before_return_html=2.0,    # Wait 2s before capturing final HTML
    page_timeout=60000,             # Navigation & script timeout (ms)
)
Key Fields:

wait_for:
"css:selector" or
"js:() => boolean"
e.g. js:() => document.querySelectorAll('.item').length > 10.

mean_delay & max_range: define random delays for arun_many() calls. 

semaphore_count: concurrency limit when crawling multiple URLs.
4.2 JavaScript Execution
run_config = CrawlerRunConfig(
    js_code=[
        "window.scrollTo(0, document.body.scrollHeight);",
        "document.querySelector('.load-more')?.click();"
    ],
    js_only=False
)
js_code can be a single string or a list of strings. 
js_only=True means “I’m continuing in the same session with new JS steps, no new full navigation.”
4.3 Anti-Bot
run_config = CrawlerRunConfig(
    magic=True,
    simulate_user=True,
    override_navigator=True
)
- magic=True tries multiple stealth features.  - simulate_user=True mimics mouse movements or random delays.  - override_navigator=True fakes some navigator properties (like user agent checks).
5. Session Management
session_id:

run_config = CrawlerRunConfig(
    session_id="my_session123"
)
If re-used in subsequent arun() calls, the same tab/page context is continued (helpful for multi-step tasks or stateful browsing).
6. Screenshot, PDF & Media Options
run_config = CrawlerRunConfig(
    screenshot=True,             # Grab a screenshot as base64
    screenshot_wait_for=1.0,     # Wait 1s before capturing
    pdf=True,                    # Also produce a PDF
    image_description_min_word_threshold=5,  # If analyzing alt text
    image_score_threshold=3,                # Filter out low-score images
)
Where they appear: - result.screenshot → Base64 screenshot string. - result.pdf → Byte array with PDF data.
7. Extraction Strategy
For advanced data extraction (CSS/LLM-based), set extraction_strategy:

run_config = CrawlerRunConfig(
    extraction_strategy=my_css_or_llm_strategy
)
The extracted data will appear in result.extracted_content.

8. Comprehensive Example
Below is a snippet combining many parameters:

import asyncio
from crawl4ai import AsyncWebCrawler, CrawlerRunConfig, CacheMode
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

async def main():
    # Example schema
    schema = {
        "name": "Articles",
        "baseSelector": "article.post",
        "fields": [
            {"name": "title", "selector": "h2", "type": "text"},
            {"name": "link",  "selector": "a",  "type": "attribute", "attribute": "href"}
        ]
    }

    run_config = CrawlerRunConfig(
        # Core
        verbose=True,
        cache_mode=CacheMode.ENABLED,
        check_robots_txt=True,   # Respect robots.txt rules

        # Content
        word_count_threshold=10,
        css_selector="main.content",
        excluded_tags=["nav", "footer"],
        exclude_external_links=True,

        # Page & JS
        js_code="document.querySelector('.show-more')?.click();",
        wait_for="css:.loaded-block",
        page_timeout=30000,

        # Extraction
        extraction_strategy=JsonCssExtractionStrategy(schema),

        # Session
        session_id="persistent_session",

        # Media
        screenshot=True,
        pdf=True,

        # Anti-bot
        simulate_user=True,
        magic=True,
    )

    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun("https://example.com/posts", config=run_config)
        if result.success:
            print("HTML length:", len(result.cleaned_html))
            print("Extraction JSON:", result.extracted_content)
            if result.screenshot:
                print("Screenshot length:", len(result.screenshot))
            if result.pdf:
                print("PDF bytes length:", len(result.pdf))
        else:
            print("Error:", result.error_message)

if __name__ == "__main__":
    asyncio.run(main())
What we covered:

1. Crawling the main content region, ignoring external links.  2. Running JavaScript to click “.show-more”.  3. Waiting for “.loaded-block” to appear.  4. Generating a screenshot & PDF of the final page.  5. Extracting repeated “article.post” elements with a CSS-based extraction strategy.

9. Best Practices
1. Use BrowserConfig for global browser settings (headless, user agent).  2. Use CrawlerRunConfig to handle the specific crawl needs: content filtering, caching, JS, screenshot, extraction, etc.  3. Keep your parameters consistent in run configs—especially if you’re part of a large codebase with multiple crawls.  4. Limit large concurrency (semaphore_count) if the site or your system can’t handle it.  5. For dynamic pages, set js_code or scan_full_page so you load all content.

10. Conclusion
All parameters that used to be direct arguments to arun() now belong in CrawlerRunConfig. This approach:

Makes code clearer and more maintainable. 
Minimizes confusion about which arguments affect global vs. per-crawl behavior. 
Allows you to create reusable config objects for different pages or tasks.
For a full reference, check out the CrawlerRunConfig Docs. 

Happy crawling with your structured, flexible config approach!
-------------------
arun_many(...) Reference
Function Signature
Differences from arun()
Dispatcher Reference
Common Pitfalls
Conclusion
arun_many(...) Reference
Note: This function is very similar to arun() but focused on concurrent or batch crawling. If you’re unfamiliar with arun() usage, please read that doc first, then review this for differences.

Function Signature
async def arun_many(
    urls: Union[List[str], List[Any]],
    config: Optional[CrawlerRunConfig] = None,
    dispatcher: Optional[BaseDispatcher] = None,
    ...
) -> Union[List[CrawlResult], AsyncGenerator[CrawlResult, None]]:
    """
    Crawl multiple URLs concurrently or in batches.

    :param urls: A list of URLs (or tasks) to crawl.
    :param config: (Optional) A default `CrawlerRunConfig` applying to each crawl.
    :param dispatcher: (Optional) A concurrency controller (e.g. MemoryAdaptiveDispatcher).
    ...
    :return: Either a list of `CrawlResult` objects, or an async generator if streaming is enabled.
    """
Differences from arun()
1. Multiple URLs:

Instead of crawling a single URL, you pass a list of them (strings or tasks). 
The function returns either a list of CrawlResult or an async generator if streaming is enabled.
2. Concurrency & Dispatchers:

dispatcher param allows advanced concurrency control. 
If omitted, a default dispatcher (like MemoryAdaptiveDispatcher) is used internally. 
Dispatchers handle concurrency, rate limiting, and memory-based adaptive throttling (see Multi-URL Crawling).
3. Streaming Support:

Enable streaming by setting stream=True in your CrawlerRunConfig.
When streaming, use async for to process results as they become available.
Ideal for processing large numbers of URLs without waiting for all to complete.
4. Parallel Execution**:

arun_many() can run multiple requests concurrently under the hood. 
Each CrawlResult might also include a dispatch_result with concurrency details (like memory usage, start/end times).
Basic Example (Batch Mode)
# Minimal usage: The default dispatcher will be used
results = await crawler.arun_many(
    urls=["https://site1.com", "https://site2.com"],
    config=CrawlerRunConfig(stream=False)  # Default behavior
)

for res in results:
    if res.success:
        print(res.url, "crawled OK!")
    else:
        print("Failed:", res.url, "-", res.error_message)
Streaming Example
config = CrawlerRunConfig(
    stream=True,  # Enable streaming mode
    cache_mode=CacheMode.BYPASS
)

# Process results as they complete
async for result in await crawler.arun_many(
    urls=["https://site1.com", "https://site2.com", "https://site3.com"],
    config=config
):
    if result.success:
        print(f"Just completed: {result.url}")
        # Process each result immediately
        process_result(result)
With a Custom Dispatcher
dispatcher = MemoryAdaptiveDispatcher(
    memory_threshold_percent=70.0,
    max_session_permit=10
)
results = await crawler.arun_many(
    urls=["https://site1.com", "https://site2.com", "https://site3.com"],
    config=my_run_config,
    dispatcher=dispatcher
)
Key Points: - Each URL is processed by the same or separate sessions, depending on the dispatcher’s strategy. - dispatch_result in each CrawlResult (if using concurrency) can hold memory and timing info.  - If you need to handle authentication or session IDs, pass them in each individual task or within your run config.

Return Value
Either a list of CrawlResult objects, or an async generator if streaming is enabled. You can iterate to check result.success or read each item’s extracted_content, markdown, or dispatch_result.

Dispatcher Reference
MemoryAdaptiveDispatcher: Dynamically manages concurrency based on system memory usage. 
SemaphoreDispatcher: Fixed concurrency limit, simpler but less adaptive. 
For advanced usage or custom settings, see Multi-URL Crawling with Dispatchers.

Common Pitfalls
1. Large Lists: If you pass thousands of URLs, be mindful of memory or rate-limits. A dispatcher can help. 

2. Session Reuse: If you need specialized logins or persistent contexts, ensure your dispatcher or tasks handle sessions accordingly. 

3. Error Handling: Each CrawlResult might fail for different reasons—always check result.success or the error_message before proceeding.

Conclusion
Use arun_many() when you want to crawl multiple URLs simultaneously or in controlled parallel tasks. If you need advanced concurrency features (like memory-based adaptive throttling or complex rate-limiting), provide a dispatcher. Each result is a standard CrawlResult, possibly augmented with concurrency stats (dispatch_result) for deeper inspection. For more details on concurrency logic and dispatchers, see the Advanced Multi-URL Crawling docs.
------------------
1. BrowserConfig – Controlling the Browser
1.1 Parameter Highlights
2. CrawlerRunConfig – Controlling Each Crawl
2.1 Parameter Highlights
2.2 Helper Methods
2.3 Example Usage
3. LLMConfig - Setting up LLM providers
3.1 Parameters
3.2 Example Usage
4. Putting It All Together
1. BrowserConfig – Controlling the Browser
BrowserConfig focuses on how the browser is launched and behaves. This includes headless mode, proxies, user agents, and other environment tweaks.

from crawl4ai import AsyncWebCrawler, BrowserConfig

browser_cfg = BrowserConfig(
    browser_type="chromium",
    headless=True,
    viewport_width=1280,
    viewport_height=720,
    proxy="http://user:pass@proxy:8080",
    user_agent="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 Chrome/116.0.0.0 Safari/537.36",
)
1.1 Parameter Highlights
Parameter	Type / Default	What It Does
browser_type	"chromium", "firefox", "webkit"
(default: "chromium")	Which browser engine to use. "chromium" is typical for many sites, "firefox" or "webkit" for specialized tests.
headless	bool (default: True)	Headless means no visible UI. False is handy for debugging.
viewport_width	int (default: 1080)	Initial page width (in px). Useful for testing responsive layouts.
viewport_height	int (default: 600)	Initial page height (in px).
proxy	str (default: None)	Single-proxy URL if you want all traffic to go through it, e.g. "http://user:pass@proxy:8080".
proxy_config	dict (default: None)	For advanced or multi-proxy needs, specify details like {"server": "...", "username": "...", ...}.
use_persistent_context	bool (default: False)	If True, uses a persistent browser context (keep cookies, sessions across runs). Also sets use_managed_browser=True.
user_data_dir	str or None (default: None)	Directory to store user data (profiles, cookies). Must be set if you want permanent sessions.
ignore_https_errors	bool (default: True)	If True, continues despite invalid certificates (common in dev/staging).
java_script_enabled	bool (default: True)	Disable if you want no JS overhead, or if only static content is needed.
cookies	list (default: [])	Pre-set cookies, each a dict like {"name": "session", "value": "...", "url": "..."}.
headers	dict (default: {})	Extra HTTP headers for every request, e.g. {"Accept-Language": "en-US"}.
user_agent	str (default: Chrome-based UA)	Your custom or random user agent. user_agent_mode="random" can shuffle it.
light_mode	bool (default: False)	Disables some background features for performance gains.
text_mode	bool (default: False)	If True, tries to disable images/other heavy content for speed.
use_managed_browser	bool (default: False)	For advanced “managed” interactions (debugging, CDP usage). Typically set automatically if persistent context is on.
extra_args	list (default: [])	Additional flags for the underlying browser process, e.g. ["--disable-extensions"].
Tips: - Set headless=False to visually debug how pages load or how interactions proceed.
- If you need authentication storage or repeated sessions, consider use_persistent_context=True and specify user_data_dir.
- For large pages, you might need a bigger viewport_width and viewport_height to handle dynamic content.

2. CrawlerRunConfig – Controlling Each Crawl
While BrowserConfig sets up the environment, CrawlerRunConfig details how each crawl operation should behave: caching, content filtering, link or domain blocking, timeouts, JavaScript code, etc.

from crawl4ai import AsyncWebCrawler, CrawlerRunConfig

run_cfg = CrawlerRunConfig(
    wait_for="css:.main-content",
    word_count_threshold=15,
    excluded_tags=["nav", "footer"],
    exclude_external_links=True,
    stream=True,  # Enable streaming for arun_many()
)
2.1 Parameter Highlights
We group them by category.

A) Content Processing
Parameter	Type / Default	What It Does
word_count_threshold	int (default: ~200)	Skips text blocks below X words. Helps ignore trivial sections.
extraction_strategy	ExtractionStrategy (default: None)	If set, extracts structured data (CSS-based, LLM-based, etc.).
markdown_generator	MarkdownGenerationStrategy (None)	If you want specialized markdown output (citations, filtering, chunking, etc.).
css_selector	str (None)	Retains only the part of the page matching this selector. Affects the entire extraction process.
target_elements	List[str] (None)	List of CSS selectors for elements to focus on for markdown generation and data extraction, while still processing the entire page for links, media, etc. Provides more flexibility than css_selector.
excluded_tags	list (None)	Removes entire tags (e.g. ["script", "style"]).
excluded_selector	str (None)	Like css_selector but to exclude. E.g. "#ads, .tracker".
only_text	bool (False)	If True, tries to extract text-only content.
prettiify	bool (False)	If True, beautifies final HTML (slower, purely cosmetic).
keep_data_attributes	bool (False)	If True, preserve data-* attributes in cleaned HTML.
remove_forms	bool (False)	If True, remove all <form> elements.
B) Caching & Session
Parameter	Type / Default	What It Does
cache_mode	CacheMode or None	Controls how caching is handled (ENABLED, BYPASS, DISABLED, etc.). If None, typically defaults to ENABLED.
session_id	str or None	Assign a unique ID to reuse a single browser session across multiple arun() calls.
bypass_cache	bool (False)	If True, acts like CacheMode.BYPASS.
disable_cache	bool (False)	If True, acts like CacheMode.DISABLED.
no_cache_read	bool (False)	If True, acts like CacheMode.WRITE_ONLY (writes cache but never reads).
no_cache_write	bool (False)	If True, acts like CacheMode.READ_ONLY (reads cache but never writes).
Use these for controlling whether you read or write from a local content cache. Handy for large batch crawls or repeated site visits.

C) Page Navigation & Timing
Parameter	Type / Default	What It Does
wait_until	str (domcontentloaded)	Condition for navigation to “complete”. Often "networkidle" or "domcontentloaded".
page_timeout	int (60000 ms)	Timeout for page navigation or JS steps. Increase for slow sites.
wait_for	str or None	Wait for a CSS ("css:selector") or JS ("js:() => bool") condition before content extraction.
wait_for_images	bool (False)	Wait for images to load before finishing. Slows down if you only want text.
delay_before_return_html	float (0.1)	Additional pause (seconds) before final HTML is captured. Good for last-second updates.
check_robots_txt	bool (False)	Whether to check and respect robots.txt rules before crawling. If True, caches robots.txt for efficiency.
mean_delay and max_range	float (0.1, 0.3)	If you call arun_many(), these define random delay intervals between crawls, helping avoid detection or rate limits.
semaphore_count	int (5)	Max concurrency for arun_many(). Increase if you have resources for parallel crawls.
D) Page Interaction
Parameter	Type / Default	What It Does
js_code	str or list[str] (None)	JavaScript to run after load. E.g. "document.querySelector('button')?.click();".
js_only	bool (False)	If True, indicates we’re reusing an existing session and only applying JS. No full reload.
ignore_body_visibility	bool (True)	Skip checking if <body> is visible. Usually best to keep True.
scan_full_page	bool (False)	If True, auto-scroll the page to load dynamic content (infinite scroll).
scroll_delay	float (0.2)	Delay between scroll steps if scan_full_page=True.
process_iframes	bool (False)	Inlines iframe content for single-page extraction.
remove_overlay_elements	bool (False)	Removes potential modals/popups blocking the main content.
simulate_user	bool (False)	Simulate user interactions (mouse movements) to avoid bot detection.
override_navigator	bool (False)	Override navigator properties in JS for stealth.
magic	bool (False)	Automatic handling of popups/consent banners. Experimental.
adjust_viewport_to_content	bool (False)	Resizes viewport to match page content height.
If your page is a single-page app with repeated JS updates, set js_only=True in subsequent calls, plus a session_id for reusing the same tab.

E) Media Handling
Parameter	Type / Default	What It Does
screenshot	bool (False)	Capture a screenshot (base64) in result.screenshot.
screenshot_wait_for	float or None	Extra wait time before the screenshot.
screenshot_height_threshold	int (~20000)	If the page is taller than this, alternate screenshot strategies are used.
pdf	bool (False)	If True, returns a PDF in result.pdf.
image_description_min_word_threshold	int (~50)	Minimum words for an image’s alt text or description to be considered valid.
image_score_threshold	int (~3)	Filter out low-scoring images. The crawler scores images by relevance (size, context, etc.).
exclude_external_images	bool (False)	Exclude images from other domains.
F) Link/Domain Handling
Parameter	Type / Default	What It Does
exclude_social_media_domains	list (e.g. Facebook/Twitter)	A default list can be extended. Any link to these domains is removed from final output.
exclude_external_links	bool (False)	Removes all links pointing outside the current domain.
exclude_social_media_links	bool (False)	Strips links specifically to social sites (like Facebook or Twitter).
exclude_domains	list ([])	Provide a custom list of domains to exclude (like ["ads.com", "trackers.io"]).
Use these for link-level content filtering (often to keep crawls “internal” or to remove spammy domains).

G) Debug & Logging
Parameter	Type / Default	What It Does
verbose	bool (True)	Prints logs detailing each step of crawling, interactions, or errors.
log_console	bool (False)	Logs the page’s JavaScript console output if you want deeper JS debugging.
2.2 Helper Methods
Both BrowserConfig and CrawlerRunConfig provide a clone() method to create modified copies:

# Create a base configuration
base_config = CrawlerRunConfig(
    cache_mode=CacheMode.ENABLED,
    word_count_threshold=200
)

# Create variations using clone()
stream_config = base_config.clone(stream=True)
no_cache_config = base_config.clone(
    cache_mode=CacheMode.BYPASS,
    stream=True
)
The clone() method is particularly useful when you need slightly different configurations for different use cases, without modifying the original config.

2.3 Example Usage
import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode

async def main():
    # Configure the browser
    browser_cfg = BrowserConfig(
        headless=False,
        viewport_width=1280,
        viewport_height=720,
        proxy="http://user:pass@myproxy:8080",
        text_mode=True
    )

    # Configure the run
    run_cfg = CrawlerRunConfig(
        cache_mode=CacheMode.BYPASS,
        session_id="my_session",
        css_selector="main.article",
        excluded_tags=["script", "style"],
        exclude_external_links=True,
        wait_for="css:.article-loaded",
        screenshot=True,
        stream=True
    )

    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(
            url="https://example.com/news",
            config=run_cfg
        )
        if result.success:
            print("Final cleaned_html length:", len(result.cleaned_html))
            if result.screenshot:
                print("Screenshot captured (base64, length):", len(result.screenshot))
        else:
            print("Crawl failed:", result.error_message)

if __name__ == "__main__":
    asyncio.run(main())

## 2.4 Compliance & Ethics

| **Parameter**          | **Type / Default**      | **What It Does**                                                                                                    |
|-----------------------|-------------------------|----------------------------------------------------------------------------------------------------------------------|
| **`check_robots_txt`**| `bool` (False)          | When True, checks and respects robots.txt rules before crawling. Uses efficient caching with SQLite backend.          |
| **`user_agent`**      | `str` (None)            | User agent string to identify your crawler. Used for robots.txt checking when enabled.                                |

```python
run_config = CrawlerRunConfig(
    check_robots_txt=True,  # Enable robots.txt compliance
    user_agent="MyBot/1.0"  # Identify your crawler
)
3. LLMConfig - Setting up LLM providers
LLMConfig is useful to pass LLM provider config to strategies and functions that rely on LLMs to do extraction, filtering, schema generation etc. Currently it can be used in the following -

LLMExtractionStrategy
LLMContentFilter
JsonCssExtractionStrategy.generate_schema
JsonXPathExtractionStrategy.generate_schema
3.1 Parameters
Parameter	Type / Default	What It Does
provider	"ollama/llama3","groq/llama3-70b-8192","groq/llama3-8b-8192", "openai/gpt-4o-mini" ,"openai/gpt-4o","openai/o1-mini","openai/o1-preview","openai/o3-mini","openai/o3-mini-high","anthropic/claude-3-haiku-20240307","anthropic/claude-3-opus-20240229","anthropic/claude-3-sonnet-20240229","anthropic/claude-3-5-sonnet-20240620","gemini/gemini-pro","gemini/gemini-1.5-pro","gemini/gemini-2.0-flash","gemini/gemini-2.0-flash-exp","gemini/gemini-2.0-flash-lite-preview-02-05","deepseek/deepseek-chat"
(default: "openai/gpt-4o-mini")	Which LLM provoder to use.
api_token	1.Optional. When not provided explicitly, api_token will be read from environment variables based on provider. For example: If a gemini model is passed as provider then,"GEMINI_API_KEY" will be read from environment variables
2. API token of LLM provider
eg: api_token = "gsk_1ClHGGJ7Lpn4WGybR7vNWGdyb3FY7zXEw3SCiy0BAVM9lL8CQv"
3. Environment variable - use with prefix "env:"
eg:api_token = "env: GROQ_API_KEY"	API token to use for the given provider
base_url	Optional. Custom API endpoint	If your provider has a custom endpoint
3.2 Example Usage
llm_config = LLMConfig(provider="openai/gpt-4o-mini", api_token=os.getenv("OPENAI_API_KEY"))
4. Putting It All Together
Use BrowserConfig for global browser settings: engine, headless, proxy, user agent.
Use CrawlerRunConfig for each crawl’s context: how to filter content, handle caching, wait for dynamic elements, or run JS.
Pass both configs to AsyncWebCrawler (the BrowserConfig) and then to arun() (the CrawlerRunConfig).
Use LLMConfig for LLM provider configurations that can be used across all extraction, filtering, schema generation tasks. Can be used in - LLMExtractionStrategy, LLMContentFilter, JsonCssExtractionStrategy.generate_schema & JsonXPathExtractionStrategy.generate_schema
# Create a modified copy with the clone() method
stream_cfg = run_cfg.clone(
    stream=True,
    cache_mode=CacheMode.BYPASS
)
------------------------
CrawlResult Reference
1. Basic Crawl Info
2. Raw / Cleaned Content
3. Markdown Fields
4. Media & Links
5. Additional Fields
6. dispatch_result (optional)
7. Example: Accessing Everything
8. Key Points & Future
CrawlResult Reference
The CrawlResult class encapsulates everything returned after a single crawl operation. It provides the raw or processed content, details on links and media, plus optional metadata (like screenshots, PDFs, or extracted JSON).

Location: crawl4ai/crawler/models.py (for reference)

class CrawlResult(BaseModel):
    url: str
    html: str
    success: bool
    cleaned_html: Optional[str] = None
    media: Dict[str, List[Dict]] = {}
    links: Dict[str, List[Dict]] = {}
    downloaded_files: Optional[List[str]] = None
    screenshot: Optional[str] = None
    pdf : Optional[bytes] = None
    markdown: Optional[Union[str, MarkdownGenerationResult]] = None
    extracted_content: Optional[str] = None
    metadata: Optional[dict] = None
    error_message: Optional[str] = None
    session_id: Optional[str] = None
    response_headers: Optional[dict] = None
    status_code: Optional[int] = None
    ssl_certificate: Optional[SSLCertificate] = None
    dispatch_result: Optional[DispatchResult] = None
    ...
Below is a field-by-field explanation and possible usage patterns.

1. Basic Crawl Info
1.1 url (str)
What: The final crawled URL (after any redirects).
Usage:

print(result.url)  # e.g., "https://example.com/"
1.2 success (bool)
What: True if the crawl pipeline ended without major errors; False otherwise.
Usage:

if not result.success:
    print(f"Crawl failed: {result.error_message}")
1.3 status_code (Optional[int])
What: The page’s HTTP status code (e.g., 200, 404).
Usage:

if result.status_code == 404:
    print("Page not found!")
1.4 error_message (Optional[str])
What: If success=False, a textual description of the failure.
Usage:

if not result.success:
    print("Error:", result.error_message)
1.5 session_id (Optional[str])
What: The ID used for reusing a browser context across multiple calls.
Usage:

# If you used session_id="login_session" in CrawlerRunConfig, see it here:
print("Session:", result.session_id)
1.6 response_headers (Optional[dict])
What: Final HTTP response headers.
Usage:

if result.response_headers:
    print("Server:", result.response_headers.get("Server", "Unknown"))
1.7 ssl_certificate (Optional[SSLCertificate])
What: If fetch_ssl_certificate=True in your CrawlerRunConfig, result.ssl_certificate contains a SSLCertificate object describing the site’s certificate. You can export the cert in multiple formats (PEM/DER/JSON) or access its properties like issuer, subject, valid_from, valid_until, etc. Usage:

if result.ssl_certificate:
    print("Issuer:", result.ssl_certificate.issuer)
2. Raw / Cleaned Content
2.1 html (str)
What: The original unmodified HTML from the final page load.
Usage:

# Possibly large
print(len(result.html))
2.2 cleaned_html (Optional[str])
What: A sanitized HTML version—scripts, styles, or excluded tags are removed based on your CrawlerRunConfig.
Usage:

print(result.cleaned_html[:500])  # Show a snippet
2.3 fit_html (Optional[str])
What: If a content filter or heuristic (e.g., Pruning/BM25) modifies the HTML, the “fit” or post-filter version.
When: This is only present if your markdown_generator or content_filter produces it.
Usage:

if result.markdown.fit_html:
    print("High-value HTML content:", result.markdown.fit_html[:300])
3. Markdown Fields
3.1 The Markdown Generation Approach
Crawl4AI can convert HTML→Markdown, optionally including:

Raw markdown
Links as citations (with a references section)
Fit markdown if a content filter is used (like Pruning or BM25)
MarkdownGenerationResult includes: - raw_markdown (str): The full HTML→Markdown conversion.
- markdown_with_citations (str): Same markdown, but with link references as academic-style citations.
- references_markdown (str): The reference list or footnotes at the end.
- fit_markdown (Optional[str]): If content filtering (Pruning/BM25) was applied, the filtered “fit” text.
- fit_html (Optional[str]): The HTML that led to fit_markdown.

Usage:

if result.markdown:
    md_res = result.markdown
    print("Raw MD:", md_res.raw_markdown[:300])
    print("Citations MD:", md_res.markdown_with_citations[:300])
    print("References:", md_res.references_markdown)
    if md_res.fit_markdown:
        print("Pruned text:", md_res.fit_markdown[:300])
3.2 markdown (Optional[Union[str, MarkdownGenerationResult]])
What: Holds the MarkdownGenerationResult.
Usage:

print(result.markdown.raw_markdown[:200])
print(result.markdown.fit_markdown)
print(result.markdown.fit_html)
Important: “Fit” content (in fit_markdown/fit_html) exists in result.markdown, only if you used a filter (like PruningContentFilter or BM25ContentFilter) within a MarkdownGenerationStrategy.
4. Media & Links
4.1 media (Dict[str, List[Dict]])
What: Contains info about discovered images, videos, or audio. Typically keys: "images", "videos", "audios".
Common Fields in each item:

src (str): Media URL
alt or title (str): Descriptive text
score (float): Relevance score if the crawler’s heuristic found it “important”
desc or description (Optional[str]): Additional context extracted from surrounding text
Usage:

images = result.media.get("images", [])
for img in images:
    if img.get("score", 0) > 5:
        print("High-value image:", img["src"])
4.2 links (Dict[str, List[Dict]])
What: Holds internal and external link data. Usually two keys: "internal" and "external".
Common Fields:

href (str): The link target
text (str): Link text
title (str): Title attribute
context (str): Surrounding text snippet
domain (str): If external, the domain
Usage:

for link in result.links["internal"]:
    print(f"Internal link to {link['href']} with text {link['text']}")
5. Additional Fields
5.1 extracted_content (Optional[str])
What: If you used extraction_strategy (CSS, LLM, etc.), the structured output (JSON).
Usage:

if result.extracted_content:
    data = json.loads(result.extracted_content)
    print(data)
5.2 downloaded_files (Optional[List[str]])
What: If accept_downloads=True in your BrowserConfig + downloads_path, lists local file paths for downloaded items.
Usage:

if result.downloaded_files:
    for file_path in result.downloaded_files:
        print("Downloaded:", file_path)
5.3 screenshot (Optional[str])
What: Base64-encoded screenshot if screenshot=True in CrawlerRunConfig.
Usage:

import base64
if result.screenshot:
    with open("page.png", "wb") as f:
        f.write(base64.b64decode(result.screenshot))
5.4 pdf (Optional[bytes])
What: Raw PDF bytes if pdf=True in CrawlerRunConfig.
Usage:

if result.pdf:
    with open("page.pdf", "wb") as f:
        f.write(result.pdf)
5.5 metadata (Optional[dict])
What: Page-level metadata if discovered (title, description, OG data, etc.).
Usage:

if result.metadata:
    print("Title:", result.metadata.get("title"))
    print("Author:", result.metadata.get("author"))
6. dispatch_result (optional)
A DispatchResult object providing additional concurrency and resource usage information when crawling URLs in parallel (e.g., via arun_many() with custom dispatchers). It contains:

task_id: A unique identifier for the parallel task.
memory_usage (float): The memory (in MB) used at the time of completion.
peak_memory (float): The peak memory usage (in MB) recorded during the task’s execution.
start_time / end_time (datetime): Time range for this crawling task.
error_message (str): Any dispatcher- or concurrency-related error encountered.
# Example usage:
for result in results:
    if result.success and result.dispatch_result:
        dr = result.dispatch_result
        print(f"URL: {result.url}, Task ID: {dr.task_id}")
        print(f"Memory: {dr.memory_usage:.1f} MB (Peak: {dr.peak_memory:.1f} MB)")
        print(f"Duration: {dr.end_time - dr.start_time}")
Note: This field is typically populated when using arun_many(...) alongside a dispatcher (e.g., MemoryAdaptiveDispatcher or SemaphoreDispatcher). If no concurrency or dispatcher is used, dispatch_result may remain None.

7. Example: Accessing Everything
async def handle_result(result: CrawlResult):
    if not result.success:
        print("Crawl error:", result.error_message)
        return

    # Basic info
    print("Crawled URL:", result.url)
    print("Status code:", result.status_code)

    # HTML
    print("Original HTML size:", len(result.html))
    print("Cleaned HTML size:", len(result.cleaned_html or ""))

    # Markdown output
    if result.markdown:
        print("Raw Markdown:", result.markdown.raw_markdown[:300])
        print("Citations Markdown:", result.markdown.markdown_with_citations[:300])
        if result.markdown.fit_markdown:
            print("Fit Markdown:", result.markdown.fit_markdown[:200])

    # Media & Links
    if "images" in result.media:
        print("Image count:", len(result.media["images"]))
    if "internal" in result.links:
        print("Internal link count:", len(result.links["internal"]))

    # Extraction strategy result
    if result.extracted_content:
        print("Structured data:", result.extracted_content)

    # Screenshot/PDF
    if result.screenshot:
        print("Screenshot length:", len(result.screenshot))
    if result.pdf:
        print("PDF bytes length:", len(result.pdf))
8. Key Points & Future
1. Deprecated legacy properties of CrawlResult
- markdown_v2 - Deprecated in v0.5. Just use markdown. It holds the MarkdownGenerationResult now! - fit_markdown and fit_html - Deprecated in v0.5. They can now be accessed via MarkdownGenerationResult in result.markdown. eg: result.markdown.fit_markdown and result.markdown.fit_html

2. Fit Content
- fit_markdown and fit_html appear in MarkdownGenerationResult, only if you used a content filter (like PruningContentFilter or BM25ContentFilter) inside your MarkdownGenerationStrategy or set them directly.
- If no filter is used, they remain None.

3. References & Citations
- If you enable link citations in your DefaultMarkdownGenerator (options={"citations": True}), you’ll see markdown_with_citations plus a references_markdown block. This helps large language models or academic-like referencing.

4. Links & Media
- links["internal"] and links["external"] group discovered anchors by domain.
- media["images"] / ["videos"] / ["audios"] store extracted media elements with optional scoring or context.

5. Error Cases
- If success=False, check error_message (e.g., timeouts, invalid URLs).
- status_code might be None if we failed before an HTTP response.

Use CrawlResult to glean all final outputs and feed them into your data pipelines, AI models, or archives. With the synergy of a properly configured BrowserConfig and CrawlerRunConfig, the crawler can produce robust, structured results here in CrawlResult.
----------------------



Extraction & Chunking Strategies API
Extraction Strategies
Chunking Strategies
Usage Examples
Best Practices
Extraction & Chunking Strategies API
This documentation covers the API reference for extraction and chunking strategies in Crawl4AI.

Extraction Strategies
All extraction strategies inherit from the base ExtractionStrategy class and implement two key methods: - extract(url: str, html: str) -> List[Dict[str, Any]] - run(url: str, sections: List[str]) -> List[Dict[str, Any]]

LLMExtractionStrategy
Used for extracting structured data using Language Models.

LLMExtractionStrategy(
    # Required Parameters
    provider: str = DEFAULT_PROVIDER,     # LLM provider (e.g., "ollama/llama2")
    api_token: Optional[str] = None,      # API token

    # Extraction Configuration
    instruction: str = None,              # Custom extraction instruction
    schema: Dict = None,                  # Pydantic model schema for structured data
    extraction_type: str = "block",       # "block" or "schema"

    # Chunking Parameters
    chunk_token_threshold: int = 4000,    # Maximum tokens per chunk
    overlap_rate: float = 0.1,           # Overlap between chunks
    word_token_rate: float = 0.75,       # Word to token conversion rate
    apply_chunking: bool = True,         # Enable/disable chunking

    # API Configuration
    base_url: str = None,                # Base URL for API
    extra_args: Dict = {},               # Additional provider arguments
    verbose: bool = False                # Enable verbose logging
)
CosineStrategy
Used for content similarity-based extraction and clustering.

CosineStrategy(
    # Content Filtering
    semantic_filter: str = None,        # Topic/keyword filter
    word_count_threshold: int = 10,     # Minimum words per cluster
    sim_threshold: float = 0.3,         # Similarity threshold

    # Clustering Parameters
    max_dist: float = 0.2,             # Maximum cluster distance
    linkage_method: str = 'ward',       # Clustering method
    top_k: int = 3,                    # Top clusters to return

    # Model Configuration
    model_name: str = 'sentence-transformers/all-MiniLM-L6-v2',  # Embedding model

    verbose: bool = False              # Enable verbose logging
)
JsonCssExtractionStrategy
Used for CSS selector-based structured data extraction.

JsonCssExtractionStrategy(
    schema: Dict[str, Any],    # Extraction schema
    verbose: bool = False      # Enable verbose logging
)

# Schema Structure
schema = {
    "name": str,              # Schema name
    "baseSelector": str,      # Base CSS selector
    "fields": [               # List of fields to extract
        {
            "name": str,      # Field name
            "selector": str,  # CSS selector
            "type": str,     # Field type: "text", "attribute", "html", "regex"
            "attribute": str, # For type="attribute"
            "pattern": str,  # For type="regex"
            "transform": str, # Optional: "lowercase", "uppercase", "strip"
            "default": Any    # Default value if extraction fails
        }
    ]
}
Chunking Strategies
All chunking strategies inherit from ChunkingStrategy and implement the chunk(text: str) -> list method.

RegexChunking
Splits text based on regex patterns.

RegexChunking(
    patterns: List[str] = None  # Regex patterns for splitting
                               # Default: [r'\n\n']
)
SlidingWindowChunking
Creates overlapping chunks with a sliding window approach.

SlidingWindowChunking(
    window_size: int = 100,    # Window size in words
    step: int = 50             # Step size between windows
)
OverlappingWindowChunking
Creates chunks with specified overlap.

OverlappingWindowChunking(
    window_size: int = 1000,   # Chunk size in words
    overlap: int = 100         # Overlap size in words
)
Usage Examples
LLM Extraction
from pydantic import BaseModel
from crawl4ai.extraction_strategy import LLMExtractionStrategy
from crawl4ai import LLMConfig

# Define schema
class Article(BaseModel):
    title: str
    content: str
    author: str

# Create strategy
strategy = LLMExtractionStrategy(
    llm_config = LLMConfig(provider="ollama/llama2"),
    schema=Article.schema(),
    instruction="Extract article details"
)

# Use with crawler
result = await crawler.arun(
    url="https://example.com/article",
    extraction_strategy=strategy
)

# Access extracted data
data = json.loads(result.extracted_content)
CSS Extraction
from crawl4ai.extraction_strategy import JsonCssExtractionStrategy

# Define schema
schema = {
    "name": "Product List",
    "baseSelector": ".product-card",
    "fields": [
        {
            "name": "title",
            "selector": "h2.title",
            "type": "text"
        },
        {
            "name": "price",
            "selector": ".price",
            "type": "text",
            "transform": "strip"
        },
        {
            "name": "image",
            "selector": "img",
            "type": "attribute",
            "attribute": "src"
        }
    ]
}

# Create and use strategy
strategy = JsonCssExtractionStrategy(schema)
result = await crawler.arun(
    url="https://example.com/products",
    extraction_strategy=strategy
)
Content Chunking
from crawl4ai.chunking_strategy import OverlappingWindowChunking
from crawl4ai import LLMConfig

# Create chunking strategy
chunker = OverlappingWindowChunking(
    window_size=500,  # 500 words per chunk
    overlap=50        # 50 words overlap
)

# Use with extraction strategy
strategy = LLMExtractionStrategy(
    llm_config = LLMConfig(provider="ollama/llama2"),
    chunking_strategy=chunker
)

result = await crawler.arun(
    url="https://example.com/long-article",
    extraction_strategy=strategy
)
Best Practices
1. Choose the Right Strategy - Use LLMExtractionStrategy for complex, unstructured content - Use JsonCssExtractionStrategy for well-structured HTML - Use CosineStrategy for content similarity and clustering

2. Optimize Chunking

# For long documents
strategy = LLMExtractionStrategy(
    chunk_token_threshold=2000,  # Smaller chunks
    overlap_rate=0.1           # 10% overlap
)
3. Handle Errors

try:
    result = await crawler.arun(
        url="https://example.com",
        extraction_strategy=strategy
    )
    if result.success:
        content = json.loads(result.extracted_content)
except Exception as e:
    print(f"Extraction failed: {e}")
4. Monitor Performance

strategy = CosineStrategy(
    verbose=True,  # Enable logging
    word_count_threshold=20,  # Filter short content
    top_k=5  # Limit results
)

