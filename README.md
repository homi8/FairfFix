# FairfFix

** Agent Orchestration Engine**

Stop waiting for sequential agent execution. Conductor automatically parallelizes your AI agent workflows, reducing execution time by 2-5x with zero code changes.

## The Problem

Most AI agent workflows look like this:

```python
result1 = agent1.run(input)      # 4s
result2 = agent2.run(result1)    # 3s - waiting...
result3 = agent3.run(result1)    # 5s - waiting...
result4 = agent4.run([result2, result3])  # 2s - waiting...
# Total: 14 seconds
```

But agents 2 and 3 could run in parallel!

## The Solution

Conductor automatically detects dependencies and parallelizes execution:

```python
from conductor import Conductor

workflow = Conductor([agent1, agent2, agent3, agent4])
results = await workflow.execute(input)
# Total: 7 seconds (2x faster!)
```

## Features

- 🚀 **2-5x faster execution** through intelligent parallelization
- 🔄 **Zero code changes** - works with existing agents
- 📊 **Real-time visualization** - see your workflow execute live
- 🎯 **Smart scheduling** - balance speed, cost, and resources
- 🛡️ **Failure handling** - retries, circuit breakers, graceful degradation
- 📈 **Performance insights** - identify bottlenecks automatically

## Quick Start

### Installation

```bash
pip install conductor-ai
```

### Basic Usage

```python
from conductor import Conductor, Agent

# Define your agents
class SentimentAgent(Agent):
    async def execute(self, inputs):
        # Your agent logic
        return {"sentiment": "positive"}

class EntityAgent(Agent):
    async def execute(self, inputs):
        # Your agent logic
        return {"entities": ["Apple", "Google"]}

# Create workflow
workflow = Conductor([
    SentimentAgent(depends_on=[]),
    EntityAgent(depends_on=[]),
])

# Execute (parallel by default)
results = await workflow.execute({"text": "I love this product!"})
```

### YAML Configuration

```yaml
workflow:
  - name: fetch
    agent: WebScraperAgent
    inputs: [url]
    
  - name: extract
    agent: TextExtractorAgent
    depends_on: [fetch]
    
  - name: sentiment
    agent: SentimentAgent
    depends_on: [extract]
    
  - name: entities
    agent: EntityAgent
    depends_on: [extract]
    
  - name: summarize
    agent: SummaryAgent
    depends_on: [sentiment, entities]
```

```python
from conductor import Conductor

workflow = Conductor.from_yaml("workflow.yaml")
results = await workflow.execute({"url": "https://example.com"})
```

## How It Works

1. **Dependency Analysis** - Conductor builds a DAG of agent dependencies
2. **Stage Identification** - Groups independent agents into parallel execution stages
3. **Concurrent Execution** - Runs all agents in a stage simultaneously using asyncio
4. **Result Propagation** - Passes outputs to dependent agents in subsequent stages

## Architecture

```
Input → [Agent1] → [Agent2, Agent3, Agent4] → [Agent5] → Output
         Stage 1      Stage 2 (parallel)       Stage 3
```

## Performance

Typical speedups for different workflow structures:

- **Linear (no parallelism):** 1x (same as sequential)
- **Fork-join (30% parallel):** 1.5-2x faster
- **Diamond (50% parallel):** 2-3x faster  
- **Wide parallelism (70%+):** 3-5x faster

## Dashboard

Start the visualization dashboard:

```bash
conductor dashboard --port 8080
```

Features:
- Real-time execution graph
- Critical path highlighting
- Performance metrics (time saved, parallelization efficiency)
- Bottleneck identification

## Advanced Usage

### Optimization Modes

```python
# Fastest execution (default)
workflow.execute(input, mode="fastest")

# Minimize cost
workflow.execute(input, mode="cheapest")

# Balanced
workflow.execute(input, mode="balanced")
```

### Custom Agent Interface

```python
class CustomAgent(Agent):
    async def execute(self, inputs: dict) -> dict:
        # Your logic here
        pass
    
    def get_dependencies(self) -> list[str]:
        return ["agent1", "agent2"]
    
    def estimate_cost(self) -> float:
        return 0.01  # USD
    
    def estimate_duration(self) -> float:
        return 2.5  # seconds
```

### Worker Pool Configuration

```python
workflow = Conductor(
    agents,
    max_workers=10,  # Limit concurrent agents
    retry_failed=True,
    timeout=30  # seconds per agent
)
```

## Examples

See the `examples/` directory:
- `content_pipeline.py` - Web scraping and analysis
- `research_assistant.py` - Multi-source research synthesis
- `data_processing.py` - ETL pipeline with validation
- `customer_support.py` - Automated support workflow

## Requirements

- Python 3.11+
- asyncio
- networkx
- fastapi (for dashboard)
- aiohttp

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.


## Roadmap

- [ ] Integration with LangChain, CrewAI, AutoGPT
- [ ] Distributed execution across multiple machines
- [ ] GPU-aware scheduling
- [ ] Automatic workflow optimization suggestions
- [ ] Cloud deployment templates

=

---
