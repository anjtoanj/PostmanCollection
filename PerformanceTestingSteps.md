1. Create postman collection
2. Declare environment and collection variables. Here am only using collection variables
3. Export the collection as json file
cat-api-performance.json
5. Open GitBash and navigate to the location
6. Install node
 node -v
 npm -v
 newman install
7. _Run the collection in newman_
   newman run
  
   newman run "cat-api-performance.json" --reporters cli,html --reporter-html-export "report.html"  
  <img width="933" height="643" alt="image" src="https://github.com/user-attachments/assets/aba9e66f-364d-4cab-a61c-50bef5093550" />

# Run Multiple Iterations

_This simulates multiple requests to the API sequentially._
newman run "cat-api-performance.json" --iteration-count 50
--iteration-count 50 → Runs the entire collection 50 times sequentially.

# Add Delay Between Requests

_To simulate real-world wait times:_
newman run "cat-api-performance.json" --iteration-count 50 --delay-request 200
--delay-request 200 → Wait 200 ms between requests.

# Run Concurrent Requests (Parallel)

Newman itself doesn’t natively support true concurrency, but you can achieve it with --delay-request + multiple terminal sessions, or using Newman + Node.js scripts.
Example with npx newman-reporter-load (community reporter for concurrency):

npm install -g newman-reporter-load
newman run "cat-api-performance.json" --reporters cli,load --iteration-count 100
This generates a load report with metrics like requests/sec, min/max/avg response times.

# Run Multiple Newman Instances in Parallel (Simulate Concurrency)

_Open multiple terminals and run Newman simultaneously:_
newman run "cat-api-performance.json" --iteration-count 50
newman run "cat-api-performance.json" --iteration-count 50

Each instance simulates a separate user
Combine results manually or with HTML reports

# Simulate Concurrency (Pseudo Load Testing)

_Newman itself is sequential, but you can simulate concurrency by running multiple Newman processes:_
Windows (PowerShell):
Start-Process "newman" -ArgumentList 'run "cat-api-performance.json" --iteration-count 50'
Start-Process "newman" -ArgumentList 'run "cat-api-performance.json" --iteration-count 50'

# To run alongwith env variables
        run: |
          newman run cat-api-performace.json \
          -e CatAPI.postman_environment.json \
          --iteration-count 100 \
          --reporters cli,html \
          --reporter-html-export newman/report.html

# Newman itself does not impose a hard maximum iteration count. The limit depends on system resources and CI environment constraints.

1. Newman Limits
No built-in iteration cap: --iteration-count can be very large (1000, 10000…).
Memory usage increases with iteration count, especially if:
You store large responses
You generate reports (HTML/JSON)
Your collection has many requests or heavy scripts

2. CI/CD Runner Limits
GitHub-Hosted Runners
Each job has:
2-core CPU
7 GB RAM
6-hour maximum runtime per job

Practical limit:
10,000+ iterations may run, but could hit memory or timeout issues.
Recommended to split into parallel jobs using matrix strategy for large loads.

Self-Hosted Runners
Limited only by your machine’s CPU, RAM, and disk.
You can theoretically run millions of iterations if resources allow.

3. Best Practices for CI
Avoid huge single jobs → split into parallel Newman jobs.
Use --delay-request for better simulation of real users.

Monitor memory and runtime; CI may cancel jobs that exceed time/memory.
Limit HTML/JSON reporting for very large iteration counts:
_newman run collection.json --iteration-count 5000 --reporters cli_

Generate HTML/JSON for smaller sample runs if needed.
Rule of Thumb

100–500 iterations per job → safe and fast
500–5000 iterations per job → monitor memory and CI runtime
>5000 iterations → use matrix/parallel jobs
-------------------------------------------------------------------
# ParallelPerformance.yml
*******************************
1. Matrix Strategy
name: API Performance Test
on: push
jobs:
  performance-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        parallel-job: [1, 2, 3, 4]   # 4 parallel jobs

GitHub creates 4 separate jobs, one for each value in the matrix (1, 2, 3, 4).
All 4 jobs run in parallel on separate Ubuntu runners (ubuntu-latest).
Each job gets its own isolated VM with 2 CPUs, 7 GB RAM, and a clean environment.

*******************************
2. Checkout Step
- uses: actions/checkout@v3

Pulls your repository into each job’s runner.
Each job operates independently, so each has a separate copy of your code and Postman collection.
*******************************
3. How the Jobs Run

Job 1 → runs on Ubuntu VM 1
Job 2 → runs on Ubuntu VM 2
Job 3 → runs on Ubuntu VM 3
Job 4 → runs on Ubuntu VM 4
You can add Newman commands in each job to run a subset of iterations.
Each job can generate its own HTML/JSON report.
*******************************
4. Example with Newman
- name: Run Performance Test
  run: |
    mkdir -p newman
    echo "Running job ${{ matrix.parallel-job }}"
    newman run cat-api-performance.json \
      --iteration-count 25 \
      --reporters cli,html \
      --reporter-html-export newman/report-${{ matrix.parallel-job }}.html


Iteration split: Each job runs 25 iterations → total 100 iterations across 4 jobs.
Reports: Each job produces report-1.html, report-2.html, etc.
*******************************
5. Upload Artifacts
- name: Upload Test Report
  uses: actions/upload-artifact@v3
  with:
    name: performance-report-${{ matrix.parallel-job }}
    path: newman/report-${{ matrix.parallel-job }}.html

Each job uploads its report independently.
*******************************
After workflow finishes, you can download all 4 reports from the Actions artifacts tab.
_Matrix creates parallel jobs on GitHub-hosted runners.
Each job is isolated, runs independently, and can run a portion of your Newman iterations.
Reports are generated per job, giving you a pseudo-concurrent load test._
*******************************

# To run Postman Newman performance tests in Azure DevOps, you create a pipeline using either a YAML file or a classic pipeline. Below is a YAML-based setup.

1. Prerequisites
Azure DevOps project created.
Postman collection (cat-api-performance.json) and environment files (optional) stored in the repo.
Newman installed in pipeline (via npm).

2. Sample Azure DevOps Pipeline (YAML)
trigger:
- main   # or your target branch

pool:
  vmImage: 'ubuntu-latest'

jobs:
- job: PerformanceTest
  displayName: 'API Performance Test with Newman'
  strategy:
    parallel: 4     # Runs 4 parallel jobs
  steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '18.x'
    displayName: 'Install Node.js'

  - script: |
      npm install -g newman
      mkdir -p newman
    displayName: 'Install Newman'

  - script: |
      newman run cat-api-performance.json \
        --iteration-count 100 \
        --reporters cli,html \
        --reporter-html-export newman/report_$(System.JobAttempt).html
    displayName: 'Run Performance Test'

  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: 'newman'
      ArtifactName: 'performance-report'
      publishLocation: 'Container'
    displayName: 'Publish Test Report'

# Key Points
strategy.parallel: 4 → Runs the job in parallel to simulate multiple users.
Artifacts → HTML report is published and accessible from Azure DevOps pipeline summary.
Iteration Count → Adjust as per load requirements (100, 500, 1000, etc.), but keep in mind agent limits.

# Best Practices for Load Testing
Use self-hosted agents for higher load, as Microsoft-hosted agents have CPU/memory limits.
Split collection into smaller sets or multiple jobs if simulating thousands of requests.
Monitor API response times and failures via --reporters cli,html,json.

*******************************
