# Fault and Failure in Distributed Systems

## Overview

Distributed systems are inherently prone to various types of faults and failures. Understanding how to design systems that can detect, handle, and recover from failures is crucial for building reliable distributed applications. This section covers fault types, failure detection techniques, recovery strategies, and leader election algorithms for automatic recovery.

## Introduction to Fault vs Failure

Understanding the distinction between faults and failures is fundamental to building resilient systems.

### Fault

A **fault** is a defect or abnormal condition that may cause a component or system to fail to perform its required function.

#### Characteristics of Faults:
- **Latent**: May exist without causing immediate problems
- **Potential**: Can lead to failures under certain conditions
- **Correctable**: May be fixed before causing failures
- **Detectable**: Can often be identified through monitoring

#### Examples of Faults:
```
Hardware Faults:
- Disk bad sectors
- Memory bit flips
- Network packet corruption
- CPU overheating

Software Faults:
- Memory leaks
- Race conditions
- Deadlocks
- Buffer overflows

Human Faults:
- Configuration errors
- Deployment mistakes
- Operational errors
- Design flaws
```

### Failure

A **failure** is the inability of a system or component to perform its required functions within specified performance requirements.

#### Characteristics of Failures:
- **Observable**: Can be detected by users or monitoring systems
- **Impact**: Affects system functionality or performance
- **Immediate**: Has current effect on system behavior
- **Requires Action**: Must be addressed to restore service

#### Examples of Failures:
```
Service Failures:
- Web server returning 500 errors
- Database connection timeouts
- API endpoints becoming unresponsive
- Authentication service down

System Failures:
- Server crashes
- Network partitions
- Data corruption
- Complete service outage
```

### Fault-to-Failure Progression

```
Fault Introduced → Fault Activated → Error State → Failure Observed
       ↑               ↑              ↑            ↑
   (Defect exists) (Conditions met) (Wrong state) (Service impact)
```

#### Example Progression:
```
1. Fault: Memory leak in application code
2. Activation: High traffic triggers excessive memory allocation
3. Error: Out of memory condition
4. Failure: Application crashes, service becomes unavailable
```

## Fault Techniques

Various techniques can be employed to prevent faults from becoming failures.

### Fault Prevention

Preventing faults from being introduced into the system.

#### Design-Time Prevention:
```
Code Reviews:
- Peer review of all code changes
- Automated static analysis
- Security vulnerability scanning
- Performance impact analysis

Testing:
- Unit testing (individual components)
- Integration testing (component interactions)
- System testing (end-to-end scenarios)
- Chaos engineering (fault injection)

Formal Methods:
- Mathematical verification of critical algorithms
- Model checking for concurrent systems
- Specification-based testing
```

#### Operational Prevention:
```
Configuration Management:
- Infrastructure as Code (IaC)
- Automated deployment pipelines
- Configuration validation
- Environment parity (dev/staging/prod)

Change Management:
- Gradual rollouts (canary deployments)
- Feature flags for controlled releases
- Rollback procedures
- Change approval processes
```

### Fault Tolerance

Designing systems to continue operating correctly even when faults occur.

#### Redundancy Techniques:

##### 1. Hardware Redundancy
```
Active-Active Configuration:
Load Balancer → [Server 1] [Server 2] [Server 3]
(All servers handle traffic simultaneously)

Active-Passive Configuration:
[Primary Server] ← Heartbeat → [Standby Server]
(Standby takes over if primary fails)
```

##### 2. Software Redundancy
```
N-Version Programming:
Input → [Version A] → Vote → Output
     → [Version B] → Vote
     → [Version C] → Vote
(Multiple implementations, majority vote wins)

Recovery Blocks:
try {
    primary_algorithm()
} catch (Exception) {
    try {
        backup_algorithm()
    } catch (Exception) {
        fallback_algorithm()
    }
}
```

