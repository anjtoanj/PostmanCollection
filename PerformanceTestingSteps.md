1. Create postman collection
2. Declare environment and collection variables. Here am only using collection variables
3. Export the collection as json file
cat-api-performance.json
5. Open GitBash and navigate to the location
6. Install node
 node -v
 npm -v
 newman install
7. run the collection in newman
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


