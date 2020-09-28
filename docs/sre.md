# System Reliability Engineering

Some potential failures to consider for each deployment:

## Availability

* Always ask what are the SLAs
* Queue full and transaction not being processed
* API backend not available
* Network failure
* DNS problems
* Update for new version fails to deploy
* Hitting unknown limits, like number of VPC interfaces
* Certificate not auto renewed on expiry, and authentication fails

## Performance

* Cold start of the function cause deploy
* Unreliable response time
* Consumer lag behind
* Event backbone not responding 
* Functions lock an external resource which becomes a bottleneck
* Backend DB starts to throttle at high load
* Serverless stars to throttle at high volume
* Front end times out, no response from the back end
* Latency measurement
* Meaningful throughput measurement
* Batch job impacting main real time processing
* No more in SLA
* Retry logic missing jitter causes bursts of retries
* Message Queues are filled too quickly, causing unhandled messages
* Network latency impacting call / response - and front end doesn't handle
* Can't scale up fast enough to support demand spikes
* Transaction chain scales at different rates or hard limits on one link break scalability

## Data Loss / Durability

* Data at rest gets corrupted
* Error while archiving central logs
* Every nth transaction fails silently, loses data
* DR site down
* Data out of synch between two data sources
* Can't find a particular orderID in the logs
* Lack of log files in serverless world
* Data replicated among active cluster is taking longer than expected
* Queue exceeds visibility limit
* Losing messages in kafka
* Duplicated messages

## Quality and correctness

* Idempotency failure causing multiple transactions in error
* Dual records created in database
* See unexpected records
* Version error after code upgrade
* Data recovery can cause ID duplicates
* Error reporting not reporting actual problem
* Are the runbooks autodated?
* Does simulated transaction bring clear understanding of the process flow and error flow?
* Inconsistent view between caller and callee? 

## Security

* Privacy laws enforcement
* GDPR
* How to identify data breach
* Data at rest not encrypted and protected
* Elevation of privileges on PROD accounts
* Access to private data
* Cannot trace who make changes
* Keep up to date with security patches
* Open source component with security vulnerability goes undetected