##### 3. Data Redundancy
```
Replication:
Master Database → [Replica 1] [Replica 2] [Replica 3]

RAID (Redundant Array of Independent Disks):
Data → [Disk 1] [Disk 2] [Disk 3] (with parity)

Erasure Coding:
Original Data → [Chunk 1] [Chunk 2] [Parity 1] [Parity 2]
(Can reconstruct from any 2 chunks)
```

#### Error Detection and Correction:

##### 1. Checksums and Hash Functions
```python
import hashlib

def store_data_with_checksum(data):
    checksum = hashlib.md5(data.encode()).hexdigest()
    return {
        'data': data,
        'checksum': checksum
    }

def verify_data_integrity(stored_data):
    data = stored_data['data']
    expected_checksum = stored_data['checksum']
    actual_checksum = hashlib.md5(data.encode()).hexdigest()
    
    if actual_checksum != expected_checksum:
        raise DataCorruptionError("Data integrity check failed")
    
    return data
```

##### 2. Error Correcting Codes (ECC)
```
Single Error Correction, Double Error Detection (SECDED):
- Can correct single-bit errors
- Can detect double-bit errors
- Used in memory systems and storage

Reed-Solomon Codes:
- Can correct multiple errors
- Used in CDs, DVDs, and distributed storage
- Trade-off between redundancy and correction capability
```

##### 3. Byzantine Fault Tolerance
```
Problem: Some nodes may behave maliciously or unpredictably

Solution: Byzantine Fault Tolerant algorithms
- Require 3f + 1 nodes to tolerate f Byzantine faults
- Use cryptographic signatures for message authentication
- Implement consensus despite malicious behavior

Examples:
- PBFT (Practical Byzantine Fault Tolerance)
- Tendermint
- HotStuff
```

### Fault Isolation

Containing faults to prevent them from spreading throughout the system.

#### Isolation Techniques:

##### 1. Bulkheads
```
Separate Resource Pools:
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Service A   │  │ Service B   │  │ Service C   │
│ Thread Pool │  │ Thread Pool │  │ Thread Pool │
│ (10 threads)│  │ (10 threads)│  │ (10 threads)│
└─────────────┘  └─────────────┘  └─────────────┘

Benefit: Failure in Service A doesn't exhaust threads for B and C
```

##### 2. Circuit Breakers
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = "CLOSED"  # CLOSED, OPEN, HALF_OPEN
        
    def call(self, func, *args, **kwargs):
        if self.state == "OPEN":
            if time.time() - self.last_failure_time > self.timeout:
                self.state = "HALF_OPEN"
            else:
                raise CircuitBreakerOpenError("Circuit breaker is open")
                
        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise
            
    def on_success(self):
        self.failure_count = 0
        self.state = "CLOSED"
        
    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            self.state = "OPEN"
```

##### 3. Timeouts and Retries
```python
import time
import random

def retry_with_exponential_backoff(func, max_retries=3, base_delay=1):
    for attempt in range(max_retries + 1):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries:
                raise e
                
            # Exponential backoff with jitter
            delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)

def call_with_timeout(func, timeout_seconds=30):
    import signal
    
    def timeout_handler(signum, frame):
        raise TimeoutError("Function call timed out")
    
    signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(timeout_seconds)
    
    try:
        result = func()
        signal.alarm(0)  # Cancel the alarm
        return result
    except TimeoutError:
        raise
