# Distributed Consensus

## Overview

Distributed consensus is the fundamental problem of getting multiple nodes in a distributed system to agree on a single value or decision, even in the presence of failures. It's the foundation for building reliable distributed systems, enabling coordination, consistency, and fault tolerance. This section explores consensus mechanisms, termination conditions, practical considerations, and real-world implementations.

## Consensus Introduction and Mechanisms

Distributed consensus addresses the challenge of achieving agreement among distributed processes, some of which may fail or behave unpredictably.

### The Consensus Problem

#### Definition:
In a distributed system with n processes, consensus requires that:
1. **Agreement**: All non-faulty processes decide on the same value
2. **Validity**: The decided value must be proposed by some process
3. **Termination**: All non-faulty processes eventually decide

#### Why Consensus is Hard:
```
Challenges:
- Network delays and partitions
- Process failures (crash, Byzantine)
- Asynchronous communication
- No global clock or shared memory
- Impossibility results (FLP theorem)
```

#### FLP Impossibility Theorem:
```
In an asynchronous network where processes can fail by crashing,
there is no deterministic algorithm that can guarantee consensus
in all possible executions.

Implication: Perfect consensus is impossible in asynchronous systems
Solution: Use timeouts, randomization, or partial synchrony assumptions
```

### Types of Consensus Problems

#### 1. Binary Consensus
```
Problem: Agree on a single bit (0 or 1)
Example: Commit/abort decision in distributed transactions
```

#### 2. Multi-Value Consensus
```
Problem: Agree on a value from a larger domain
Example: Leader election, configuration updates
```

#### 3. Byzantine Consensus
```
Problem: Achieve consensus despite malicious or arbitrary failures
Requirement: At least 3f + 1 nodes to tolerate f Byzantine failures
```

#### 4. Atomic Broadcast
```
Problem: Deliver messages to all processes in the same order
Relationship: Equivalent to consensus (can solve one using the other)
```

### Consensus in Different Failure Models

#### 1. Crash Failure Model
```
Assumptions:
- Processes fail by stopping (crash-stop)
- Failed processes don't recover
- Network is reliable but asynchronous

Algorithms:
- Paxos
- Raft
- PBFT (for crash failures)
```

#### 2. Byzantine Failure Model
```
Assumptions:
- Processes can fail arbitrarily (malicious behavior)
- May send conflicting messages
- Network may be unreliable

Algorithms:
- PBFT (Practical Byzantine Fault Tolerance)
- Tendermint
- HotStuff
```

#### 3. Omission Failure Model
```
Assumptions:
- Processes may fail to send or receive messages
- No malicious behavior
- Between crash and Byzantine models

Algorithms:
- Modified Paxos variants
- Gossip-based protocols
```

## Termination Conditions

Understanding when and how consensus algorithms terminate is crucial for their practical application.

### Termination Guarantees

#### 1. Deterministic Termination
```
Property: Algorithm guarantees termination in finite time
Limitation: Impossible in pure asynchronous systems (FLP theorem)
Solution: Add synchrony assumptions or failure detectors
```

#### 2. Probabilistic Termination
```
Property: Algorithm terminates with probability 1
Approach: Use randomization to break symmetry
Example: Randomized consensus algorithms
```

#### 3. Conditional Termination
```
Property: Termination guaranteed under certain conditions
Conditions:
- Network synchrony periods
- Failure detector accuracy
- Bounded message delays
```

### Synchrony Models

#### 1. Synchronous Model
```
Assumptions:
- Bounded message delays
- Bounded processing time
- Synchronized clocks

Termination: Guaranteed in finite time
Reality: Rarely exists in practice
```

#### 2. Asynchronous Model
```
Assumptions:
- Unbounded message delays
- Unbounded processing time
- No synchronized clocks

Termination: Impossible to guarantee (FLP theorem)
Reality: Most real networks are asynchronous
```

#### 3. Partially Synchronous Model
```
Assumptions:
- Eventually synchronous (bounds exist but unknown)
- Periods of synchrony and asynchrony
- Realistic failure detectors

Termination: Guaranteed eventually
Reality: Most practical systems
```

