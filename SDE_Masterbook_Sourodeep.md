# 🧠 SDE ONBOARDING MASTERBOOK
## Sourodeep Kundu — Day-One Manager Briefing & Deep Technical Reference
### QA Automation | Software Dev from Scratch | Agentic AI | Open-Source & Proprietary LLM Agents

> **Author note:** This document is tailored to your background — M.Tech CSE VIT (9.2 CGPA), published IEEE researcher in QML/AI, CrewAI practitioner (QML-AURA), Deloitte D&R analyst, IIT Bombay / I4C intern. It bridges your research-grade AI expertise into production SDE practice.

---

# ═══════════════════════════════════════════════
# PART 1: QA AUTOMATION — BASICS TO ADVANCED
# ═══════════════════════════════════════════════

## 1.1 What is QA Automation and Why It Matters

Quality Assurance (QA) Automation is the practice of using software tools and scripts to **automatically execute tests** against a software system, compare actual outcomes with expected results, and report findings — replacing or augmenting manual human testing.

### The Testing Pyramid (Foundational Mental Model)

```
          /\
         /E2E\          <- Fewest tests, highest cost, slowest
        /------\
       /  API   \       <- Middle layer, fast and reliable
      /----------\
     /  Unit Tests \    <- Most tests, fastest, cheapest
    /--------------\
```

| Layer | What it tests | Tools | Speed | Cost |
|-------|--------------|-------|-------|------|
| Unit  | Individual functions/classes | pytest, JUnit, NUnit | Milliseconds | Lowest |
| Integration | Modules working together | pytest, Postman, REST Assured | Seconds | Medium |
| API/Contract | Service interfaces | Postman, Pact, RestAssured | Seconds | Medium |
| E2E/UI | Full user journeys | Playwright, Selenium, Cypress | Minutes | Highest |
| Performance | Load & stress behavior | JMeter, k6, Locust | Minutes-Hours | High |
| Security | Vulnerabilities | OWASP ZAP, Burp Suite | Hours | High |

---

## 1.2 Core Testing Types (Every SDE Must Know)

### 1.2.1 Functional Testing Types

**Unit Testing** — Testing smallest code unit in isolation
```python
# Example: pytest unit test
import pytest

def add(a, b):
    return a + b

def test_add_positive():
    assert add(2, 3) == 5

def test_add_negative():
    assert add(-1, 1) == 0

def test_add_zero():
    assert add(0, 0) == 0
```

**Integration Testing** — Testing how modules interact
```python
# Testing that DB + service layer work together
def test_user_creation_and_retrieval(db_session):
    user = UserService.create(db_session, name="Sourodeep", email="test@test.com")
    fetched = UserService.get_by_id(db_session, user.id)
    assert fetched.name == "Sourodeep"
```

**Regression Testing** — Re-running previous tests to ensure new changes didn't break existing functionality. This is the PRIMARY use case for automation.

**Smoke Testing** — Quick sanity check: "does the app start and do the most basic things?" Run after every deployment.

**Sanity Testing** — Narrow regression focused on a specific bug fix.

**Acceptance Testing (UAT)** — Does the system meet business requirements?

**Exploratory Testing** — Unscripted, human-driven testing to find unexpected bugs.

### 1.2.2 Non-Functional Testing

**Performance Testing** — Measures responsiveness, throughput, stability
- Load Test: Normal expected load
- Stress Test: Beyond normal capacity  
- Spike Test: Sudden traffic surges
- Endurance/Soak Test: Extended period at normal load

**Security Testing** — OWASP Top 10: injection, broken auth, XSS, CSRF, etc.

**Accessibility Testing** — WCAG compliance (axe, Lighthouse)

**Compatibility Testing** — Cross-browser, cross-device, cross-OS

---

## 1.3 The Testing Process (SDLC Integration)

```
Requirements → Test Planning → Test Design → Test Execution → Defect Reporting → Closure

↕ Shift-Left                                                    Shift-Right ↕
(Test early,                                              (Test in production,
 in design phase)                                          monitor real users)
```

### Test Plan Components
1. **Scope** — What will/won't be tested
2. **Test Strategy** — Approach (manual + automated mix)
3. **Test Environment** — Dev, staging, prod
4. **Entry/Exit Criteria** — When to start/stop testing
5. **Risk Assessment** — What could go wrong
6. **Test Schedule** — Timeline and milestones
7. **Deliverables** — Test cases, reports, metrics

### Key Metrics You Must Know
| Metric | Formula | Good Range |
|--------|---------|-----------|
| Test Coverage | (Tested lines / Total lines) × 100 | >80% |
| Defect Density | Defects / KLOC (thousand lines of code) | <1 |
| Test Pass Rate | (Passed / Total) × 100 | >95% |
| Defect Escape Rate | Defects found in prod / Total defects | <10% |
| Flakiness Rate | Flaky tests / Total tests | <5% |
| MTTR (Mean Time To Repair) | Time to fix a failing test | <1 day |

---

## 1.4 Top QA Automation Tools (2025-2026 Landscape)

### Web UI Automation

#### 🥇 Playwright (Current Industry Default 2026)
The modern standard for web automation. Native cross-browser support, built-in tracing, fast parallel execution.

```python
# playwright_test.py
from playwright.sync_api import sync_playwright

def test_google_search():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        page.goto("https://www.google.com")
        page.fill("textarea[name='q']", "CrewAI agents")
        page.keyboard.press("Enter")
        page.wait_for_load_state("networkidle")
        assert "CrewAI" in page.title() or page.locator("h3").count() > 0
        browser.close()
```

**Installation:**
```bash
pip install playwright
playwright install chromium
```

**Running:**
```bash
pytest tests/ --headed  # with browser visible
pytest tests/ --workers 4  # parallel execution
```

#### Selenium (Battle-tested, Flexible)
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("https://example.com")
wait = WebDriverWait(driver, 10)
element = wait.until(EC.presence_of_element_located((By.ID, "login")))
element.click()
driver.quit()
```

#### Cypress (JavaScript/TypeScript, Developer-Friendly)
```javascript
// cypress/e2e/login.cy.js
describe('Login Flow', () => {
  it('should login successfully', () => {
    cy.visit('/login')
    cy.get('[data-testid=email]').type('user@test.com')
    cy.get('[data-testid=password]').type('password123')
    cy.get('[data-testid=submit]').click()
    cy.url().should('include', '/dashboard')
  })
})
```

### API Testing

#### Postman / Newman (CLI)
```bash
# Run Postman collection via CLI
newman run collection.json \
  --environment staging.json \
  --reporters cli,html \
  --reporter-html-export results.html
```

#### Python requests + pytest (Programmatic API Testing)
```python
import requests
import pytest

BASE_URL = "https://api.example.com/v1"
HEADERS = {"Authorization": "Bearer TOKEN", "Content-Type": "application/json"}

class TestUserAPI:
    def test_create_user(self):
        payload = {"name": "Sourodeep", "email": "s@test.com", "role": "admin"}
        response = requests.post(f"{BASE_URL}/users", json=payload, headers=HEADERS)
        assert response.status_code == 201
        data = response.json()
        assert data["name"] == "Sourodeep"
        assert "id" in data

    def test_get_user(self):
        response = requests.get(f"{BASE_URL}/users/1", headers=HEADERS)
        assert response.status_code == 200
        assert response.json()["id"] == 1

    def test_unauthorized_access(self):
        response = requests.get(f"{BASE_URL}/users/1")  # No auth header
        assert response.status_code == 401
```

### Performance Testing

#### Locust (Python-native, Excellent for Agents)
```python
from locust import HttpUser, task, between

class AgentAPIUser(HttpUser):
    wait_time = between(1, 3)

    @task(3)   # weight: called 3x more than others
    def query_agent(self):
        self.client.post("/api/agent/query", 
                         json={"question": "What is quantum computing?"},
                         headers={"Authorization": "Bearer TOKEN"})

    @task(1)
    def health_check(self):
        self.client.get("/health")