```

## Failure Techniques

When faults cannot be prevented or tolerated, systems must be designed to fail gracefully and recover quickly.

### Failure Detection

Identifying when failures have occurred so that recovery actions can be initiated.

#### Detection Methods:

##### 1. Heartbeat Monitoring
```python
class HeartbeatMonitor:
    def __init__(self, nodes, heartbeat_interval=5, failure_threshold=3):
        self.nodes = nodes
        self.heartbeat_interval = heartbeat_interval
        self.failure_threshold = failure_threshold
        self.last_heartbeat = {}
        self.failure_count = {}
        
    def start_monitoring(self):
        for node in self.nodes:
            self.last_heartbeat[node] = time.time()
            self.failure_count[node] = 0
            
        while True:
            self.check_heartbeats()
            time.sleep(self.heartbeat_interval)
            
    def check_heartbeats(self):
        current_time = time.time()
        
        for node in self.nodes:
            if current_time - self.last_heartbeat[node] > self.heartbeat_interval:
                self.failure_count[node] += 1
                
                if self.failure_count[node] >= self.failure_threshold:
                    self.handle_node_failure(node)
            else:
                self.failure_count[node] = 0
                
    def receive_heartbeat(self, node):
        self.last_heartbeat[node] = time.time()
        self.failure_count[node] = 0
        
    def handle_node_failure(self, node):
        print(f"Node {node} has failed - initiating recovery")
        self.initiate_recovery(node)
```

##### 2. Health Checks
```python
class HealthChecker:
    def __init__(self, services):
        self.services = services
        self.health_status = {}
        
    def check_service_health(self, service):
        checks = [
            self.check_database_connection(service),
            self.check_external_dependencies(service),
            self.check_resource_usage(service),
            self.check_business_logic(service)
        ]
        
        return all(checks)
        
    def check_database_connection(self, service):
        try:
            service.database.execute("SELECT 1")
            return True
        except Exception:
            return False
            
    def check_external_dependencies(self, service):
        for dependency in service.dependencies:
            try:
                response = requests.get(f"{dependency}/health", timeout=5)
                if response.status_code != 200:
                    return False
            except Exception:
                return False
        return True
        
    def check_resource_usage(self, service):
        cpu_usage = service.get_cpu_usage()
        memory_usage = service.get_memory_usage()
        
        return cpu_usage < 90 and memory_usage < 90
        
    def continuous_health_monitoring(self):
        while True:
            for service in self.services:
                is_healthy = self.check_service_health(service)
                self.health_status[service.name] = is_healthy
                
                if not is_healthy:
                    self.trigger_alert(service)
                    
            time.sleep(30)  # Check every 30 seconds
```

##### 3. Distributed Failure Detection
```python
class PhiAccrualFailureDetector:
    """
    Phi Accrual Failure Detector - used in Cassandra and Akka
    Provides a suspicion level rather than binary alive/dead
    """
    def __init__(self, threshold=8.0, max_sample_size=1000):
        self.threshold = threshold
        self.max_sample_size = max_sample_size
        self.arrival_intervals = {}
        
    def heartbeat(self, node):
        current_time = time.time()
        
        if node not in self.arrival_intervals:
            self.arrival_intervals[node] = []
        else:
            last_time = self.arrival_intervals[node][-1] if self.arrival_intervals[node] else current_time
            interval = current_time - last_time
            
            self.arrival_intervals[node].append(interval)
            
            # Keep only recent samples
            if len(self.arrival_intervals[node]) > self.max_sample_size:
                self.arrival_intervals[node].pop(0)
                
    def phi(self, node):
        if node not in self.arrival_intervals or len(self.arrival_intervals[node]) < 2:
            return 0.0
            
        intervals = self.arrival_intervals[node]
        mean = sum(intervals) / len(intervals)
        variance = sum((x - mean) ** 2 for x in intervals) / len(intervals)
        std_dev = math.sqrt(variance)
        
        current_time = time.time()
        last_heartbeat = intervals[-1] if intervals else current_time
        time_since_last = current_time - last_heartbeat
        
        # Calculate phi value
        phi_value = -math.log10(1 - self.cdf(time_since_last, mean, std_dev))
        return phi_value
        
    def is_available(self, node):
        return self.phi(node) < self.threshold