### Failure Detectors

Failure detectors provide information about process failures to help achieve consensus.

#### Types of Failure Detectors:

##### 1. Perfect Failure Detector (P)
```
Properties:
- Strong Completeness: Eventually detects all failures
- Strong Accuracy: Never suspects correct processes

Implementation: Impossible in asynchronous systems
Use: Theoretical baseline
```

##### 2. Eventually Perfect Failure Detector (◊P)
```
Properties:
- Strong Completeness: Eventually detects all failures
- Eventual Strong Accuracy: Eventually stops suspecting correct processes

Implementation: Possible with timeouts and heartbeats
Use: Practical consensus algorithms
```

##### 3. Weak Failure Detector (W)
```
Properties:
- Weak Completeness: Some correct process detects all failures
- Weak Accuracy: Some correct process is never suspected

Use: Sufficient for consensus in asynchronous systems
```

#### Implementation Example:
```python
class EventuallyPerfectFailureDetector:
    def __init__(self, nodes, initial_timeout=1000):
        self.nodes = nodes
        self.suspected = set()
        self.timeout = initial_timeout
        self.last_heartbeat = {}
        self.heartbeat_history = {}
        
    def start_monitoring(self):
        for node in self.nodes:
            self.last_heartbeat[node] = time.time()
            self.heartbeat_history[node] = []
            
        while True:
            self.check_timeouts()
            self.adapt_timeout()
            time.sleep(0.1)
            
    def receive_heartbeat(self, node):
        current_time = time.time()
        
        # Update heartbeat time
        if node in self.last_heartbeat:
            interval = current_time - self.last_heartbeat[node]
            self.heartbeat_history[node].append(interval)
            
            # Keep only recent history
            if len(self.heartbeat_history[node]) > 100:
                self.heartbeat_history[node].pop(0)
                
        self.last_heartbeat[node] = current_time
        
        # Remove from suspected if present
        if node in self.suspected:
            self.suspected.remove(node)
            
    def check_timeouts(self):
        current_time = time.time()
        
        for node in self.nodes:
            if node in self.last_heartbeat:
                time_since_heartbeat = current_time - self.last_heartbeat[node]
                
                if time_since_heartbeat > self.timeout / 1000:
                    if node not in self.suspected:
                        self.suspected.add(node)
                        self.on_suspect(node)
                        
    def adapt_timeout(self):
        # Adapt timeout based on network conditions
        all_intervals = []
        for node_intervals in self.heartbeat_history.values():
            all_intervals.extend(node_intervals)
            
        if all_intervals:
            mean_interval = sum(all_intervals) / len(all_intervals)
            # Set timeout to 4x mean interval (heuristic)
            self.timeout = max(1000, mean_interval * 4 * 1000)
            
    def is_suspected(self, node):
        return node in self.suspected
        
    def on_suspect(self, node):
        print(f"Node {node} is suspected to have failed")
```

## Practical Considerations

Real-world consensus implementations must address various practical challenges beyond theoretical guarantees.

### Performance Considerations

#### 1. Latency Optimization
```python
class OptimizedConsensus:
    def __init__(self):
        self.fast_path_enabled = True
        self.batching_enabled = True
        self.pipeline_depth = 10
        
    def propose_value(self, value):
        if self.fast_path_enabled and self.can_use_fast_path():
            # Fast path: Skip prepare phase if conditions are met
            return self.fast_path_consensus(value)
        else:
            # Normal path: Full consensus protocol
            return self.normal_path_consensus(value)
            
    def can_use_fast_path(self):
        # Fast path conditions:
        # - No recent failures
        # - Network is stable
        # - Previous consensus completed successfully
        return (not self.recent_failures() and 
                self.network_stable() and 
                self.last_consensus_successful())
                
    def batch_proposals(self, proposals):
        # Batch multiple proposals for efficiency
        if len(proposals) >= self.batch_size or self.batch_timeout_reached():
            return self.consensus_on_batch(proposals)
        else:
            self.add_to_pending_batch(proposals)
```