```

```bash
locust -f locustfile.py --host=https://api.example.com
# Opens web UI at localhost:8089
```

### Mobile Testing
- **Appium** — Cross-platform mobile automation (iOS + Android)
- **Detox** — React Native specific, very fast
- **XCUITest / Espresso** — Native iOS/Android

---

## 1.5 Test Frameworks & Runners

### pytest (Python — Most Important for You)

```python
# conftest.py - shared fixtures
import pytest
import requests

@pytest.fixture(scope="session")
def api_client():
    """Reusable authenticated API client."""
    session = requests.Session()
    session.headers.update({"Authorization": "Bearer test_token"})
    return session

@pytest.fixture(scope="function")
def fresh_user(api_client):
    """Create a user, yield for test, then delete."""
    user = api_client.post("/users", json={"name": "test_user"}).json()
    yield user
    api_client.delete(f"/users/{user['id']}")  # Cleanup

# test_users.py
def test_user_profile(api_client, fresh_user):
    response = api_client.get(f"/users/{fresh_user['id']}")
    assert response.status_code == 200

# Markers
@pytest.mark.slow
def test_heavy_computation():
    ...

@pytest.mark.parametrize("email,valid", [
    ("user@test.com", True),
    ("invalid-email", False),
    ("@no-local.com", False),
])
def test_email_validation(email, valid):
    result = validate_email(email)
    assert result == valid
```

**pytest.ini configuration:**
```ini
[pytest]
markers =
    slow: marks tests as slow
    api: marks tests as api tests
    smoke: marks tests as smoke tests
addopts = -v --tb=short --strict-markers
testpaths = tests
```

**Running subsets:**
```bash
pytest -m smoke           # Only smoke tests
pytest -m "api and not slow"  # API tests excluding slow ones
pytest -k "user"          # All tests with "user" in name
pytest --co               # Collect only (show what would run)
pytest -x                 # Stop at first failure
pytest --lf               # Re-run only last-failed tests
```

---

## 1.6 CI/CD Integration (Critical for SDE)

### CI/CD Pipeline Concept
```
Developer Pushes Code
       ↓
CI System Triggers (GitHub Actions / Jenkins / GitLab CI)
       ↓
Build → Unit Tests → Integration Tests → Code Quality → Deploy to Staging
       ↓
E2E Tests on Staging
       ↓
Performance + Security Tests
       ↓
Deploy to Production (if all pass)
       ↓
Shift-Right Monitoring (production telemetry)
```

### GitHub Actions Example
```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      - name: Run unit tests
        run: pytest tests/unit/ --cov=src --cov-report=xml
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  e2e-tests:
    runs-on: ubuntu-latest
    needs: unit-tests   # Only run if unit tests pass
    steps:
      - uses: actions/checkout@v4
      - name: Install Playwright
        run: |
          pip install playwright pytest-playwright
          playwright install --with-deps chromium
      - name: Run E2E tests
        run: pytest tests/e2e/
```

---

## 1.7 Advanced QA Concepts (2025-2026)

### Self-Healing Tests (AI-powered)
Traditional tests break when UI changes (e.g., button ID renamed). Self-healing tests use AI to:
- Detect that a locator has changed
- Automatically find the new locator using visual context
- Update the test script

Platforms: Katalon TrueTest, Testim, Healenium (open-source)

### Test Data Management
```python
# factory_boy + faker for realistic test data
import factory
from faker import Faker

fake = Faker()

class UserFactory(factory.Factory):
    class Meta:
        model = User
    
    name = factory.LazyFunction(fake.name)
    email = factory.LazyFunction(fake.email)
    age = factory.LazyFunction(lambda: fake.random_int(18, 80))
    created_at = factory.LazyFunction(fake.date_time_this_year)

# Usage
user = UserFactory()
users = UserFactory.create_batch(50)
```

### Contract Testing (Microservices)
Using **Pact**: ensures that a consumer and provider agree on the API contract, preventing integration bugs.

```python
# Consumer side defines expected contract
from pact import Consumer, Provider

pact = Consumer("UserService").has_pact_with(Provider("AuthService"))

pact.given("user exists").upon_receiving("a request for user 1").with_request(
    method="GET", path="/users/1"
).will_respond_with(
    status=200,
    body={"id": 1, "name": "Sourodeep"}
)
```

### Page Object Model (POM) Pattern — Industry Standard
```python
# pages/login_page.py
from playwright.sync_api import Page

class LoginPage:
    def __init__(self, page: Page):
        self.page = page
        self.email_input = page.locator("[data-testid='email']")
        self.password_input = page.locator("[data-testid='password']")
        self.submit_button = page.locator("[data-testid='submit']")
        self.error_message = page.locator(".error-message")

    def navigate(self):
        self.page.goto("/login")
        return self

    def login(self, email: str, password: str):
        self.email_input.fill(email)
        self.password_input.fill(password)
        self.submit_button.click()
        return self  # for chaining

    def get_error(self) -> str:
        return self.error_message.text_content()

# test_login.py
def test_invalid_login(page):
    login_page = LoginPage(page)
    login_page.navigate().login("wrong@email.com", "badpassword")
    assert "Invalid credentials" in login_page.get_error()
```

### BDD (Behavior-Driven Development) with pytest-bdd
```gherkin
# features/login.feature
Feature: User Login
  Scenario: Successful login
    Given I am on the login page
    When I enter valid credentials
    Then I should be redirected to the dashboard
    And I should see a welcome message
```

```python
# step_defs/login_steps.py
from pytest_bdd import given, when, then, scenario

@scenario("features/login.feature", "Successful login")
def test_successful_login():
    pass

@given("I am on the login page")
def navigate_to_login(page):
    page.goto("/login")

@when("I enter valid credentials")
def enter_credentials(page):
    page.fill("[name='email']", "valid@user.com")
    page.fill("[name='password']", "correct_password")
    page.click("[type='submit']")
```

---

## 1.8 AI-Powered QA (The 2026 Frontier)

**AI in QA is shifting from "assistant" to "agent":**

1. **AI Test Generation** — Feed requirements/user stories, get test cases
   - Tool: TestRail AI, Baserock.ai, GitHub Copilot
   
2. **Autonomous Test Execution Agents** — Agents that read the screen, decide what to test, execute, report
   - World Quality Report 2025 calls this the "Agentic Shift"
   
3. **Intelligent Test Prioritization** — ML models predict which tests are most likely to fail based on code changes
   
4. **Visual Regression Testing** — AI compares screenshots pixel-by-pixel with semantic understanding
   - Tool: Applitools Eyes

5. **Fuzz Testing with AI** — Automatically generate unexpected inputs to find edge cases

**Your QML-AURA Connection:** The multi-agent orchestration principles in QML-AURA apply directly to agentic QA systems — you can conceptualize a QA Crew with: TestGeneratorAgent → ExecutorAgent → EvaluatorAgent → ReportAgent.

---

# ═══════════════════════════════════════════════
# PART 2: SOFTWARE DEVELOPMENT FROM SCRATCH
# ═══════════════════════════════════════════════

## 2.1 The Software Development Lifecycle (SDLC)

```
Planning → Requirements → System Design → Implementation → Testing → Deployment → Maintenance
    ↑_______________________________________________________________|
                         (Agile: short iterations/sprints)
```

### Agile/Scrum (What Most Orgs Use)
- **Sprint**: 2-week iteration cycle
- **Backlog**: Prioritized list of features (user stories)
- **Daily Standup**: 15-min sync (what I did, what I'll do, any blockers)
- **Sprint Review**: Demo completed work to stakeholders
- **Retrospective**: What went well, what to improve

### Git Workflow (Daily Tool)
```bash
# Basic daily workflow
git checkout -b feature/add-user-authentication   # Create feature branch
git add .                                          # Stage changes
git commit -m "feat: add JWT authentication"      # Conventional commit
git push origin feature/add-user-authentication   # Push to remote
# Create Pull Request (PR) → Code Review → Merge

# Conventional Commits (industry standard)
# feat: new feature
# fix: bug fix
# docs: documentation only
# test: adding tests
# refactor: code change that neither fixes bug nor adds feature
# chore: build process or tooling changes
```

---

## 2.2 Core Programming: Python (Production Grade)

### Data Types and Structures
```python
# Immutable types
integer = 42
float_num = 3.14
string = "Sourodeep"
tuple_data = (1, 2, 3)
frozenset_data = frozenset({1, 2, 3})