```

### Graceful Degradation

Designing systems to continue providing reduced functionality when components fail.

#### Degradation Strategies:

##### 1. Feature Toggles
```python
class FeatureToggleManager:
    def __init__(self):
        self.toggles = {}
        self.fallback_behaviors = {}
        
    def register_feature(self, feature_name, fallback_behavior):
        self.toggles[feature_name] = True
        self.fallback_behaviors[feature_name] = fallback_behavior
        
    def disable_feature(self, feature_name):
        self.toggles[feature_name] = False
        
    def is_enabled(self, feature_name):
        return self.toggles.get(feature_name, False)
        
    def execute_with_fallback(self, feature_name, primary_function, *args, **kwargs):
        if self.is_enabled(feature_name):
            try:
                return primary_function(*args, **kwargs)
            except Exception:
                # Disable feature on failure
                self.disable_feature(feature_name)
                
        # Use fallback behavior
        fallback = self.fallback_behaviors.get(feature_name)
        if fallback:
            return fallback(*args, **kwargs)
        else:
            raise FeatureUnavailableError(f"Feature {feature_name} is disabled")

# Usage example
feature_manager = FeatureToggleManager()

def get_recommendations_ml(user_id):
    # Complex ML-based recommendations
    return ml_service.get_recommendations(user_id)

def get_recommendations_simple(user_id):
    # Simple rule-based recommendations
    return simple_recommendation_engine.get_recommendations(user_id)

feature_manager.register_feature("ml_recommendations", get_recommendations_simple)

def get_user_recommendations(user_id):
    return feature_manager.execute_with_fallback(
        "ml_recommendations",
        get_recommendations_ml,
        user_id
    )
```

##### 2. Service Mesh and Load Shedding
```python
class LoadShedder:
    def __init__(self, max_queue_size=1000, shed_probability=0.1):
        self.max_queue_size = max_queue_size
        self.shed_probability = shed_probability
        self.current_queue_size = 0
        self.total_requests = 0
        self.shed_requests = 0
        
    def should_shed_request(self, request):
        self.total_requests += 1
        
        # Shed based on queue size
        if self.current_queue_size > self.max_queue_size:
            return True
            
        # Shed based on request priority
        if hasattr(request, 'priority') and request.priority < 5:
            if random.random() < self.shed_probability:
                return True
                
        # Shed based on system load
        cpu_usage = self.get_cpu_usage()
        if cpu_usage > 80:
            shed_rate = (cpu_usage - 80) / 20  # Linear increase from 80% to 100%
            if random.random() < shed_rate:
                return True
                
        return False
        
    def process_request(self, request, handler):
        if self.should_shed_request(request):
            self.shed_requests += 1
            raise ServiceOverloadedError("Request shed due to high load")
            
        self.current_queue_size += 1
        try:
            return handler(request)
        finally:
            self.current_queue_size -= 1
```

### Recovery Strategies

Techniques for restoring system functionality after failures.

#### Recovery Approaches:

##### 1. Restart and Retry
```python
class ServiceRecoveryManager:
    def __init__(self, service):
        self.service = service
        self.restart_count = 0
        self.max_restarts = 3
        self.restart_window = 300  # 5 minutes
        self.restart_times = []
        
    def attempt_recovery(self):
        current_time = time.time()
        
        # Clean old restart times
        self.restart_times = [t for t in self.restart_times 
                             if current_time - t < self.restart_window]
        
        if len(self.restart_times) >= self.max_restarts:
            raise MaxRestartsExceededError("Too many restarts in time window")
            
        try:
            self.service.stop()
            time.sleep(5)  # Grace period
            self.service.start()
            
            self.restart_times.append(current_time)
            return True
            
        except Exception as e:
            raise RecoveryFailedError(f"Failed to restart service: {e}")