#### 2. Throughput Optimization
```python
class PipelinedConsensus:
    def __init__(self, pipeline_depth=10):
        self.pipeline_depth = pipeline_depth
        self.active_instances = {}
        self.next_instance_id = 0
        
    def start_consensus_instance(self, value):
        if len(self.active_instances) < self.pipeline_depth:
            instance_id = self.next_instance_id
            self.next_instance_id += 1
            
            instance = ConsensusInstance(instance_id, value)
            self.active_instances[instance_id] = instance
            
            # Start consensus in background
            threading.Thread(
                target=self.run_consensus_instance,
                args=(instance,)
            ).start()
            
            return instance_id
        else:
            raise PipelineFullError("Consensus pipeline is full")
            
    def run_consensus_instance(self, instance):
        try:
            result = self.execute_consensus_protocol(instance)
            self.on_consensus_complete(instance.id, result)
        finally:
            del self.active_instances[instance.id]
```

### Network Partitions

Handling network partitions is crucial for practical consensus systems.

#### Partition Tolerance Strategies:

##### 1. Majority Quorum
```python
class QuorumBasedConsensus:
    def __init__(self, nodes):
        self.nodes = nodes
        self.quorum_size = len(nodes) // 2 + 1
        
    def can_make_progress(self, available_nodes):
        return len(available_nodes) >= self.quorum_size
        
    def handle_partition(self, partition1, partition2):
        # Only the majority partition can make progress
        if len(partition1) >= self.quorum_size:
            self.continue_in_partition(partition1)
            self.pause_in_partition(partition2)
        elif len(partition2) >= self.quorum_size:
            self.continue_in_partition(partition2)
            self.pause_in_partition(partition1)
        else:
            # No majority - system becomes unavailable
            self.pause_all_operations()
```

##### 2. Partition Recovery
```python
class PartitionRecoveryManager:
    def __init__(self):
        self.partition_log = []
        self.recovery_in_progress = False
        
    def detect_partition_healing(self):
        # Detect when partitioned nodes reconnect
        if self.all_nodes_reachable() and not self.recovery_in_progress:
            self.start_partition_recovery()
            
    def start_partition_recovery(self):
        self.recovery_in_progress = True
        
        try:
            # 1. Exchange state information
            states = self.collect_states_from_all_nodes()
            
            # 2. Identify conflicts and missing decisions
            conflicts = self.identify_conflicts(states)
            
            # 3. Resolve conflicts using deterministic rules
            for conflict in conflicts:
                self.resolve_conflict(conflict)
                
            # 4. Synchronize all nodes to consistent state
            self.synchronize_all_nodes()
            
        finally:
            self.recovery_in_progress = False
```

### Byzantine Fault Tolerance

Handling malicious or arbitrary failures requires special considerations.

#### Byzantine Consensus Requirements:
```
Network Requirements:
- At least 3f + 1 nodes to tolerate f Byzantine failures
- Authenticated communication (digital signatures)
- Bounded message delays (for liveness)

Algorithm Properties:
- Safety: Never decide on different values
- Liveness: Eventually decide (under synchrony assumptions)
- Validity: Decided value must be proposed by correct node
```