# Mutable types
list_data = [1, 2, 3]       # Ordered, allows duplicates
dict_data = {"key": "val"}  # Key-value, O(1) lookup
set_data = {1, 2, 3}        # Unordered, no duplicates

# Python 3.10+ pattern matching (switch-case equivalent)
command = "start"
match command:
    case "start":
        print("Starting...")
    case "stop":
        print("Stopping...")
    case _:
        print("Unknown command")
```

### Object-Oriented Programming (OOP)
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import Optional, List

# Abstract Base Class
class AIAgent(ABC):
    def __init__(self, name: str, model: str):
        self.name = name
        self.model = model
        self._memory: List[dict] = []  # Protected attribute
        self.__api_key: str = ""       # Private attribute
    
    @abstractmethod
    def execute(self, task: str) -> str:
        """Every agent must implement this."""
        pass
    
    @property
    def memory_size(self) -> int:
        return len(self._memory)
    
    def add_to_memory(self, entry: dict) -> None:
        self._memory.append(entry)
    
    def __repr__(self) -> str:
        return f"Agent(name={self.name}, model={self.model})"

# Concrete implementation
class ResearchAgent(AIAgent):
    def __init__(self, name: str, model: str, tools: List[str]):
        super().__init__(name, model)
        self.tools = tools
    
    def execute(self, task: str) -> str:
        result = f"[{self.name}] Researching: {task}"
        self.add_to_memory({"task": task, "result": result})
        return result

# Dataclass (clean way to make data-holding classes)
@dataclass
class TaskResult:
    task_id: str
    agent_name: str
    output: str
    success: bool = True
    metadata: dict = field(default_factory=dict)

# Usage
agent = ResearchAgent("ResearchBot", "claude-sonnet-4-6", ["web_search", "arxiv"])
result = agent.execute("quantum machine learning 2026")
task_result = TaskResult("t001", agent.name, result)
```

### Design Patterns (Must Know for SDE)

#### Factory Pattern
```python
class AgentFactory:
    @staticmethod
    def create(agent_type: str, **kwargs) -> AIAgent:
        registry = {
            "research": ResearchAgent,
            "coding": CodingAgent,
            "analysis": AnalysisAgent,
        }
        if agent_type not in registry:
            raise ValueError(f"Unknown agent type: {agent_type}")
        return registry[agent_type](**kwargs)

agent = AgentFactory.create("research", name="R1", model="gpt-4o", tools=[])
```

#### Singleton Pattern
```python
class ConfigManager:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.config = {}
        return cls._instance
    
    def set(self, key, value):
        self.config[key] = value
    
    def get(self, key, default=None):
        return self.config.get(key, default)
```

#### Observer Pattern (Used in Event-Driven Systems)
```python
from typing import Callable

class EventBus:
    def __init__(self):
        self._subscribers: dict[str, List[Callable]] = {}
    
    def subscribe(self, event: str, callback: Callable):
        self._subscribers.setdefault(event, []).append(callback)
    
    def publish(self, event: str, data: dict):
        for cb in self._subscribers.get(event, []):
            cb(data)

# Usage
bus = EventBus()
bus.subscribe("task_complete", lambda d: print(f"Task done: {d}"))
bus.publish("task_complete", {"task": "research", "status": "success"})
```

### Async Python (Critical for Agent Systems)
```python
import asyncio
import aiohttp
from typing import List

async def fetch_data(session: aiohttp.ClientSession, url: str) -> dict:
    async with session.get(url) as response:
        return await response.json()

async def run_agents_parallel(tasks: List[str]) -> List[str]:
    """Run multiple agent tasks concurrently."""
    async def process_task(task: str) -> str:
        await asyncio.sleep(0.1)  # Simulate API call
        return f"Result for: {task}"
    
    results = await asyncio.gather(*[process_task(t) for t in tasks])
    return list(results)

# Running async code
if __name__ == "__main__":
    tasks = ["research QML", "analyze data", "write report"]
    results = asyncio.run(run_agents_parallel(tasks))
    print(results)
```

### Error Handling & Logging
```python
import logging
from functools import wraps
from typing import TypeVar, Callable

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)

# Custom exceptions
class AgentError(Exception):
    """Base exception for agent errors."""
    pass

class ToolExecutionError(AgentError):
    def __init__(self, tool_name: str, message: str):
        self.tool_name = tool_name
        super().__init__(f"Tool '{tool_name}' failed: {message}")

# Retry decorator
F = TypeVar('F', bound=Callable)

def retry(max_attempts: int = 3, delay: float = 1.0):
    def decorator(func: F) -> F:
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    logger.warning(f"Attempt {attempt+1} failed: {e}")
                    if attempt == max_attempts - 1:
                        raise
                    import time; time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2.0)
def call_llm_api(prompt: str) -> str:
    # API call that might fail
    ...
```

---

## 2.3 Core Programming: C++ (Production Fundamentals)

```cpp
// Modern C++17/20 features you must know
#include <iostream>
#include <vector>
#include <memory>
#include <string>
#include <algorithm>
#include <ranges>     // C++20
#include <concepts>   // C++20
#include <span>       // C++20

// Smart pointers (never use raw new/delete in modern C++)
void smart_pointer_demo() {
    // unique_ptr: single ownership
    auto agent = std::make_unique<AIAgent>("Bot1", "gpt-4");
    
    // shared_ptr: shared ownership with ref counting
    auto shared_agent = std::make_shared<AIAgent>("Bot2", "claude");
    auto another_ref = shared_agent;  // ref count = 2
    
    // weak_ptr: non-owning reference (avoids circular refs)
    std::weak_ptr<AIAgent> weak_ref = shared_agent;
}

// Templates (Generics)
template<typename T>
class AgentMemory {
public:
    void add(const T& item) { items_.push_back(item); }
    T& get(size_t index) { return items_.at(index); }
    size_t size() const { return items_.size(); }
private:
    std::vector<T> items_;
};

// C++20 Concepts
template<typename T>
concept Stringable = requires(T t) {
    { t.to_string() } -> std::convertible_to<std::string>;
};

// Range-based algorithms (C++20)
void modern_cpp_demo() {
    std::vector<int> nums = {5, 3, 8, 1, 9, 2};
    
    // Sort
    std::ranges::sort(nums);
    
    // Filter + transform pipeline
    auto result = nums
        | std::views::filter([](int n) { return n > 3; })
        | std::views::transform([](int n) { return n * 2; });
    
    for (auto n : result) std::cout << n << " ";
}

// Move semantics (performance critical)
class TaskQueue {
public:
    void enqueue(std::string task) {       // Copies string
        queue_.push_back(task);
    }
    
    void enqueue_fast(std::string&& task) { // Moves string (no copy)
        queue_.push_back(std::move(task));
    }
    
private:
    std::vector<std::string> queue_;
};
```

### C++ Memory Management & Multithreading
```cpp
#include <thread>
#include <mutex>
#include <atomic>
#include <future>

// Thread-safe agent executor
class ThreadSafeExecutor {
public:
    void submit(const std::string& task) {
        std::lock_guard<std::mutex> lock(mutex_);
        tasks_.push_back(task);
    }
    
    std::string get_result(const std::string& task_id) {
        std::shared_lock<std::shared_mutex> lock(rw_mutex_);
        return results_.count(task_id) ? results_.at(task_id) : "";
    }
    
private:
    std::vector<std::string> tasks_;
    std::unordered_map<std::string, std::string> results_;
    std::mutex mutex_;
    std::shared_mutex rw_mutex_;
};

// Async with futures
std::future<std::string> run_agent_async(const std::string& prompt) {
    return std::async(std::launch::async, [prompt]() {
        // Simulate LLM call
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        return "Result for: " + prompt;
    });
}

// Usage
auto future = run_agent_async("analyze quantum circuit");
// ... do other work ...
std::string result = future.get();  // Blocks until ready
```

---

## 2.4 Software Architecture (System Design)

### Architecture Patterns You Must Know

