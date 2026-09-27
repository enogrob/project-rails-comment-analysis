# **Project Rails Comment Analysis**

[Homepage](https://github.com/enogrob/rails_comment_analysis)

![rails image](images/project.png)


## Contents

- [Summary](#summary)
- [Architecture](#architecture)
  - [Decisions made](#decisions-made)
  - [Statistical formulas](#statistical-formulas)
  - [Example requests](#example-requests)
  - [Key Concepts](#key-concepts)
- [Tech Stack](#tech-stack)
- [References](#references)

---

## Summary

**Rails Comment Analysis** is a Ruby on Rails 8 API application designed to automate the import, translation, analysis, and approval of user-generated comments from external sources. The system fetches user, post, and comment data from a public API, translates comment bodies, and applies keyword-based approval logic. It is built for developers, data analysts, and teams seeking to automate comment moderation and gain insights into comment quality and trends.

**Key features:**
- **Automated import** of users, posts, and comments from external APIs
- **Real-time translation** of comment bodies using LibreTranslate
- **Keyword-based** comment approval and rejection
- **Statistical analysis** of comment lengths (mean, median, stddev)
- **RESTful and legacy API endpoints** for analysis, progress, and keyword management
- **Background processing** with Sidekiq and Redis for scalable analysis
- **Full test coverage** with RSpec and SimpleCov

**Relevance:**
The project demonstrates best practices in Rails API design (well-aligned with SOLID principles) background job processing, and test-driven development. Its modular, service-oriented structure and use of modern gems make it a strong reference for scalable, maintainable API projects.


## Architecture


### System Overview

```mermaid
graph TD
  subgraph API_Layer["🌐 API Layer"]
    AnalysisController["🌐 AnalysisController"] -->|starts job| AnalyzeUserWorker
    AnalyzeController["🌐 AnalyzeController"] -->|imports through| ImportUserDataService
    KeywordsController["🌐 KeywordsController"] -->|manages| Keyword
    ProgressController["🌐 ProgressController"] -->|reads status from| User
  end
  subgraph Services["⚙️ Services"]
    ImportUserDataService[["⚙️ ImportUserDataService"]] -->|translates with| TranslateService
    ImportUserDataService -->|applies| CommentApprovalService
    ImportUserDataService -->|imports| User
    ImportUserDataService -->|imports| Post
    ImportUserDataService -->|imports| Comment
    CommentApprovalService -->|evaluates| Comment
    CommentMetricsService[["⚙️ CommentMetricsService"]] -->|measures| Comment
    CommentMetricsService -->|calculates for| User
  end
  subgraph Models["🗃️ Models"]
    User[("👤 User")] -->|has many| Post[("📝 Post")]
    Post -->|has many| Comment[("💬 Comment")]
    Keyword[("🔑 Keyword")]
  end
  subgraph Workers["🧵 Workers"]
    AnalyzeUserWorker{{"🧵 AnalyzeUserWorker"}} -->|runs| ImportUserDataService
  end
  subgraph External["🌍 External APIs"]
    JSONPlaceholder(["🌍 JSONPlaceholder"])
    LibreTranslate(["🌍 LibreTranslate API"])
    ImportUserDataService -->|fetches from| JSONPlaceholder
    TranslateService -->|sends translation requests to| LibreTranslate
  end
  KeywordsController -->|informs| CommentApprovalService

  classDef process fill:#DCEBFA,stroke:#355C7D,color:#1E293B;
  classDef data fill:#DDF2E1,stroke:#3F6B4F,color:#1E3324;
  classDef external fill:#FBE4F0,stroke:#8E496D,color:#3F2434;
  class AnalysisController,AnalyzeController,KeywordsController,ProgressController,ImportUserDataService,TranslateService,CommentApprovalService,CommentMetricsService,AnalyzeUserWorker process;
  class User,Post,Comment,Keyword data;
  class JSONPlaceholder,LibreTranslate external;
  linkStyle default stroke:#52606D,stroke-width:1.5px;
  style API_Layer fill:#EDF4FB,stroke:#355C7D,color:#1E293B;
  style Services fill:#EDF4FB,stroke:#355C7D,color:#1E293B;
  style Models fill:#EAF5EC,stroke:#3F6B4F,color:#1E3324;
  style Workers fill:#EDF4FB,stroke:#355C7D,color:#1E293B;
  style External fill:#FCECF3,stroke:#8E496D,color:#3F2434;
```


#### Alternative Perspectives

<details>
<summary>Gems Dependency Diagram</summary>

```mermaid
graph TD
  Rails["💎 Rails"] -->|includes| ActiveRecord["🗄️ ActiveRecord"]
  Rails -->|uses| Sidekiq["🧵 Sidekiq"]
  Rails -->|uses| Redis[("📦 Redis")]
  Rails -->|uses| HTTParty["🌐 HTTParty"]
  Rails -->|uses| AASM["🔄 AASM"]
  Rails -->|uses| RSpec["🧪 RSpec"]
  Rails -->|uses| FactoryBot["🏭 FactoryBot"]
  Rails -->|uses| SimpleCov["📊 SimpleCov"]
  Sidekiq -->|queues jobs in| Redis
  ImportUserDataService[["⚙️ ImportUserDataService"]] -->|makes requests with| HTTParty
  TranslateService[["⚙️ TranslateService"]] -->|makes requests with| HTTParty
  Comment["💬 Comment"] -->|uses| AASM
  CommentMetricsService[["⚙️ CommentMetricsService"]] -->|caches in| Redis
  AnalyzeUserWorker{{"🧵 AnalyzeUserWorker"}} -->|runs on| Sidekiq

  classDef process fill:#DCEBFA,stroke:#355C7D,color:#1E293B;
  classDef data fill:#DDF2E1,stroke:#3F6B4F,color:#1E3324;
  class Rails,ActiveRecord,Sidekiq,HTTParty,AASM,RSpec,FactoryBot,SimpleCov,ImportUserDataService,TranslateService,Comment,CommentMetricsService,AnalyzeUserWorker process;
  class Redis data;
  linkStyle default stroke:#52606D,stroke-width:1.5px;
```

</details>

<details>
<summary>Dependencies and Models</summary>

```mermaid
erDiagram
    USER ||--o{ POST : has_many
    POST ||--o{ COMMENT : has_many
    COMMENT }o--|| KEYWORD : triggers_approval
```

</details>

<details>
<summary>Mind Map - Interconnected Themes</summary>

```mermaid
mindmap
  root((🗨️ Project Comment Analysis))
    🌐 API
      🧭 AnalysisController
      📥 AnalyzeController
      🔑 KeywordsController
      📈 ProgressController
    ⚙️ Services
      📦 ImportUserDataService
      🌐 TranslateService
      ✅ CommentApprovalService
      📊 CommentMetricsService
    🗃️ Models
      👤 User
      📝 Post
      💬 Comment
      🔑 Keyword
    🧵 Workers
      🔄 AnalyzeUserWorker
    🌍 External APIs
      🧪 JSONPlaceholder
      🌐 LibreTranslate
    🧪 Testing
      ✅ RSpec
      🏭 FactoryBot
      📊 SimpleCov
    ⏱️ Background Jobs
      🧵 Sidekiq
      📦 Redis
    🚢 Deployment
      🐳 Docker
```

</details>

<details>
<summary>Deployment Architecture</summary>

```mermaid
graph LR
  Client(["👤 Client"]) -->|sends requests to| RailsAPI["🌐 Rails API App"]
  RailsAPI -->|enqueues jobs with| Sidekiq{{"🧵 Sidekiq"}}
  RailsAPI -->|caches through| Redis[("📦 Redis")]
  RailsAPI -->|persists data in| DB[("🗄️ SQLite3")]
  Sidekiq -->|uses queue in| Redis
  RailsAPI -->|translates with| LibreTranslate(["🌍 LibreTranslate API"])
  RailsAPI -->|imports from| JSONPlaceholder(["🌍 JSONPlaceholder API"])
  RailsAPI -.->|containerized by| Docker["🐳 Docker"]
  Sidekiq -.->|containerized by| Docker
  Redis -.->|containerized by| Docker
  DB -.->|containerized by| Docker

  classDef process fill:#DCEBFA,stroke:#355C7D,color:#1E293B;
  classDef data fill:#DDF2E1,stroke:#3F6B4F,color:#1E3324;
  classDef external fill:#FBE4F0,stroke:#8E496D,color:#3F2434;
  class RailsAPI,Sidekiq,Docker process;
  class Redis,DB data;
  class Client,LibreTranslate,JSONPlaceholder external;
  linkStyle default stroke:#52606D,stroke-width:1.5px;
```

</details>

<details>
<summary>Git Graph</summary>

```mermaid
gitGraph
  commit id: "💎 rails-initial-setup"
  commit id: "🧪 rspec-initial-setup"
  commit id: "🗃️ model-initial-setup"
  commit id: "⚙️ service-initial-setup"
  commit id: "🌐 controller-routes-initial-setup"
  commit id: "🧵 add-sidekiq-redis"
  commit id: "✅ add-tests"
  commit id: "📊 add-simplecov"
  commit id: "📘 add-readme"
```

</details>

#### Decisions made

- **Layered architecture:** controllers, services, models, workers
- **Service objects** encapsulate business logic (import, metrics, approval, translation)
- **Sidekiq** for background jobs; Redis for caching and queueing
- **RESTful and legacy endpoints** for compatibility
- **Full test coverage** enforced

#### Statistical formulas

- **Mean:** sum of comment body lengths / number of comments
- **Median:** middle value of sorted comment body lengths
- **Standard deviation:** sqrt of average squared difference from mean

#### Example requests

<details>
<summary>Start analysis for a user</summary>

```bash
curl -X POST \
  http://localhost:3000/analysis \
  -H 'Content-Type: application/json' \
  -d '{"username": "Bret"}'
# Response: { "job_id": "...", "message": "Analysis started for Bret" }
```

</details>

<details>
<summary>Check job progress</summary>

```bash
curl http://localhost:3000/progress/<job_id>
# Replace <job_id> with the value returned from the previous request
# Response: { "job_id": "...", "progress": "100%" }
```

</details>

<details>
<summary>Add a keyword</summary>

```bash
curl -X POST \
  http://localhost:3000/keywords \
  -d 'keyword[word]=foo'
# Response: { "id": ..., "word": "foo", ... }
```

</details>

<details>
<summary>List all keywords</summary>

```bash
curl http://localhost:3000/keywords
# Response: [ { "id": ..., "word": "foo" }, ... ]
```

</details>

<details>
<summary>Update a keyword</summary>

```bash
curl -X PUT \
  http://localhost:3000/keywords/1 \
  -d 'keyword[word]=bar'
# Response: { "id": 1, "word": "bar", ... }
```

</details>

<details>
<summary>Delete a keyword</summary>

```bash
curl -X DELETE http://localhost:3000/keywords/1
# Response: (204 No Content)
```

</details>

#### Key Concepts

- **ImportUserDataService:** Imports user, post, and comment data from external API
- **TranslateService:** Translates comment bodies to Portuguese
- **CommentApprovalService:** Approves/rejects comments based on keyword matches
- **CommentMetricsService:** Calculates statistics for comment lengths
- **AnalyzeUserWorker:** Runs import and metrics in background


## Tech Stack

- Ruby 3.2.2
- Rails 8.0.2 (API mode)
- Sidekiq 8.0 (background jobs)
- Redis 5.4 (caching, queueing)
- HTTParty (HTTP requests)
- AASM (state machines)
- RSpec, FactoryBot, SimpleCov (testing)
- Docker (deployment)


## References

- [Project Homepage](https://github.com/enogrob/rails_comment_analysis)
- [Rails Guides](https://guides.rubyonrails.org/)
- [Sidekiq Documentation](https://sidekiq.org/)
- [LibreTranslate API](https://libretranslate.com/docs/)
- [jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com/)