#### PBFT Implementation Outline:
```python
class PBFTNode:
    def __init__(self, node_id, total_nodes):
        self.node_id = node_id
        self.total_nodes = total_nodes
        self.f = (total_nodes - 1) // 3  # Max Byzantine failures
        self.view = 0
        self.sequence_number = 0
        self.state = "NORMAL"
        
        # Message logs
        self.pre_prepare_log = {}
        self.prepare_log = {}
        self.commit_log = {}
        
    def client_request(self, request):
        if self.is_primary():
            self.start_consensus(request)
        else:
            self.forward_to_primary(request)
            
    def start_consensus(self, request):
        # Phase 1: Pre-prepare
        self.sequence_number += 1
        pre_prepare_msg = {
            'view': self.view,
            'sequence': self.sequence_number,
            'digest': self.hash(request),
            'request': request
        }
        
        self.broadcast_pre_prepare(pre_prepare_msg)
        self.pre_prepare_log[self.sequence_number] = pre_prepare_msg
        
    def handle_pre_prepare(self, msg):
        if self.validate_pre_prepare(msg):
            # Phase 2: Prepare
            prepare_msg = {
                'view': msg['view'],
                'sequence': msg['sequence'],
                'digest': msg['digest'],
                'node_id': self.node_id
            }
            
            self.broadcast_prepare(prepare_msg)
            self.add_to_prepare_log(prepare_msg)
            
    def handle_prepare(self, msg):
        self.add_to_prepare_log(msg)
        
        # Check if we have 2f prepare messages
        if self.count_prepare_messages(msg['sequence']) >= 2 * self.f:
            # Phase 3: Commit
            commit_msg = {
                'view': msg['view'],
                'sequence': msg['sequence'],
                'digest': msg['digest'],
                'node_id': self.node_id
            }
            
            self.broadcast_commit(commit_msg)
            self.add_to_commit_log(commit_msg)
            
    def handle_commit(self, msg):
        self.add_to_commit_log(msg)
        
        # Check if we have 2f + 1 commit messages
        if self.count_commit_messages(msg['sequence']) >= 2 * self.f + 1:
            # Execute the request
            self.execute_request(msg['sequence'])
```

### Optimizations and Variants

#### 1. Fast Byzantine Paxos
```
Optimization: Reduce message complexity in common case
Normal case: 2 message delays instead of 3
Fallback: Use full protocol when fast path fails
```

#### 2. HotStuff
```
Improvements over PBFT:
- Linear message complexity
- Simpler view change protocol
- Better liveness properties
- Used in blockchain systems
```

#### 3. Tendermint
```
Features:
- Immediate finality
- Fork accountability
- Application-agnostic
- Used in Cosmos blockchain
```

## Examples: Paxos, Raft, Zookeeper Zab Protocol

### Paxos

Paxos is the foundational consensus algorithm that inspired many modern distributed systems.

#### Basic Paxos Algorithm:

##### Phase 1: Prepare
```python
class PaxosProposer:
    def __init__(self, node_id, acceptors):
        self.node_id = node_id
        self.acceptors = acceptors
        self.proposal_number = 0
        
    def propose(self, value):
        self.proposal_number += 1
        proposal_id = (self.proposal_number, self.node_id)
        
        # Phase 1a: Send prepare requests
        prepare_responses = []
        for acceptor in self.acceptors:
            try:
                response = acceptor.prepare(proposal_id)
                prepare_responses.append(response)
            except Exception:
                continue
                
        # Check if majority responded
        if len(prepare_responses) > len(self.acceptors) // 2:
            # Phase 2a: Send accept requests
            chosen_value = self.choose_value(prepare_responses, value)
            return self.send_accept_requests(proposal_id, chosen_value)
        else:
            raise ConsensusFailedError("Failed to get majority in prepare phase")
            
    def choose_value(self, responses, proposed_value):
        # Choose value from highest-numbered accepted proposal
        # or use proposed value if no previous proposals
        highest_proposal = None
        chosen_value = proposed_value
        
        for response in responses:
            if response.get('accepted_proposal'):
                proposal = response['accepted_proposal']
                if (highest_proposal is None or 
                    proposal['id'] > highest_proposal['id']):
                    highest_proposal = proposal
                    chosen_value = proposal['value']
                    
        return chosen_value

class PaxosAcceptor:
    def __init__(self, node_id):
        self.node_id = node_id
        self.promised_proposal = None
        self.accepted_proposal = None
        
    def prepare(self, proposal_id):
        if (self.promised_proposal is None or 
            proposal_id > self.promised_proposal):
            
            self.promised_proposal = proposal_id
            
            return {
                'promise': True,
                'accepted_proposal': self.accepted_proposal
            }
        else:
            return {'promise': False}
            
    def accept(self, proposal_id, value):
        if (self.promised_proposal is None or 
            proposal_id >= self.promised_proposal):
            
            self.promised_proposal = proposal_id
            self.accepted_proposal = {
                'id': proposal_id,
                'value': value
            }
            
            return {'accepted': True}
        else:
            return {'accepted': False}
```