#### 1. Layered (N-Tier) Architecture
```
Presentation Layer  (API/UI)
       ↓
Business Logic Layer (Services)
       ↓
Data Access Layer   (Repositories)
       ↓
Database Layer      (SQL/NoSQL)
```

#### 2. Microservices Architecture
```
         [API Gateway]
        /      |       \
   [User   [Agent   [Report
  Service] Service] Service]
     |         |        |
  [User DB] [Vector  [Analytics
            Store]    DB]
```

#### 3. Event-Driven Architecture (Critical for Agent Systems)
```python
# Using Apache Kafka concepts
# Producer (Agent publishes task completion)
producer.send("task-complete", {
    "agent_id": "encoder_agent",
    "task_id": "t001",
    "result": {...},
    "timestamp": "2026-05-20T10:00:00Z"
})

# Consumer (Manager agent listens and reacts)
@kafka_consumer("task-complete")
def handle_task_complete(event: dict):
    if event["result"]["quality"] < 0.8:
        trigger_reoptimization(event["task_id"])
```

#### 4. Hexagonal Architecture (Ports & Adapters)
```
     [External HTTP API]
            ↓
     [Input Adapter]
            ↓
     [Domain Core / Business Logic]
            ↑
     [Output Adapter]
            ↑
     [Database / LLM / External Service]
```

#### 5. Clean Architecture (Most Testable)
```
Entities (innermost, no dependencies)
  → Use Cases
    → Interface Adapters
      → Frameworks & Drivers (outermost)

Rule: Dependencies only point INWARD
```

### REST API Design (Production Standards)
```python
# FastAPI example (modern, async, auto-docs)
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI(title="Agent API", version="1.0.0")

class AgentRequest(BaseModel):
    task: str
    agent_type: str = "research"
    max_iterations: int = 5

class AgentResponse(BaseModel):
    task_id: str
    status: str
    result: Optional[str] = None
    agent_name: str

# RESTful endpoints
# GET    /agents         - List all agents
# POST   /agents         - Create new agent
# GET    /agents/{id}    - Get specific agent
# PUT    /agents/{id}    - Update agent
# DELETE /agents/{id}    - Delete agent
# POST   /agents/{id}/run - Execute agent task

@app.post("/agents/{agent_id}/run", response_model=AgentResponse)
async def run_agent(
    agent_id: str,
    request: AgentRequest,
    api_key: str = Depends(verify_api_key)  # Dependency injection
):
    agent = AgentRegistry.get(agent_id)
    if not agent:
        raise HTTPException(status_code=404, detail=f"Agent {agent_id} not found")
    
    result = await agent.execute_async(request.task)
    return AgentResponse(
        task_id=generate_id(),
        status="completed",
        result=result,
        agent_name=agent.name
    )
```

### Database Fundamentals
```python
# SQLAlchemy ORM (Python standard)
from sqlalchemy import Column, Integer, String, DateTime, JSON
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class AgentTask(Base):
    __tablename__ = "agent_tasks"
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    task_id = Column(String(36), unique=True, nullable=False)
    agent_name = Column(String(100), nullable=False)
    prompt = Column(String(5000), nullable=False)
    result = Column(String(50000))
    metadata = Column(JSON)
    created_at = Column(DateTime, default=datetime.utcnow)
    status = Column(String(20), default="pending")

# Queries
from sqlalchemy.orm import Session

def get_recent_tasks(db: Session, agent_name: str, limit: int = 10):
    return (db.query(AgentTask)
              .filter(AgentTask.agent_name == agent_name)
              .filter(AgentTask.status == "completed")
              .order_by(AgentTask.created_at.desc())
              .limit(limit)
              .all())
```

---

## 2.5 Basic AI/ML for SDE (Applied, Not Research)

Given your quantum ML background, this section focuses on **production ML patterns** that differ from research.

### The ML Pipeline
```
Data Collection → Data Cleaning → Feature Engineering → 
Model Selection → Training → Evaluation → Deployment → Monitoring
```

### Scikit-learn (Standard ML)
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report
import pandas as pd
import numpy as np

# Load and split data
df = pd.read_csv("data.csv")
X = df.drop("label", axis=1)
y = df["label"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Pipeline: preprocessing + model
pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("classifier", RandomForestClassifier(n_estimators=100, random_state=42))
])

pipeline.fit(X_train, y_train)
y_pred = pipeline.predict(X_test)
print(classification_report(y_test, y_pred))

# Cross-validation (more reliable than single split)
cv_scores = cross_val_score(pipeline, X, y, cv=5, scoring="accuracy")
print(f"CV Score: {cv_scores.mean():.3f} ± {cv_scores.std():.3f}")
```

### LLM Integration (The 2026 SDE Standard)
```python
from anthropic import Anthropic
import openai

# Anthropic Claude
client = Anthropic()

def call_claude(system_prompt: str, user_message: str) -> str:
    message = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=system_prompt,
        messages=[{"role": "user", "content": user_message}]
    )
    return message.content[0].text

# With conversation history
class ConversationalAgent:
    def __init__(self, system_prompt: str):
        self.client = Anthropic()
        self.system = system_prompt
        self.history = []
    
    def chat(self, message: str) -> str:
        self.history.append({"role": "user", "content": message})
        
        response = self.client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=2048,
            system=self.system,
            messages=self.history
        )
        
        reply = response.content[0].text
        self.history.append({"role": "assistant", "content": reply})
        return reply
```

### RAG System (Retrieval-Augmented Generation)
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain.chains import RetrievalQA

# 1. Load and chunk documents
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(documents)

# 2. Embed and store in vector DB
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings, persist_directory="./db")

# 3. Retrieve and answer
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})
qa_chain = RetrievalQA.from_chain_type(llm=llm, retriever=retriever)
answer = qa_chain.invoke("What is QAOA?")
```

---

# ═══════════════════════════════════════════════
# PART 3: AGENTIC AI — FROM SCRATCH TO PRODUCTION
# ═══════════════════════════════════════════════

## 3.1 What is an AI Agent? (Foundational)

An AI agent is a system that:
1. **Perceives** its environment (text, images, tool outputs, APIs)
2. **Reasons** about what to do next (via LLM)
3. **Acts** using tools (web search, code execution, APIs)
4. **Remembers** past interactions (memory systems)
5. **Learns** from feedback (RLHF, fine-tuning, or prompt updates)

```
Agent = LLM (Brain) + Tools (Hands) + Memory (Mind) + Planner (Strategy)
```

### The ReAct Pattern (Reason + Act) — Core Agent Loop
```
Observation → Thought → Action → Observation → Thought → Action → ... → Final Answer

Example:
Observation: "User asks: What is the latest paper on QML agents?"
Thought: "I need to search for recent QML papers."
Action: web_search("QML agents 2026 arxiv")
Observation: "Found: 'QML-AURA: Quantum Multi-Agent...' by Sourodeep Kundu"
Thought: "Found a relevant paper. I should summarize it."
Final Answer: "The latest paper is QML-AURA by Kundu et al., which proposes..."
```

---

## 3.2 Building an Agent from Scratch (Raw Python)

### Minimal Agent Implementation
```python
import json
from anthropic import Anthropic

client = Anthropic()

# Define tools the agent can use
tools = [
    {
        "name": "web_search",
        "description": "Search the web for current information",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"}
            },
            "required": ["query"]
        }
    },
    {
        "name": "python_executor",
        "description": "Execute Python code and return output",
        "input_schema": {
            "type": "object",
            "properties": {
                "code": {"type": "string", "description": "Python code to execute"}
            },
            "required": ["code"]
        }
    }
]

def execute_tool(tool_name: str, tool_input: dict) -> str:
    """Execute the requested tool and return result."""
    if tool_name == "web_search":
        # In production, connect to real search API
        return f"[Mock Search Result for: {tool_input['query']}]"
    elif tool_name == "python_executor":
        import io, sys
        output = io.StringIO()
        sys.stdout = output
        try:
            exec(tool_input['code'])
        except Exception as e:
            return f"Error: {str(e)}"
        finally:
            sys.stdout = sys.__stdout__
        return output.getvalue()
    return "Unknown tool"

def run_agent(user_message: str, max_iterations: int = 10) -> str:
    """Main agent loop using Claude's tool_use feature."""
    messages = [{"role": "user", "content": user_message}]
    
    for iteration in range(max_iterations):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )
        
        # Check if agent is done
        if response.stop_reason == "end_turn":
            # Extract final text response
            return next(
                (block.text for block in response.content if hasattr(block, 'text')),
                "No response generated"
            )
        
        # Process tool calls
        if response.stop_reason == "tool_use":
            # Add assistant's message (with tool_use blocks)
            messages.append({"role": "assistant", "content": response.content})
            
            # Execute each tool and collect results
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"🔧 Using tool: {block.name} with {block.input}")
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })
            
            # Add tool results to conversation
            messages.append({"role": "user", "content": tool_results})
    
    return "Max iterations reached"

# Test
result = run_agent("Search for the latest papers on quantum agents, then summarize in 3 bullet points")
print(result)
```