```

##### 2. State Recovery
```python
class CheckpointManager:
    def __init__(self, checkpoint_interval=60):
        self.checkpoint_interval = checkpoint_interval
        self.last_checkpoint = time.time()
        
    def create_checkpoint(self, state):
        checkpoint_data = {
            'timestamp': time.time(),
            'state': state,
            'version': self.get_state_version(state)
        }
        
        # Persist checkpoint to durable storage
        self.save_checkpoint(checkpoint_data)
        self.last_checkpoint = time.time()
        
    def should_checkpoint(self):
        return time.time() - self.last_checkpoint > self.checkpoint_interval
        
    def recover_from_checkpoint(self):
        latest_checkpoint = self.load_latest_checkpoint()
        if latest_checkpoint:
            return latest_checkpoint['state']
        else:
            return self.get_initial_state()
            
    def save_checkpoint(self, checkpoint_data):
        # Implementation depends on storage system
        # Could be database, file system, or distributed storage
        with open(f"checkpoint_{checkpoint_data['timestamp']}.json", 'w') as f:
            json.dump(checkpoint_data, f)
```

##### 3. Rollback and Compensation
```python
class CompensationManager:
    def __init__(self):
        self.compensation_actions = []
        
    def register_compensation(self, action, compensation):
        self.compensation_actions.append((action, compensation))
        
    def execute_with_compensation(self, action, compensation):
        try:
            result = action()
            self.register_compensation(action, compensation)
            return result
        except Exception:
            # If action fails, no compensation needed
            raise
            
    def compensate_all(self):
        # Execute compensations in reverse order
        for action, compensation in reversed(self.compensation_actions):
            try:
                compensation()
            except Exception as e:
                # Log compensation failure but continue
                print(f"Compensation failed: {e}")
                
        self.compensation_actions.clear()

# Usage example
def transfer_money(from_account, to_account, amount):
    compensation_manager = CompensationManager()
    
    try:
        # Debit from source account
        compensation_manager.execute_with_compensation(
            lambda: debit_account(from_account, amount),
            lambda: credit_account(from_account, amount)
        )
        
        # Credit to destination account
        compensation_manager.execute_with_compensation(
            lambda: credit_account(to_account, amount),
            lambda: debit_account(to_account, amount)
        )
        
        return "Transfer successful"
        
    except Exception as e:
        compensation_manager.compensate_all()
        raise TransferFailedError(f"Transfer failed: {e}")
```

## Leader Election for Auto Recovery

Leader election is a fundamental problem in distributed systems where multiple nodes need to coordinate and one node needs to be designated as the leader.

### Introduction and Use Cases

Leader election ensures that exactly one node in a distributed system acts as the coordinator or decision-maker at any given time.

#### Use Cases:

##### 1. Database Master Selection
```
Multiple database replicas → Elect one as master for writes
Benefits:
- Consistent write ordering
- Avoid split-brain scenarios
- Automatic failover
```

##### 2. Distributed Lock Management
```
Multiple nodes need exclusive access → Elect leader to manage locks
Benefits:
- Centralized lock coordination
- Deadlock detection and resolution
- Fair lock allocation
```

##### 3. Task Coordination
```
Distributed workers → Elect leader to assign tasks
Benefits:
- Avoid duplicate work
- Load balancing
- Progress monitoring
```

##### 4. Configuration Management
```
Multiple service instances → Elect leader to manage configuration
Benefits:
- Consistent configuration updates
- Centralized decision making
- Conflict resolution
```

### Bully Algorithm

The Bully Algorithm is a simple leader election algorithm where the node with the highest ID becomes the leader.

#### Algorithm Steps:

##### 1. Election Initiation
```
When a node detects leader failure or starts up:
1. Send ELECTION message to all nodes with higher IDs
2. If no response within timeout, declare self as leader
3. If any response received, wait for COORDINATOR message
```

##### 2. Election Response
```
When receiving ELECTION message:
1. Send OK response to sender
2. Start own election process (if not already started)
```

##### 3. Leader Announcement
```
When becoming leader:
1. Send COORDINATOR message to all nodes with lower IDs
2. Start acting as leader
```

#### Implementation:
```python
class BullyElection:
    def __init__(self, node_id, all_nodes):
        self.node_id = node_id
        self.all_nodes = all_nodes
        self.leader_id = None
        self.election_in_progress = False
        self.election_timeout = 5  # seconds
        
    def start_election(self):
        if self.election_in_progress:
            return
            
        self.election_in_progress = True
        higher_nodes = [n for n in self.all_nodes if n > self.node_id]
        
        if not higher_nodes:
            # No higher nodes, become leader
            self.become_leader()
            return
            
        # Send ELECTION to higher nodes
        responses = []
        for node in higher_nodes:
            try:
                response = self.send_election_message(node)
                if response == "OK":
                    responses.append(node)
            except NodeUnavailableError:
                continue
                
        if not responses:
            # No responses from higher nodes, become leader
            self.become_leader()
        else:
            # Wait for COORDINATOR message
            self.wait_for_coordinator()
            
    def handle_election_message(self, sender_id):
        if sender_id < self.node_id:
            # Send OK response
            self.send_ok_response(sender_id)
            # Start own election if not already in progress
            if not self.election_in_progress:
                self.start_election()
        return "OK"
        
    def become_leader(self):
        self.leader_id = self.node_id
        self.election_in_progress = False
        
        # Announce leadership to lower nodes
        lower_nodes = [n for n in self.all_nodes if n < self.node_id]
        for node in lower_nodes:
            try:
                self.send_coordinator_message(node)
            except NodeUnavailableError:
                continue
                
        print(f"Node {self.node_id} is now the leader")
        
    def handle_coordinator_message(self, leader_id):
        self.leader_id = leader_id
        self.election_in_progress = False
        print(f"Node {leader_id} is the new leader")
        
    def wait_for_coordinator(self):
        # Wait for COORDINATOR message with timeout
        start_time = time.time()
        while time.time() - start_time < self.election_timeout:
            if not self.election_in_progress:
                return  # Received COORDINATOR message
            time.sleep(0.1)
            
        # Timeout - start new election
        self.start_election()