#### Multi-Paxos:
```python
class MultiPaxos:
    def __init__(self, node_id, nodes):
        self.node_id = node_id
        self.nodes = nodes
        self.leader = None
        self.log = {}  # slot -> value
        self.next_slot = 1
        
    def propose_value(self, value):
        if self.is_leader():
            return self.leader_propose(value)
        else:
            return self.forward_to_leader(value)
            
    def leader_propose(self, value):
        slot = self.next_slot
        self.next_slot += 1
        
        # Skip prepare phase if we're the established leader
        if self.is_established_leader():
            return self.skip_to_accept_phase(slot, value)
        else:
            return self.full_paxos_round(slot, value)
```

### Raft (Detailed Implementation)

Raft provides a more understandable alternative to Paxos with similar guarantees.

#### Complete Raft Implementation:
```python
class RaftNode:
    def __init__(self, node_id, cluster):
        # Persistent state
        self.node_id = node_id
        self.current_term = 0
        self.voted_for = None
        self.log = []  # List of log entries
        
        # Volatile state
        self.commit_index = 0
        self.last_applied = 0
        self.state = "FOLLOWER"
        self.leader_id = None
        
        # Leader state (reinitialized after election)
        self.next_index = {}  # For each server, index of next log entry to send
        self.match_index = {}  # For each server, index of highest log entry known to be replicated
        
        # Cluster configuration
        self.cluster = cluster
        self.election_timeout = random.uniform(150, 300)
        self.heartbeat_interval = 50
        
    def start(self):
        self.reset_election_timer()
        self.start_main_loop()
        
    def start_main_loop(self):
        while True:
            if self.state == "LEADER":
                self.send_heartbeats()
                time.sleep(self.heartbeat_interval / 1000)
            else:
                if self.election_timeout_expired():
                    self.start_election()
                time.sleep(10)  # Check every 10ms
                
    def start_election(self):
        self.state = "CANDIDATE"
        self.current_term += 1
        self.voted_for = self.node_id
        self.reset_election_timer()
        
        votes_received = 1  # Vote for self
        
        # Send RequestVote RPCs to all other servers
        for node in self.cluster:
            if node != self.node_id:
                try:
                    response = self.send_request_vote(node)
                    if response.get('vote_granted'):
                        votes_received += 1
                except Exception:
                    continue
                    
        # Check if won election
        if votes_received > len(self.cluster) // 2:
            self.become_leader()
        else:
            self.become_follower()
            
    def send_request_vote(self, node):
        last_log_index = len(self.log) - 1
        last_log_term = self.log[last_log_index]['term'] if self.log else 0
        
        request = {
            'term': self.current_term,
            'candidate_id': self.node_id,
            'last_log_index': last_log_index,
            'last_log_term': last_log_term
        }
        
        return self.send_rpc(node, 'request_vote', request)
        
    def handle_request_vote(self, request):
        term = request['term']
        candidate_id = request['candidate_id']
        last_log_index = request['last_log_index']
        last_log_term = request['last_log_term']
        
        # Update term if higher
        if term > self.current_term:
            self.current_term = term
            self.voted_for = None
            self.become_follower()
            
        vote_granted = False
        
        if (term == self.current_term and
            (self.voted_for is None or self.voted_for == candidate_id) and
            self.is_log_up_to_date(last_log_index, last_log_term)):
            
            vote_granted = True
            self.voted_for = candidate_id
            self.reset_election_timer()
            
        return {
            'term': self.current_term,
            'vote_granted': vote_granted
        }
        
    def become_leader(self):
        self.state = "LEADER"
        self.leader_id = self.node_id
        
        # Initialize leader state
        for node in self.cluster:
            if node != self.node_id:
                self.next_index[node] = len(self.log)
                self.match_index[node] = 0
                
        # Send initial heartbeats
        self.send_heartbeats()
        
    def send_heartbeats(self):
        for node in self.cluster:
            if node != self.node_id:
                self.send_append_entries(node, heartbeat=True)
                
    def send_append_entries(self, node, heartbeat=False):
        prev_log_index = self.next_index[node] - 1
        prev_log_term = self.log[prev_log_index]['term'] if prev_log_index >= 0 else 0
        
        entries = []
        if not heartbeat and self.next_index[node] < len(self.log):
            entries = self.log[self.next_index[node]:]
            
        request = {
            'term': self.current_term,
            'leader_id': self.node_id,
            'prev_log_index': prev_log_index,
            'prev_log_term': prev_log_term,
            'entries': entries,
            'leader_commit': self.commit_index
        }
        
        try:
            response = self.send_rpc(node, 'append_entries', request)
            self.handle_append_entries_response(node, request, response)
        except Exception:
            pass  # Handle network failures
            
    def handle_append_entries_response(self, node, request, response):
        if response['term'] > self.current_term:
            self.current_term = response['term']
            self.become_follower()
            return
            
        if self.state == "LEADER" and response['term'] == self.current_term:
            if response['success']:
                # Update next_index and match_index
                self.match_index[node] = request['prev_log_index'] + len(request['entries'])
                self.next_index[node] = self.match_index[node] + 1
                
                # Update commit_index if majority has replicated
                self.update_commit_index()
            else:
                # Decrement next_index and retry
                self.next_index[node] = max(0, self.next_index[node] - 1)
                self.send_append_entries(node)
                
    def update_commit_index(self):
        # Find highest index replicated on majority of servers
        for index in range(self.commit_index + 1, len(self.log)):
            count = 1  # Count self
            for node in self.cluster:
                if node != self.node_id and self.match_index.get(node, 0) >= index:
                    count += 1
                    
            if count > len(self.cluster) // 2 and self.log[index]['term'] == self.current_term:
                self.commit_index = index
            else:
                break
```