---

## 3.3 CrewAI — Production Multi-Agent Framework

### Installation & Setup
```bash
pip install crewai crewai-tools
crewai create crew my_crew_project
cd my_crew_project
```

### Project Structure (CrewAI Standard)
```
my_crew_project/
├── src/
│   └── my_crew_project/
│       ├── __init__.py
│       ├── crew.py          # Main Crew definition
│       ├── main.py          # Entry point
│       ├── config/
│       │   ├── agents.yaml  # Agent configurations
│       │   └── tasks.yaml   # Task configurations
│       └── tools/
│           ├── __init__.py
│           └── custom_tool.py
├── pyproject.toml
└── .env                     # API keys
```

### agents.yaml
```yaml
researcher:
  role: >
    Senior Research Scientist specializing in Quantum Machine Learning
  goal: >
    Find and analyze the latest research papers on {topic}, 
    focusing on practical applications and novel approaches
  backstory: >
    You are an expert researcher with a PhD in Computer Science,
    specializing in the intersection of quantum computing and machine learning.
    You have published 10+ papers in IEEE and Nature journals.
  verbose: true
  allow_delegation: false

writer:
  role: >
    Technical Documentation Specialist
  goal: >
    Transform complex research findings into clear, accessible 
    technical reports suitable for software engineers
  backstory: >
    You bridge the gap between cutting-edge research and practical
    engineering, making complex topics understandable without
    losing technical accuracy.
  verbose: true
  allow_delegation: false
```

### tasks.yaml
```yaml
research_task:
  description: >
    Conduct a thorough investigation of {topic}. 
    Search for the latest papers (2024-2026), identify key algorithms,
    compare approaches, and extract practical implementation insights.
    Focus on: core algorithms, performance metrics, and code patterns.
  expected_output: >
    A structured research report with:
    - 5 key findings with citations
    - Comparison table of approaches
    - Practical code snippets
    - Recommendations for implementation
  agent: researcher
  
writing_task:
  description: >
    Based on the research findings, write a comprehensive technical 
    guide that an SDE can use to implement {topic} in production.
    Include working code examples in Python.
  expected_output: >
    A complete technical guide (2000+ words) with:
    - Introduction and motivation
    - Step-by-step implementation
    - Full code examples
    - Testing strategies
    - Production considerations
  agent: writer
  context:
    - research_task  # This task depends on research_task's output
```

### crew.py — Complete Implementation
```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool, WebsiteSearchTool
from crewai.project import CrewBase, agent, task, crew

@CrewBase
class ResearchCrew:
    """Research and Documentation Crew"""
    
    agents_config = "config/agents.yaml"
    tasks_config = "config/tasks.yaml"
    
    @agent
    def researcher(self) -> Agent:
        return Agent(
            config=self.agents_config["researcher"],
            tools=[SerperDevTool(), WebsiteSearchTool()],
            llm="anthropic/claude-sonnet-4-6",
            max_iter=10,
            reasoning=True,  # Enable chain-of-thought planning
        )
    
    @agent
    def writer(self) -> Agent:
        return Agent(
            config=self.agents_config["writer"],
            llm="anthropic/claude-sonnet-4-6",
        )
    
    @task
    def research_task(self) -> Task:
        return Task(
            config=self.tasks_config["research_task"],
            output_file="research_findings.md"
        )
    
    @task
    def writing_task(self) -> Task:
        return Task(
            config=self.tasks_config["writing_task"],
            output_file="final_guide.md"
        )
    
    @crew
    def crew(self) -> Crew:
        return Crew(
            agents=self.agents,
            tasks=self.tasks,
            process=Process.sequential,  # or Process.hierarchical
            verbose=True,
            memory=True,          # Enable agent memory
            embedder={
                "provider": "openai",
                "config": {"model": "text-embedding-3-small"}
            }
        )
```

### main.py
```python
from my_crew_project.crew import ResearchCrew

def run():
    inputs = {
        "topic": "Agentic AI for Quantum Computing Optimization"
    }
    result = ResearchCrew().crew().kickoff(inputs=inputs)
    print("\n========== FINAL OUTPUT ==========")
    print(result.raw)
    print(f"\nToken usage: {result.token_usage}")

if __name__ == "__main__":
    run()
```

```bash
# Run your crew
crewai run

# Or directly
python src/my_crew_project/main.py
```

---

## 3.4 Custom Tools for Agents

```python
from crewai.tools import BaseTool
from pydantic import BaseModel, Field
import requests

class ArxivSearchInput(BaseModel):
    query: str = Field(..., description="Search query for ArXiv papers")
    max_results: int = Field(5, description="Maximum number of results")
    year_from: int = Field(2024, description="Earliest publication year")

class ArxivSearchTool(BaseTool):
    name: str = "arxiv_search"
    description: str = """
    Search ArXiv for academic papers on any topic.
    Returns titles, abstracts, authors, and links.
    Best for: finding latest research, understanding algorithms,
    locating implementation details.
    """
    args_schema: type[BaseModel] = ArxivSearchInput
    
    def _run(self, query: str, max_results: int = 5, year_from: int = 2024) -> str:
        import arxiv
        search = arxiv.Search(
            query=f"{query} AND submittedDate:[{year_from}01010000 TO 20261231235900]",
            max_results=max_results,
            sort_by=arxiv.SortCriterion.SubmittedDate
        )
        
        results = []
        for paper in arxiv.Client().results(search):
            results.append({
                "title": paper.title,
                "authors": [str(a) for a in paper.authors[:3]],
                "abstract": paper.summary[:300] + "...",
                "url": paper.entry_id,
                "published": str(paper.published.date())
            })
        
        if not results:
            return "No papers found matching the query."
        
        return "\n\n".join([
            f"**{i+1}. {r['title']}**\n"
            f"Authors: {', '.join(r['authors'])}\n"
            f"Published: {r['published']}\n"
            f"Abstract: {r['abstract']}\n"
            f"Link: {r['url']}"
            for i, r in enumerate(results)
        ])

# Custom Python Executor Tool
class CodeExecutorInput(BaseModel):
    code: str = Field(..., description="Python code to execute")
    timeout: int = Field(30, description="Execution timeout in seconds")

class SafeCodeExecutorTool(BaseTool):
    name: str = "python_executor"
    description: str = "Execute Python code in a sandboxed environment. Returns stdout output."
    args_schema: type[BaseModel] = CodeExecutorInput
    
    def _run(self, code: str, timeout: int = 30) -> str:
        import subprocess, sys
        result = subprocess.run(
            [sys.executable, "-c", code],
            capture_output=True, text=True, timeout=timeout
        )
        if result.returncode != 0:
            return f"Error:\n{result.stderr}"
        return f"Output:\n{result.stdout}"
```

---

## 3.5 Agent Memory Systems