```

#### Bully Algorithm Characteristics:

**Advantages:**
✅ **Simple**: Easy to understand and implement
✅ **Deterministic**: Always elects highest ID node
✅ **Self-stabilizing**: Eventually converges to correct leader

**Disadvantages:**
❌ **Network Overhead**: Many messages during election
❌ **Unfair**: Always favors highest ID node
❌ **Cascading Elections**: Node failures can trigger multiple elections

### Raft Algorithm

Raft is a consensus algorithm designed to be more understandable than Paxos while providing the same guarantees.

#### Raft Concepts:

##### 1. Node States
```
Follower: Default state, responds to leaders and candidates
Candidate: Seeks votes to become leader
Leader: Handles client requests and manages log replication
```

##### 2. Terms
```
Term: Logical clock that increases monotonically
- Each term has at most one leader
- Terms help detect stale information
- Nodes reject messages from earlier terms
```

##### 3. Log Replication
```
Leader receives client request → Appends to local log → 
Replicates to followers → Commits when majority acknowledges
```

#### Raft Leader Election:

##### Election Process:
```python
class RaftNode:
    def __init__(self, node_id, cluster_nodes):
        self.node_id = node_id
        self.cluster_nodes = cluster_nodes
        self.state = "FOLLOWER"
        self.current_term = 0
        self.voted_for = None
        self.leader_id = None
        self.election_timeout = random.uniform(150, 300)  # milliseconds
        self.last_heartbeat = time.time()
        
    def start_election(self):
        self.state = "CANDIDATE"
        self.current_term += 1
        self.voted_for = self.node_id
        self.reset_election_timeout()
        
        votes_received = 1  # Vote for self
        
        # Request votes from other nodes
        for node in self.cluster_nodes:
            if node != self.node_id:
                try:
                    response = self.request_vote(node)
                    if response.get("vote_granted"):
                        votes_received += 1
                except Exception:
                    continue
                    
        # Check if won election
        if votes_received > len(self.cluster_nodes) // 2:
            self.become_leader()
        else:
            self.become_follower()
            
    def request_vote(self, node):
        request = {
            "term": self.current_term,
            "candidate_id": self.node_id,
            "last_log_index": self.get_last_log_index(),
            "last_log_term": self.get_last_log_term()
        }
        return self.send_message(node, "request_vote", request)
        
    def handle_request_vote(self, request):
        term = request["term"]
        candidate_id = request["candidate_id"]
        
        # Update term if higher
        if term > self.current_term:
            self.current_term = term
            self.voted_for = None
            self.become_follower()
            
        vote_granted = False
        
        # Grant vote if:
        # 1. Haven't voted in this term, or already voted for this candidate
        # 2. Candidate's log is at least as up-to-date as ours
        if (term == self.current_term and 
            (self.voted_for is None or self.voted_for == candidate_id) and
            self.is_log_up_to_date(request)):
            
            vote_granted = True
            self.voted_for = candidate_id
            self.reset_election_timeout()
            
        return {
            "term": self.current_term,
            "vote_granted": vote_granted
        }
        
    def become_leader(self):
        self.state = "LEADER"
        self.leader_id = self.node_id
        print(f"Node {self.node_id} became leader for term {self.current_term}")
        
        # Send heartbeats to maintain leadership
        self.start_heartbeat_timer()
        
    def become_follower(self):
        self.state = "FOLLOWER"
        self.leader_id = None
        
    def send_heartbeat(self):
        if self.state == "LEADER":
            for node in self.cluster_nodes:
                if node != self.node_id:
                    try:
                        self.send_append_entries(node, heartbeat=True)
                    except Exception:
                        continue
                        
    def handle_append_entries(self, request):
        term = request["term"]
        leader_id = request["leader_id"]
        
        # Update term if higher
        if term > self.current_term:
            self.current_term = term
            self.voted_for = None
            
        # Accept leadership if term is current
        if term == self.current_term:
            self.become_follower()
            self.leader_id = leader_id
            self.last_heartbeat = time.time()
            
        return {
            "term": self.current_term,
            "success": term >= self.current_term
        }
        
    def check_election_timeout(self):
        if (self.state in ["FOLLOWER", "CANDIDATE"] and 
            time.time() - self.last_heartbeat > self.election_timeout / 1000):
            self.start_election()