### ZooKeeper Atomic Broadcast (Zab)

Zab is the consensus protocol used by Apache ZooKeeper for maintaining consistency.

#### Zab Protocol Phases:

##### Phase 1: Leader Election
```python
class ZabNode:
    def __init__(self, node_id, cluster):
        self.node_id = node_id
        self.cluster = cluster
        self.state = "LOOKING"
        self.current_epoch = 0
        self.zxid = 0  # ZooKeeper transaction ID
        self.leader_id = None
        
    def start_leader_election(self):
        self.state = "LOOKING"
        self.current_epoch += 1
        
        # Send vote for self initially
        vote = {
            'epoch': self.current_epoch,
            'zxid': self.zxid,
            'node_id': self.node_id
        }
        
        votes = {self.node_id: vote}
        
        # Exchange votes with other nodes
        for node in self.cluster:
            if node != self.node_id:
                try:
                    response = self.send_vote(node, vote)
                    votes[node] = response
                except Exception:
                    continue
                    
        # Determine leader based on votes
        leader_vote = self.determine_leader(votes)
        
        if leader_vote['node_id'] == self.node_id:
            self.become_leader()
        else:
            self.become_follower(leader_vote['node_id'])
            
    def determine_leader(self, votes):
        # Leader selection criteria:
        # 1. Highest epoch
        # 2. Highest zxid (if epoch is same)
        # 3. Highest node_id (if zxid is same)
        
        best_vote = None
        for vote in votes.values():
            if (best_vote is None or
                vote['epoch'] > best_vote['epoch'] or
                (vote['epoch'] == best_vote['epoch'] and vote['zxid'] > best_vote['zxid']) or
                (vote['epoch'] == best_vote['epoch'] and vote['zxid'] == best_vote['zxid'] and vote['node_id'] > best_vote['node_id'])):
                best_vote = vote
                
        return best_vote
```