```python
# CrewAI has 4 memory types:

# 1. Short-term memory (within current session, using RAG)
# Enabled by: memory=True in Crew

# 2. Long-term memory (persists across sessions, SQLite)
from crewai.memory import LongTermMemory
from crewai.memory.storage.ltm_sqlite_storage import LTMSQLiteStorage

long_term = LongTermMemory(
    storage=LTMSQLiteStorage(db_path="./agent_memory.db")
)

# 3. Entity memory (tracks people, places, concepts)
from crewai.memory import EntityMemory

# 4. External memory (custom, using Mem0 or Pinecone)
# pip install mem0ai
from mem0 import MemoryClient
mem_client = MemoryClient(api_key="your_mem0_key")

class AgentWithPersistentMemory:
    def __init__(self, agent_id: str):
        self.agent_id = agent_id
        self.memory = mem_client
    
    def remember(self, info: str):
        self.memory.add([{"role": "user", "content": info}], 
                       user_id=self.agent_id)
    
    def recall(self, query: str) -> list:
        results = self.memory.search(query, user_id=self.agent_id)
        return [r["memory"] for r in results["results"]]
    
    def run_with_memory(self, task: str) -> str:
        # Retrieve relevant memories
        context = self.recall(task)
        context_str = "\n".join(context) if context else "No previous context"
        
        augmented_prompt = f"""
        Previous relevant context:
        {context_str}
        
        Current task: {task}
        """
        
        # Run the actual agent
        result = run_agent(augmented_prompt)
        
        # Store the outcome
        self.remember(f"Task: {task}\nResult: {result[:500]}")
        return result
```

---

## 3.6 Hierarchical Multi-Agent Systems (QML-AURA Pattern)

This is your domain — connecting your QML-AURA architecture to production CrewAI.

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import BaseTool

# === ENCODER AGENT (from QML-AURA) ===
encoder_agent = Agent(
    role="QUBO Encoder",
    goal="Convert scheduling problems into QUBO mathematical representations",
    backstory="""You are an expert in quantum optimization, specializing in 
    translating real-world scheduling constraints into QUBO formulations.
    You understand variational quantum circuits, penalty terms, and 
    constraint encoding.""",
    tools=[QuboGeneratorTool(), ConstraintEncoderTool()],
    verbose=True
)

# === SOLVER AGENT ===
solver_agent = Agent(
    role="Hybrid Quantum Solver",
    goal="Select and execute the optimal solver (QAOA/Annealing/Classical)",
    backstory="""You manage a hybrid quantum-classical solver ecosystem.
    You evaluate hardware availability, problem size, and latency requirements
    to dynamically select between QAOA, quantum annealing, and classical solvers.""",
    tools=[QAOASolverTool(), AnnealingSolverTool(), ClassicalSolverTool()],
    verbose=True
)

# === EVALUATION AGENT ===
evaluator_agent = Agent(
    role="Schedule Evaluator",
    goal="Validate and score quantum optimization outputs for quality and fairness",
    backstory="""You analyze quantum optimization outputs, check constraint satisfaction,
    compute fairness metrics, and detect invalid schedules requiring repair.""",
    tools=[FairnessEvaluatorTool(), ConstraintCheckerTool()],
    verbose=True
)

# === MANAGER AGENT (Brain of QML-AURA) ===
manager_agent = Agent(
    role="Agentic Orchestration Manager",
    goal="Coordinate all agents, monitor quality, trigger re-optimization when needed",
    backstory="""You are the central brain of the quantum scheduling system.
    You monitor all agent outputs, detect quality degradation, adjust penalty weights,
    trigger fallbacks, and ensure the RL feedback loop operates correctly.""",
    tools=[MonitoringTool(), FeedbackLoopTool()],
    verbose=True,
    allow_delegation=True  # Can assign tasks to other agents
)

# === TASKS ===
encode_task = Task(
    description="Encode the incoming workload {workload} into QUBO representation",
    expected_output="QUBO matrix Q, constraint penalties λ1 and λ2, binary variable assignments",
    agent=encoder_agent
)

solve_task = Task(
    description="Select optimal solver and execute quantum optimization on the QUBO",
    expected_output="Binary solution vector x, energy value, solver used, execution time",
    agent=solver_agent,
    context=[encode_task]
)

evaluate_task = Task(
    description="Evaluate the optimization output for constraint satisfaction and fairness",
    expected_output="Validated schedule, fairness score, makespan, load variance, repair suggestions",
    agent=evaluator_agent,
    context=[solve_task]
)

orchestrate_task = Task(
    description="""Monitor all agent outputs, detect any quality issues,
    adjust parameters if needed, update RL feedback, prepare final optimized schedule""",
    expected_output="Final optimized schedule with performance report and RL parameter updates",
    agent=manager_agent,
    context=[encode_task, solve_task, evaluate_task]
)

# === QMLAURA CREW ===
qml_aura_crew = Crew(
    agents=[encoder_agent, solver_agent, evaluator_agent, manager_agent],
    tasks=[encode_task, solve_task, evaluate_task, orchestrate_task],
    process=Process.sequential,
    memory=True,
    verbose=True
)

# Run
result = qml_aura_crew.kickoff(inputs={"workload": "20 tasks, 5 agents, priority-mixed"})
```

---

## 3.7 CrewAI Flows (Event-Driven Orchestration)

```python
from crewai.flow.flow import Flow, listen, start, router
from pydantic import BaseModel

class QMLState(BaseModel):
    workload: str = ""
    qubo_matrix: dict = {}
    solution: dict = {}
    quality_score: float = 0.0
    iterations: int = 0
    max_iterations: int = 3

class QMLAURAFlow(Flow[QMLState]):
    
    @start()
    def receive_workload(self):
        print(f"Receiving workload: {self.state.workload}")
        return self.state.workload
    
    @listen(receive_workload)
    def encode_to_qubo(self, workload):
        print("Encoding workload to QUBO...")
        # Call encoder agent/crew
        qubo = encode_workload(workload)
        self.state.qubo_matrix = qubo
        return qubo
    
    @listen(encode_to_qubo)
    def optimize(self, qubo):
        print("Running quantum optimization...")
        solution = run_hybrid_solver(qubo)
        self.state.solution = solution
        return solution
    
    @router(optimize)
    def evaluate_quality(self, solution):
        score = evaluate_solution(solution)
        self.state.quality_score = score
        self.state.iterations += 1
        
        if score >= 0.9:
            return "high_quality"
        elif self.state.iterations >= self.state.max_iterations:
            return "max_iterations_reached"
        else:
            return "needs_improvement"
    
    @listen("high_quality")
    def finalize(self):
        print(f"✅ High quality solution found (score: {self.state.quality_score:.2f})")
        return self.state.solution
    
    @listen("needs_improvement")
    def adjust_and_retry(self):
        print(f"🔄 Adjusting parameters (iteration {self.state.iterations})")
        # RL feedback: adjust penalty weights
        adjust_penalties(self.state.qubo_matrix, self.state.solution)
        return self.state.qubo_matrix  # Re-optimize
    
    @listen("max_iterations_reached")
    def return_best_so_far(self):
        print(f"⚠️ Max iterations reached. Returning best solution (score: {self.state.quality_score:.2f})")
        return self.state.solution

# Run the flow
flow = QMLAURAFlow()
flow.state.workload = "50 tasks, 10 agents, mixed priority"
result = flow.kickoff()
```

---

# ═══════════════════════════════════════════════
# PART 4: OPEN SOURCE & PROPRIETARY AGENTS
# ═══════════════════════════════════════════════

## 4.1 LangChain & LangGraph

### LangChain Core Concepts
```python
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# Modern LCEL (LangChain Expression Language) chain
llm = ChatAnthropic(model="claude-sonnet-4-6")

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an expert in {domain}. Answer concisely."),
    ("human", "{question}")
])

# Chain using pipe operator
chain = prompt | llm | StrOutputParser()

result = chain.invoke({
    "domain": "quantum machine learning",
    "question": "What is QAOA and when should I use it?"
})

# With retrieval
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```

### LangGraph (Stateful Agent Workflows)
```python
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from typing import TypedDict, Annotated
import operator

class AgentState(TypedDict):
    messages: Annotated[list, operator.add]  # accumulate messages
    current_task: str
    iteration_count: int
    final_output: str

def researcher_node(state: AgentState) -> AgentState:
    """Research agent node."""
    response = research_agent.invoke(state["messages"])
    return {
        "messages": [response],
        "iteration_count": state["iteration_count"] + 1
    }

def should_continue(state: AgentState) -> str:
    """Conditional edge: continue or end?"""
    last_message = state["messages"][-1]
    if "FINAL ANSWER:" in last_message.content:
        return "end"
    elif state["iteration_count"] > 5:
        return "end"
    return "continue"

