# GitLab Sidekiq Basics

GitLab Sidekiq is a crucial component responsible for asynchronous job processing. It handles background tasks, ensuring the main GitLab application remains responsive.

## What is Sidekiq?

Sidekiq is a background job processing framework for Ruby, based on Redis. GitLab leverages Sidekiq to execute tasks that don't need immediate attention, such as:

* Sending emails
* Processing repository updates
* Running CI/CD pipelines
* And many other background operations

## Key Concepts

### Queues

* Sidekiq uses queues to organize jobs.
* Jobs are enqueued into specific queues based on their type and priority.
* GitLab configures numerous queues to manage different workloads.

    **Example Queue Configuration (Conceptual):**

    ```ruby
    # gitlab.rb (simplified)
    sidekiq['queue_groups'] = [
      { name: "mailers", limit: 5 },
      { name: "pipeline_processing", limit: 10 },
      { name: "default", limit: 20 }
    ]
    ```

### Workers

* Sidekiq workers are the processes that pick up and execute jobs from the queues.
* GitLab runs multiple Sidekiq processes, each potentially processing jobs from one or more queues.
* The number of concurrent workers can be configured.

### Jobs

* A job represents a specific background task to be executed.
* Jobs are Ruby classes that define the work to be done.

    **Example Job (Conceptual):**

    ```ruby
    # app/workers/send_notification_email_worker.rb
    class SendNotificationEmailWorker
      include Sidekiq::Worker

      def perform(user_id, subject, body)
        user = User.find(user_id)
        NotificationMailer.email(user, subject, body).deliver_now
      end
    end
    ```

    To enqueue this job:

    ```ruby
    SendNotificationEmailWorker.perform_async(123, "Important Update", "Check out the latest news!")
    ```

### Redis

* Sidekiq relies on Redis as its message broker and job storage.
* Redis holds the queues and the state of the jobs.
* A healthy Redis connection is essential for Sidekiq to function correctly.

## Monitoring Sidekiq

Monitoring Sidekiq is crucial for GitLab health. Key metrics include:

| Metric           | Description                                    |
| :--------------- | :--------------------------------------------- |
| Queue Length     | Number of jobs waiting in each queue           |
| Processing Jobs  | Number of jobs currently being executed        |
| Failed Jobs      | Number of jobs that have failed                |
| Latency          | Time jobs spend in the queue before processing |
| CPU/Memory Usage | Resource consumption of Sidekiq processes      |

## Troubleshooting Common Issues

* **High Queue Length:** Indicates potential bottlenecks or overloaded queues.
* **High Failure Rate:** Suggests problems with the job logic or external dependencies.
* **Stuck Jobs:** Jobs that remain in the processing state for an unusually long time.
* **Redis Connection Issues:** Can prevent Sidekiq from processing jobs.

## Configuration

Sidekiq behavior can be configured in GitLab's configuration file (`gitlab.rb`) or via environment variables. This includes:

* Number of Sidekiq processes
* Queues each process handles
* Concurrency (number of threads per process)
* Redis connection details

```ruby
# gitlab.rb (partial example)
sidekiq['concurrency'] = 25
sidekiq['queues'] = %w[mailers:5 pipeline_processing:10 default:20]
```


```go
+---------------------+     (1. User Interaction)     +---------------------+
| User (Web Browser)  |---------------------------->| Nginx (Web Server)  |
+---------------------+                               +---------------------+
                                                          | (HTTP Request)
                                                          v
                                                +---------------------+
                                                | Puma (Rack Server)  |
                                                +---------------------+
                                                          | (Passes Request)
                                                          v
                                                +---------------------+
                                                | GitLab (Rails App)  |
                                                +---------------------+
                                          /|\          |          \
                                         / | \         |           \
     (2. Data Storage & Retrieval)       /  |  \        |            \    (3. Background Jobs)
+---------------------+         +---------------------+  |     +---------------------+
| PostgreSQL          |<--------| Praegect (Caching)  |<-'     | Sidekiq (Job Queue) |
| (Main Database)     |---------| (Rails Cache Layer)|        +---------------------+
+---------------------+         +---------------------+                | (Pulls & Processes Jobs)
                                         ^                             v
                                         |                             +---------------------+
                                         |                             | GitLab (Rails App)  |
                                         |                             +---------------------+
                                         |                                       | (Triggers Gitaly/Workhorse)
                                         |                                       v
     (4. Repository Operations)            |                       +---------------------+     (5. File Uploads/Downloads)
+---------------------+                   |                       | Gitaly (Git Service)|-----+---------------------+
| gitlab-shell        |-------------------+                       +---------------------+     | Workhorse (Proxy)   |
| (SSH Access, Hooks)|                                                         | (Git Repository Data) |     +---------------------+
+---------------------+                                                         v                               | (Proxies File I/O)
                                                                      +---------------------+                               v
                                                                      | PostgreSQL          |---------------------------->| Client (Browser)  |
                                                                      | (Repository Meta)   |                               +---------------------+
                                                                      +---------------------+

+---------------------+     (6. Real-time Data & Caching)     +---------------------+
| Redis               |<------------------------------------->| GitLab (Rails App)  |
| (Cache, Queues, Pub/Sub)|                                   +---------------------+

+---------------------+     (7. Monitoring & Observability)
| Prometheus          |------------------------------------->| GitLab (Rails App)  |
| (Metrics Collection)|                                   +---------------------+
+---------------------+                                         | (Scrapes Metrics)
                                                                  v
                                                        +---------------------+
                                                        | Grafana             |
                                                        | (Data Visualization)|
                                                        +---------------------+
                                                                  ^
                                                                  | (Queries & Visualizes Metrics)
                                                        +---------------------+
                                                        | User (Web Browser)  |
                                                        +---------------------+
```