##### Phase 2: Discovery
```python
def discovery_phase(self):
    """Leader discovers the most recent state from followers"""
    if self.state != "LEADER":
        return
        
    follower_states = []
    
    # Collect state from all followers
    for node in self.cluster:
        if node != self.node_id:
            try:
                state = self.request_follower_state(node)
                follower_states.append(state)
            except Exception:
                continue
                
    # Determine the most up-to-date state
    latest_zxid = max([state['last_zxid'] for state in follower_states] + [self.zxid])
    
    # Update own state if necessary
    if latest_zxid > self.zxid:
        self.sync_to_latest_state(latest_zxid, follower_states)
        
    self.zxid = latest_zxid
```

##### Phase 3: Synchronization
```python
def synchronization_phase(self):
    """Synchronize all followers to leader's state"""
    if self.state != "LEADER":
        return
        
    for node in self.cluster:
        if node != self.node_id:
            try:
                self.synchronize_follower(node)
            except Exception:
                continue
                
def synchronize_follower(self, follower):
    follower_zxid = self.get_follower_zxid(follower)
    
    if follower_zxid < self.zxid:
        # Send missing transactions
        missing_txns = self.get_transactions_after(follower_zxid)
        self.send_sync_message(follower, missing_txns)
    elif follower_zxid > self.zxid:
        # Follower has transactions leader doesn't have
        # This shouldn't happen in normal operation
        raise InconsistentStateError("Follower ahead of leader")
```

##### Phase 4: Broadcast
```python
def broadcast_transaction(self, transaction):
    """Atomic broadcast of transaction to all followers"""
    if self.state != "LEADER":
        raise NotLeaderError("Only leader can broadcast transactions")
        
    # Assign new zxid
    self.zxid += 1
    transaction['zxid'] = self.zxid
    
    # Phase 1: Propose to all followers
    acks = 0
    for node in self.cluster:
        if node != self.node_id:
            try:
                response = self.send_proposal(node, transaction)
                if response.get('ack'):
                    acks += 1
            except Exception:
                continue
                
    # Phase 2: Commit if majority acknowledged
    if acks >= len(self.cluster) // 2:
        # Commit locally
        self.commit_transaction(transaction)
        
        # Send commit to all followers
        for node in self.cluster:
            if node != self.node_id:
                try:
                    self.send_commit(node, transaction['zxid'])
                except Exception:
                    continue
                    
        return True
    else:
        # Abort transaction
        return False
```

## Consensus Use Cases

### 1. Distributed Databases
```
Use Case: Ensure all replicas agree on transaction order
Examples: Google Spanner, CockroachDB, TiDB
Consensus Role: Order transactions, elect primary replica
```

### 2. Configuration Management
```
Use Case: Consistent configuration across distributed services
Examples: etcd, Consul, ZooKeeper
Consensus Role: Agree on configuration updates, leader election
```

### 3. Blockchain Systems
```
Use Case: Agree on next block in the chain
Examples: Bitcoin (Proof of Work), Ethereum 2.0 (Proof of Stake)
Consensus Role: Block ordering, fork resolution
```

### 4. Distributed Locking
```
Use Case: Coordinate exclusive access to resources
Examples: Chubby, etcd distributed locks
Consensus Role: Elect lock manager, coordinate lock grants
```

### 5. State Machine Replication
```
Use Case: Keep multiple replicas in sync
Examples: Raft-based systems, PBFT systems
Consensus Role: Order state machine operations
```

## Conclusion

Distributed consensus is fundamental to building reliable distributed systems. Key takeaways:

1. **Understand the Impossibility**: Perfect consensus is impossible in asynchronous systems
2. **Choose Appropriate Model**: Select consensus algorithm based on failure model and requirements
3. **Consider Practical Factors**: Network partitions, performance, and Byzantine faults
4. **Implement Proper Termination**: Use failure detectors and timeouts appropriately
5. **Learn from Real Systems**: Study Paxos, Raft, and Zab implementations

The choice of consensus algorithm depends on your specific requirements for consistency, availability, performance, and fault tolerance.

## Next Steps

In the following sections, we'll explore:
- **Asynchronous Programming**: Building event-driven distributed systems
- **Rate Limiting**: Controlling access and preventing overload
- **Security**: Protecting distributed consensus systems