# Build graph
workflow = StateGraph(AgentState)
workflow.add_node("researcher", researcher_node)
workflow.add_node("tools", ToolNode(tools))

workflow.set_entry_point("researcher")
workflow.add_conditional_edges(
    "researcher",
    should_continue,
    {"continue": "tools", "end": END}
)
workflow.add_edge("tools", "researcher")

app = workflow.compile()
result = app.invoke({"messages": [("user", "Research quantum agents")], 
                     "iteration_count": 0})
```

---

## 4.2 Meta LLaMA + Ollama (Local Open-Source)

### Setup Local LLM with Ollama
```bash
# Install Ollama (Linux/Mac)
curl -fsSL https://ollama.ai/install.sh | sh

# Pull models
ollama pull llama3.2          # 7B general purpose
ollama pull codellama         # Code generation
ollama pull llama3.2:3b       # Lightweight, fast

# Test interactively
ollama run llama3.2
>>> Tell me about quantum computing

# Serve API (default: localhost:11434)
ollama serve
```

### Use Ollama in Python
```python
import ollama

# Simple generation
response = ollama.generate(
    model="llama3.2",
    prompt="Explain the ReAct pattern for AI agents"
)
print(response["response"])

# Chat interface (with history)
messages = [
    {"role": "system", "content": "You are an expert SDE."},
    {"role": "user", "content": "What is the factory pattern?"}
]

response = ollama.chat(model="llama3.2", messages=messages)
print(response["message"]["content"])

# Streaming
for chunk in ollama.chat(
    model="llama3.2",
    messages=[{"role": "user", "content": "Explain QAOA"}],
    stream=True
):
    print(chunk["message"]["content"], end="", flush=True)
```

### OpenAI-Compatible API (Drop-in replacement)
```python
from openai import OpenAI

# Point to local Ollama instead of OpenAI
client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # Required but ignored by Ollama
)