```

#### Raft Advantages:

✅ **Strong Consistency**: Provides linearizable consistency
✅ **Fault Tolerance**: Tolerates (n-1)/2 failures
✅ **Understandable**: Easier to understand than Paxos
✅ **Proven**: Well-tested in production systems

#### Raft Disadvantages:

❌ **Network Overhead**: Requires majority for all operations
❌ **Latency**: Multiple round trips for consensus
❌ **Complexity**: More complex than simple algorithms

### Comparison: Bully vs Raft

| Aspect | Bully Algorithm | Raft Algorithm |
|--------|----------------|----------------|
| **Complexity** | Simple | Moderate |
| **Fault Tolerance** | Basic | Strong |
| **Consistency** | Eventual | Strong |
| **Network Messages** | High during election | Moderate |
| **Use Cases** | Simple coordination | Critical consensus |
| **Recovery Time** | Fast | Moderate |

## Conclusion

Fault and failure handling in distributed systems requires a comprehensive approach:

1. **Distinguish Faults from Failures**: Understand the progression from defects to service impact
2. **Implement Multiple Techniques**: Use prevention, tolerance, and recovery strategies
3. **Design for Graceful Degradation**: Maintain partial functionality during failures
4. **Plan Recovery Strategies**: Have automated recovery mechanisms
5. **Choose Appropriate Leader Election**: Select algorithms based on consistency requirements

The key is to build systems that expect failures and handle them gracefully, rather than trying to prevent all failures.

## Next Steps

In the following sections, we'll explore:
- **Distributed Consensus**: Advanced consensus algorithms and their applications
- **Asynchronous Programming**: Building resilient event-driven systems
- **Monitoring**: Observability for fault detection and recovery