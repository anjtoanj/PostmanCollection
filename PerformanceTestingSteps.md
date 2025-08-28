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