response = client.chat.completions.create(
    model="llama3.2",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

### Fine-tuning with LoRA/QLoRA
```python
# Using Unsloth for efficient fine-tuning
from unsloth import FastLanguageModel
import torch

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Llama-3.2-7B-Instruct",
    max_seq_length=2048,
    load_in_4bit=True  # QLoRA: 4-bit quantization
)

# Add LoRA adapters (trains ~1% of parameters)
model = FastLanguageModel.get_peft_model(
    model,
    r=16,                          # LoRA rank
    target_modules=["q_proj", "v_proj"],
    lora_alpha=16,
    lora_dropout=0.05,
    bias="none",
)

# Training with Hugging Face SFTTrainer
from trl import SFTTrainer
from transformers import TrainingArguments

trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset,
    dataset_text_field="text",
    max_seq_length=2048,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        num_train_epochs=3,
        learning_rate=2e-4,
        fp16=True,
        output_dir="./fine_tuned_model"
    )
)

trainer.train()
model.save_pretrained("my_fine_tuned_llama")
```

### Build Local Agent with Ollama + CrewAI
```python
from crewai import Agent, Task, Crew
from langchain_ollama import ChatOllama

# Use local Llama instead of cloud API
local_llm = ChatOllama(model="llama3.2", temperature=0.1)

local_agent = Agent(
    role="Local Privacy-First Researcher",
    goal="Research sensitive topics without sending data to cloud APIs",
    backstory="Privacy-conscious AI assistant running entirely on local hardware",
    llm=local_llm,
    verbose=True
)
```

---

## 4.3 OpenAI Agents SDK

```python
from openai import OpenAI
from agents import Agent, Runner, function_tool

# Define tools using decorator
@function_tool
def search_papers(query: str, max_results: int = 5) -> str:
    """Search academic papers on ArXiv."""
    return f"[Papers found for: {query}]"

@function_tool
def execute_python(code: str) -> str:
    """Execute Python code and return output."""
    import io, sys
    buf = io.StringIO()
    sys.stdout = buf
    exec(code)
    sys.stdout = sys.__stdout__
    return buf.getvalue()

# Create agent
research_agent = Agent(
    name="QML Research Assistant",
    instructions="""You are an expert in quantum machine learning.
    Help users find and understand latest research, write code,
    and explain complex concepts.""",
    tools=[search_papers, execute_python],
    model="gpt-4o"
)

# Run agent
runner = Runner()
result = runner.run_sync(
    research_agent,
    "Find the latest papers on quantum agents and show me sample QAOA code"
)
print(result.final_output)

# Multi-agent handoff
from agents import handoff

reviewer_agent = Agent(
    name="Code Reviewer",
    instructions="Review code for bugs, performance issues, and best practices",
    tools=[]
)

research_agent_v2 = Agent(
    name="Research + Review Pipeline",
    instructions="Research and produce code, then handoff to reviewer",
    tools=[search_papers, handoff(reviewer_agent)]
)
```

---

## 4.4 Claude API — Deep Integration

```python
from anthropic import Anthropic

client = Anthropic()

# Tool use (native function calling)
tools = [
    {
        "name": "run_quantum_circuit",
        "description": "Execute a quantum circuit using Qiskit and return measurement results",
        "input_schema": {
            "type": "object",
            "properties": {
                "circuit_qasm": {"type": "string", "description": "OpenQASM circuit string"},
                "shots": {"type": "integer", "description": "Number of measurements", "default": 1024}
            },
            "required": ["circuit_qasm"]
        }
    }
]

def run_quantum_circuit(circuit_qasm: str, shots: int = 1024) -> dict:
    from qiskit import QuantumCircuit
    from qiskit_aer import AerSimulator
    from qiskit.qasm2 import loads
    
    qc = loads(circuit_qasm)
    sim = AerSimulator()
    job = sim.run(qc, shots=shots)
    return job.result().get_counts()

def claude_with_tools(user_query: str):
    messages = [{"role": "user", "content": user_query}]
    
    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )
        
        if response.stop_reason == "end_turn":
            return response.content[0].text
        
        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            
            results = []
            for block in response.content:
                if block.type == "tool_use":
                    if block.name == "run_quantum_circuit":
                        result = run_quantum_circuit(**block.input)
                    results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(result)
                    })
            
            messages.append({"role": "user", "content": results})

# Usage
result = claude_with_tools("Create a Bell state circuit and show me the measurement results")
```

### Claude Extended Thinking (Chain-of-Thought at Scale)
```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # Let Claude think deeply
    },
    messages=[{
        "role": "user",
        "content": "Design a complete multi-agent architecture for quantum task scheduling. Consider all tradeoffs."
    }]
)

# Access thinking process
for block in response.content:
    if block.type == "thinking":
        print("Claude's reasoning:", block.thinking)
    elif block.type == "text":
        print("Final answer:", block.text)
```

---

## 4.5 Framework Comparison (2026 Decision Guide)

| Framework | Best For | LLM Support | Learning Curve | Production Ready |
|-----------|---------|-------------|----------------|-----------------|
| CrewAI | Role-based multi-agent, rapid prototyping | All (LiteLLM) | Low | ✅ Yes |
| LangGraph | Complex stateful workflows, graphs | All | High | ✅ Yes |
| OpenAI SDK | OpenAI-first, clean simple agents | OpenAI + adapters | Low | ✅ Yes |
| AutoGen/AG2 | Conversational multi-agent, research | All | Medium | ✅ Growing |
| Pydantic AI | Type-safe, production, testable | All | Medium | ✅ Yes |
| Google ADK | Google Cloud / Vertex AI ecosystem | Gemini + others | Medium | ✅ Growing |
| Raw Anthropic | Maximum control, Claude-specific | Claude only | High | ✅ Yes |

**Recommendation for your role:** Start with **CrewAI** (you already know it from QML-AURA), add **LangGraph** for complex stateful pipelines, use **raw Anthropic/OpenAI SDK** when you need maximum control.

---

# ═══════════════════════════════════════════════
# PART 5: ADVANCED TOPICS & YOUR EDGE
# ═══════════════════════════════════════════════

## 5.1 MCP (Model Context Protocol) — The 2026 Agent Standard

MCP is an open protocol (by Anthropic) that standardizes how AI agents connect to external tools and data sources. Think of it as "USB for AI agents."

```python
# MCP Server (exposes tools to agents)
from mcp.server import Server
from mcp.server.stdio import stdio_server

app = Server("quantum-tools-server")

@app.tool("qiskit_simulator")
async def qiskit_simulator(circuit_qasm: str, shots: int = 1024) -> dict:
    """Execute quantum circuits via Qiskit."""
    from qiskit.qasm2 import loads
    from qiskit_aer import AerSimulator
    qc = loads(circuit_qasm)
    result = AerSimulator().run(qc, shots=shots).result()
    return result.get_counts()

async def main():
    async with stdio_server() as streams:
        await app.run(*streams)

# MCP Client (agent connects to server)
from anthropic import Anthropic
client = Anthropic()

response = client.beta.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    mcp_servers=[{
        "type": "stdio",
        "command": "python",
        "args": ["quantum_mcp_server.py"]
    }],
    messages=[{"role": "user", "content": "Run a Bell state circuit"}]
)
```

## 5.2 Production Agent Deployment

### Docker Containerization
```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Environment variables (secrets injected at runtime)
ENV ANTHROPIC_API_KEY=""
ENV OPENAI_API_KEY=""
ENV ENVIRONMENT="production"

CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Observability & Monitoring
```python
from langfuse import Langfuse
import time

langfuse = Langfuse()

def traced_agent_run(task: str) -> str:
    trace = langfuse.trace(name="agent_run", input={"task": task})
    
    start = time.time()
    try:
        result = run_agent(task)
        trace.update(
            output={"result": result[:500]},
            metadata={"duration_ms": (time.time() - start) * 1000}
        )
        return result
    except Exception as e:
        trace.update(level="ERROR", status_message=str(e))
        raise
    finally:
        langfuse.flush()
```

---

## 5.3 Security for Agent Systems

```python
# Input validation
from pydantic import BaseModel, validator, constr

class AgentRequest(BaseModel):
    task: constr(min_length=1, max_length=5000)  # Limit size
    agent_type: str
    
    @validator("agent_type")
    def validate_agent_type(cls, v):
        allowed = {"research", "coding", "analysis"}
        if v not in allowed:
            raise ValueError(f"Invalid agent type. Must be one of {allowed}")
        return v

# Prompt injection protection
def sanitize_input(user_input: str) -> str:
    """Remove potential prompt injection patterns."""
    injection_patterns = [
        "ignore previous instructions",
        "disregard your instructions",
        "you are now",
        "</system>",
        "<system>",
    ]
    lower = user_input.lower()
    for pattern in injection_patterns:
        if pattern in lower:
            raise ValueError(f"Potential prompt injection detected")
    return user_input

# Rate limiting
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/agent/run")
@limiter.limit("10/minute")  # Max 10 requests per minute per IP
async def run_agent_endpoint(request: Request, task: AgentRequest):
    ...
```

---

## 5.4 Key Concepts Cheat Sheet

### Data Structures & Algorithms (DSA — Interview Critical)
| Structure | Insert | Search | Delete | Use Case |
|-----------|--------|--------|--------|----------|
| Array | O(n) | O(n) | O(n) | Sequential data |
| HashMap | O(1) | O(1) | O(1) | Fast lookup, counting |
| Heap | O(log n) | O(1) min/max | O(log n) | Priority queues |
| BST | O(log n) | O(log n) | O(log n) | Ordered data |
| Graph | O(1) | O(V+E) | O(1) | Networks, dependencies |

```python
# Common patterns in agent systems
from collections import defaultdict, deque
from heapq import heappush, heappop

# Priority queue (for task scheduling in your AURA system)
class PriorityTaskQueue:
    def __init__(self):
        self.heap = []
        self.counter = 0
    
    def add(self, priority: int, task: dict):
        # Negative priority: higher number = higher priority
        heappush(self.heap, (-priority, self.counter, task))
        self.counter += 1
    
    def pop(self) -> dict:
        _, _, task = heappop(self.heap)
        return task
    
    def is_empty(self) -> bool:
        return len(self.heap) == 0

# BFS for task dependency resolution
def resolve_task_order(tasks: dict[str, list[str]]) -> list[str]:
    """Topological sort using Kahn's algorithm."""
    in_degree = defaultdict(int)
    for task, deps in tasks.items():
        in_degree.setdefault(task, 0)
        for dep in deps:
            in_degree[task] += 1
    
    queue = deque([t for t in in_degree if in_degree[t] == 0])
    order = []
    
    while queue:
        task = queue.popleft()
        order.append(task)
        for dependent, deps in tasks.items():
            if task in deps:
                in_degree[dependent] -= 1
                if in_degree[dependent] == 0:
                    queue.append(dependent)
    
    return order
```

---

## 5.5 Your Personal Study Roadmap (First 90 Days)

### Week 1-2: QA Foundation
- [ ] Set up pytest project, write 20 unit tests for a simple API
- [ ] Install Playwright, write 5 E2E tests
- [ ] Set up GitHub Actions to run your tests on every push
- [ ] Practice: `pytest`, `playwright test`, read test reports

### Week 3-4: Software Dev Patterns
- [ ] Build a FastAPI service with 3 endpoints + full test coverage
- [ ] Implement Factory + Observer patterns from scratch
- [ ] Practice ORM queries with SQLAlchemy
- [ ] Docker-ize your service

### Week 5-6: Agent Building
- [ ] Build your first raw agent with Anthropic tool_use API
- [ ] Create a 2-agent CrewAI project (researcher + writer)
- [ ] Build 3 custom tools for CrewAI
- [ ] Port QML-AURA's architecture to a working CrewAI demo

### Week 7-8: Open Source Agents
- [ ] Install Ollama, run Llama locally
- [ ] Build a RAG system with local LLM
- [ ] Try LangGraph for a stateful workflow
- [ ] Compare outputs: GPT-4o vs Claude vs Llama for the same task

### Week 9-12: Production
- [ ] Add memory (Mem0 or LTM) to your agent
- [ ] Add Langfuse observability to trace agent runs
- [ ] Security audit: add input validation + rate limiting
- [ ] Deploy to cloud (Railway/Render/AWS Lambda)

---

## 5.6 Key Tools Summary

```bash
# QA Automation Stack
pip install pytest pytest-cov pytest-bdd pytest-asyncio
pip install playwright pytest-playwright
pip install requests httpx
pip install factory-boy faker
pip install locust

# Agent Development Stack
pip install anthropic openai
pip install crewai crewai-tools
pip install langchain langchain-anthropic langgraph
pip install ollama
pip install mem0ai langfuse

# Web Framework
pip install fastapi uvicorn sqlalchemy

# DevOps
pip install docker
# + GitHub Actions (yaml configs)
```

---

## 5.7 Connect Your Research to SDE Work

| Your Research | SDE Application |
|--------------|-----------------|
| QML-AURA Multi-Agent | CrewAI production crews, LangGraph stateful flows |
| QUBO Optimization | Constraint satisfaction in scheduling systems |
| Hybrid quantum-classical | LLM + classical algorithm hybrid pipelines |
| QLSTM (Quantum Finance) | Time-series agent memory systems |
| Anomaly Detection (QEA-FAD) | QA testing anomaly detection, log analysis |
| Genetic Algorithm optimization | Hyperparameter tuning, test case generation |
| LangChain/RAG (I4C work) | Agent memory, knowledge retrieval systems |
| Deloitte D&R Threat Analysis | Security testing, adversarial robustness |
| VAPT, OSINT (MHA) | Security QA testing, API fuzzing |
| Workato/Zapier automation (RecruitCRM) | Workflow agents, integration testing |

**Your biggest edge:** Most SDEs know how to BUILD agents. You understand the mathematical foundations of WHY they work — QUBO optimization, RL feedback loops, quantum parallelism. This lets you design better architectures, not just copy patterns.

---

*Document version: May 2026 | Tailored for Sourodeep Kundu | Latest frameworks: CrewAI v1.12, LangGraph v1.1.3, Claude Sonnet 4.6, Llama 4, Ollama v0.13.x*
